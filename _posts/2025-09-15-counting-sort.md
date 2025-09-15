---
layout: post
title: "Counting Sort Performance in Python"
description: "Explore how Counting Sort, a distribution-based algorithm, can outperform Python's built-in Powersort in specific scenarios. Learn about O(n+k) complexity, implementation details, and benchmark results comparing counting sort against Python's sorted() function."
date: 2025-09-15
categories: [algorithms, python, benchmark, computer science]
tags: [algorithms, python, bechmark, computer science]
author: brunodantas
image: /assets/images/cosmic.png
seo:
  type: BlogPosting
---

When studying the field of [Sorting Algorithms](https://en.wikipedia.org/wiki/Sorting_algorithm), many Computer Science courses stop at some very famous ones, like Quicksort and Merge Sort. They are very efficient, performing at O(n log n) time, and can be applied to any data that can be compared. In fact, real-world sorting implementations, like CPython's [Timsort](https://en.wikipedia.org/wiki/Timsort) and [Powersort](https://en.wikipedia.org/wiki/Powersort) are derived from those simpler ones, while adding some pretty advanced techniques.

Most people stop there, for many reasons. Perhaps lack of curiosity. Perhaps we just had to move on after seeing too many algorithms that achieve the same thing. Perhaps people are convinced those are the best we can achieve. After all, if there was a faster algorithm, it would be well known, right?

Turns out that there's a whole class of alternative sorting algorithms that are asymptotically faster than the well known algorithms. And some of them are really simple. And we can beat Python's `sorted` (i.e. Powersort) in our benchmark with them*!

## What is this?

Today we are going to focus on [Counting Sort](https://en.wikipedia.org/wiki/Counting_sort), an algorithm from the class of [Distribution Sorts](https://en.wikipedia.org/wiki/Sorting_algorithm#Distribution_sorts). This class is especially interesting because it distributes elements into other data structures and then gathers them back afterwards, instead of comparing elements to each other.

If you studied Computer Science, you may remember that there are some algorithms that trade memory-efficiency for time-efficiency (see: [memoization](https://en.wikipedia.org/wiki/Memoization)). That's similar to what Distribution Sorts do. For instance, Counting Sort uses a big array to store every element, in addition to an output array, in case you're not sorting in-place.

The catch is: Counting Sort and others only work for **arrays of non-negative integers**. So, if you can map your objects clealy into non-negative integers beforehand, it should work. Given that restriction the complexity of Counting Sort is as follows.
- Time: O(n + k)
- Space: O(n + k)

Where n is the length of the array, and k is the maximum element of the array.

## Counting Sort

The basic idea is that you allocate a big array with size k (or list in our case) where each index i represents the values from the input array, and each stored value v represents the number of occurences of each element. After we populate this array, we just need to iterate it and populate our output array with each element i times its occurrences v.

My implementation of Counting Sort is defined as follows.

```python
from typing import List

def countingsort(lst: List[int]) -> List[int]:
    """Simple counting sort implementation assuming non-negative integers"""
    # Handle empty list
    if not lst:
        return []
    
    # Find the maximum value
    max_val = max(lst)

    # Initialize occurrences list
    # Using list multiplication for better performance
    occurs = [0] * (max_val + 1)
    
    # Count occurrences
    for i in lst:
        occurs[i] += 1
    
    # Build output list
    out = []
    for i, v in enumerate(occurs):
        # If there's at least one occurrence of i
        if v > 0:
            # Add the element i, v times
            out += [i] * v
    
    return out
```

This program should return the same result as the `sorted` function, as long as the input contains only non-negative integers. What might differ between them is the execution time and memory usage. 

Let's go through this algorithm and analyze its performance in various situations.

## Analysis

The memory usage of this program can be determined from verifying that the `occurs` list has size v where v is the largest element in the input, and that the output list has size `n`, size it has the same size as the input.

The time complexity isn't as straightforward to analyze because we're using Python's efficient list multiplication. Let's assume that appending occurrences in the last loop takes constant time.

We can see that computing the max value is an iteration over n elements, while counting occurrences takes k iterations, and building the output takes k iterations as well, resulting in 2k + n.

As we can see, this algorithm is insensitive to factors such the sorted input, likely because it doesn't make any comparison.

So, the best case for it is where the largest value is not too high, i.e. the lower k is, the more this algorithm approximates linear time/space.

The worst case is when k is too high, particularly when it is much bigger than the other elements.

## Benchmark

I decided to compare our Python Counting Sort (CS) code to CPython's implementation of Powersort from the `sorted` in Python 3.13. The experiments consist of 10 executions of each function given lists with random non-negative integers. I have varied the list size from 64 to 16,777,216, and range of numbers (k) from 100 to 100,000.

It is expected for CS to underperform, since our code is implemented using Python code, while Python's builtin functions are implemented in C. However, we found some surprising results.

<pre style="overflow-y: scroll; max-height: 200px;">
Very Small range (0-100)
--------------------------------------------------
Size     CountingSort    sorted()        CS_Mem     Sort_Mem   Speedup    Winner
--------------------------------------------------------------------------------
64       55.9μs          6.1μs           1.7KB      0.6KB      0.11x      sorted()
128      87.3μs          9.1μs           2.3KB      1.1KB      0.10x      sorted()
256      120.8μs         18.5μs          3.3KB      2.1KB      0.15x      sorted()
512      121.0μs         33.2μs          5.8KB      4.1KB      0.27x      sorted()
1024     123.6μs         68.4μs          9.9KB      12.1KB     0.55x      sorted()
2048     168.9μs         140.2μs         18.7KB     24.0KB     0.83x      sorted()
4096     226.0μs         301.7μs         36.8KB     48.0KB     1.34x      countingsort
8192     347.8μs         551.7μs         70.5KB     95.8KB     1.59x      countingsort
16384    580.1μs         1.1ms           131.9KB    191.5KB    1.95x      countingsort
32768    2.9ms           2.3ms           294.9KB    382.7KB    0.80x      sorted()
65536    12.6ms          4.5ms           551.9KB    765.6KB    0.35x      sorted()
131072   30.5ms          8.9ms           1.1MB      1.5MB      0.29x      sorted()
262144   67.7ms          18.1ms          2.1MB      3.0MB      0.27x      sorted()
524288   141.4ms         36.6ms          4.3MB      6.0MB      0.26x      sorted()
1048576  297.1ms         73.2ms          8.5MB      12.0MB     0.25x      sorted()
2097152  591.8ms         147.5ms         17.1MB     23.9MB     0.25x      sorted()
4194304  1.19s           300.0ms         34.2MB     47.8MB     0.25x      sorted()
8388608  2.39s           610.4ms         68.4MB     95.7MB     0.26x      sorted()
16777216 4.79s           1.23s           136.7MB    191.4MB    0.26x      sorted()

Small range (0-1000)
--------------------------------------------------
Size     CountingSort    sorted()        CS_Mem     Sort_Mem   Speedup    Winner
--------------------------------------------------------------------------------
64       329.6μs         6.9μs           9.9KB      0.6KB      0.02x      sorted()
128      365.7μs         10.3μs          11.6KB     1.1KB      0.03x      sorted()
256      475.2μs         19.6μs          14.7KB     2.1KB      0.04x      sorted()
512      609.3μs         32.7μs          20.7KB     4.1KB      0.05x      sorted()
1024     788.2μs         71.6μs          29.3KB     12.1KB     0.09x      sorted()
2048     984.1μs         146.2μs         42.9KB     24.1KB     0.15x      sorted()
4096     1.3ms           335.7μs         63.4KB     48.1KB     0.25x      sorted()
8192     1.3ms           745.9μs         96.9KB     96.1KB     0.56x      sorted()
16384    1.8ms           1.4ms           166.0KB    192.1KB    0.80x      sorted()
32768    2.3ms           2.8ms           290.0KB    384.0KB    1.25x      countingsort
65536    3.7ms           6.0ms           545.7KB    767.9KB    1.64x      countingsort
131072   5.7ms           11.9ms          1.1MB      1.5MB      2.08x      countingsort
262144   13.4ms          24.3ms          2.2MB      3.0MB      1.82x      countingsort
524288   91.0ms          49.2ms          4.2MB      6.0MB      0.54x      sorted()
1048576  260.0ms         99.6ms          8.4MB      12.0MB     0.38x      sorted()
2097152  653.7ms         202.5ms         17.1MB     24.0MB     0.31x      sorted()
4194304  1.23s           408.7ms         33.9MB     48.0MB     0.33x      sorted()
8388608  2.52s           818.4ms         66.5MB     96.0MB     0.33x      sorted()
16777216 5.30s           1.67s           135.6MB    191.9MB    0.32x      sorted()

Medium range (0-10000)
--------------------------------------------------
Size     CountingSort    sorted()        CS_Mem     Sort_Mem   Speedup    Winner
--------------------------------------------------------------------------------
64       3.4ms           6.8μs           80.2KB     0.6KB      0.00x      sorted()
128      3.3ms           8.8μs           82.7KB     1.1KB      0.00x      sorted()
256      3.9ms           21.0μs          87.1KB     2.1KB      0.01x      sorted()
512      3.6ms           41.3μs          95.5KB     4.1KB      0.01x      sorted()
1024     4.3ms           72.7μs          113.0KB    12.1KB     0.02x      sorted()
2048     4.8ms           151.5μs         146.2KB    24.1KB     0.03x      sorted()
4096     6.7ms           317.6μs         199.9KB    48.1KB     0.05x      sorted()
8192     8.0ms           705.1μs         293.7KB    96.1KB     0.09x      sorted()
16384    10.0ms          1.5ms           425.9KB    192.1KB    0.15x      sorted()
32768    11.7ms          3.1ms           618.0KB    384.1KB    0.27x      sorted()
65536    12.8ms          6.6ms           913.6KB    768.1KB    0.52x      sorted()
131072   14.8ms          13.6ms          1.4MB      1.5MB      0.91x      sorted()
262144   19.4ms          28.7ms          2.4MB      3.0MB      1.48x      countingsort
524288   28.3ms          59.6ms          4.8MB      6.0MB      2.11x      countingsort
1048576  47.1ms          124.3ms         9.2MB      12.0MB     2.64x      countingsort
2097152  82.5ms          260.5ms         16.8MB     24.0MB     3.16x      countingsort
4194304  601.5ms         537.1ms         35.6MB     48.0MB     0.89x      sorted()
8388608  1.90s           1.09s           68.1MB     96.0MB     0.58x      sorted()
16777216 4.41s           2.23s           129.3MB    192.0MB    0.51x      sorted()

Large range (0-100000)
--------------------------------------------------
Size     CountingSort    sorted()        CS_Mem     Sort_Mem   Speedup    Winner
--------------------------------------------------------------------------------
64       32.0ms          10.9μs          780.9KB    0.6KB      0.00x      sorted()
128      32.5ms          11.4μs          780.7KB    1.1KB      0.00x      sorted()
256      32.7ms          18.7μs          789.2KB    2.1KB      0.00x      sorted()
512      33.0ms          34.5μs          799.1KB    4.1KB      0.00x      sorted()
1024     33.5ms          72.0μs          816.8KB    12.1KB     0.00x      sorted()
2048     34.2ms          152.8μs         854.8KB    24.1KB     0.00x      sorted()
4096     35.7ms          324.0μs         923.2KB    48.1KB     0.01x      sorted()
8192     39.1ms          696.9μs         1.0MB      96.1KB     0.02x      sorted()
16384    44.9ms          1.5ms           1.3MB      192.1KB    0.03x      sorted()
32768    54.9ms          3.2ms           1.8MB      384.1KB    0.06x      sorted()
65536    72.1ms          6.7ms           2.6MB      768.1KB    0.09x      sorted()
131072   91.8ms          14.4ms          3.7MB      1.5MB      0.16x      sorted()
262144   110.3ms         30.6ms          5.4MB      3.0MB      0.28x      sorted()
524288   122.1ms         65.5ms          7.6MB      6.0MB      0.54x      sorted()
1048576  143.4ms         138.8ms         12.3MB     12.0MB     0.97x      sorted()
2097152  176.6ms         298.4ms         19.8MB     24.0MB     1.69x      countingsort
4194304  247.6ms         645.2ms         36.6MB     48.0MB     2.61x      countingsort
8388608  403.5ms         1.39s           74.1MB     96.0MB     3.43x      countingsort
16777216 693.8ms         2.93s           145.2MB    192.0MB    4.22x      countingsort
</pre>

As you can see, CS had much lower execution times than `sorted` in some very specific cases, while also using less memory with inputs of size 1000 or less. These results suggest that there's a certain ratio of size per range where CS is particularly efficient. Unfortunately though, this means that the use cases for CS should be even more limited. For this particular implementation, at least.

Inevitably, I think, we should go deeper and explore an implementation that's, comparably to `sorted`, implemented and compiled in a performant language.

## Rust Implementation

The next step would be for me to test `sorted` against an implementation in either C or Rust, since the two languages are the most straightforward for creating efficient compiled packages for Python.

C was one of the first languages in my education, but something about [Python's instructions for C modules](https://docs.python.org/3/extending/extending.html) put me off. Rust's [Maturin](https://www.maturin.rs/index.html) seems much easier to use, on the other hand, so I went with Rust.

I actually don't know any Rust yet, and thought of implementing CS in it myself while doing a tutorial. Maybe another day: I decided to use an [existing CS package](https://gitlab.com/counting_sort/crate/-/tree/main/) and just import it from Python. With a little help from the docs plus my *smart automated assistant*, I've reached the following code for this module. I just needed to build it with `maturin develop -r` and it got compiled, optimized, and placed in my virtualenv.

```rust
use pyo3::prelude::*;
use pyo3::types::PyList;
use counting_sort::CountingSort;

#[pyfunction]
fn counting_sort(lst: &Bound<'_, PyList>) -> PyResult<Vec<usize>> {
    let vec: Vec<usize> = lst.extract()?;
    match vec.iter().cnt_sort() {
        Ok(sorted_vec) => Ok(sorted_vec),
        Err(_) => Err(pyo3::exceptions::PyValueError::new_err("Failed to sort")),
    }
}

/// A Python module implemented in Rust.
#[pymodule]
fn counting_sort_rust(m: &Bound<'_, PyModule>) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(counting_sort, m)?)?;
    Ok(())
}
```

Now we can use the same benchmark to test it against Python's Powersort.

## Rust Benchmark

This time we went with different range increments, because the results were the same starting at k = 500.

<pre style="overflow-y: scroll; max-height: 200px;">
Very Small range (0-100)
--------------------------------------------------
Size     CountingSort    sorted()        CS_Mem     Sort_Mem   Speedup    Winner
--------------------------------------------------------------------------------
64       3.4μs           5.0μs           0.6KB      0.6KB      1.49x      countingsort
128      5.0μs           8.4μs           1.1KB      1.1KB      1.67x      countingsort
256      6.7μs           14.6μs          2.1KB      2.1KB      2.19x      countingsort
512      8.5μs           35.9μs          4.1KB      4.1KB      4.24x      countingsort
1024     18.5μs          64.1μs          8.1KB      12.1KB     3.47x      countingsort
2048     40.0μs          150.5μs         16.1KB     24.0KB     3.76x      countingsort
4096     53.1μs          277.5μs         32.1KB     48.0KB     5.23x      countingsort
8192     105.7μs         589.5μs         64.1KB     95.8KB     5.58x      countingsort
16384    214.8μs         1.2ms           128.1KB    191.5KB    5.43x      countingsort
32768    412.7μs         2.3ms           256.1KB    382.7KB    5.61x      countingsort
65536    813.6μs         4.6ms           512.1KB    765.6KB    5.68x      countingsort
131072   1.7ms           9.3ms           1.0MB      1.5MB      5.58x      countingsort
262144   3.6ms           18.8ms          2.0MB      3.0MB      5.14x      countingsort
524288   7.5ms           38.1ms          4.0MB      6.0MB      5.10x      countingsort
1048576  15.0ms          77.2ms          8.0MB      12.0MB     5.14x      countingsort
2097152  30.1ms          154.3ms         16.0MB     23.9MB     5.13x      countingsort
4194304  61.9ms          311.4ms         32.0MB     47.8MB     5.03x      countingsort
8388608  122.4ms         620.7ms         64.0MB     95.7MB     5.07x      countingsort
16777216 245.5ms         1.25s           128.0MB    191.4MB    5.09x      countingsort

Very Small range (0-200)
--------------------------------------------------
Size     CountingSort    sorted()        CS_Mem     Sort_Mem   Speedup    Winner
--------------------------------------------------------------------------------
64       4.4μs           4.1μs           0.6KB      0.6KB      0.92x      sorted()
128      3.7μs           8.4μs           1.1KB      1.1KB      2.30x      countingsort
256      6.2μs           14.8μs          2.1KB      2.1KB      2.40x      countingsort
512      8.6μs           39.5μs          4.1KB      4.1KB      4.57x      countingsort
1024     16.6μs          65.0μs          8.1KB      12.1KB     3.92x      countingsort
2048     27.8μs          147.0μs         16.1KB     24.1KB     5.29x      countingsort
4096     57.0μs          290.0μs         32.1KB     48.1KB     5.09x      countingsort
8192     159.4μs         616.6μs         64.1KB     96.0KB     3.87x      countingsort
16384    279.7μs         1.3ms           128.1KB    191.9KB    4.48x      countingsort
32768    481.8μs         2.5ms           256.1KB    383.3KB    5.21x      countingsort
65536    816.6μs         5.0ms           512.1KB    766.8KB    6.18x      countingsort
131072   1.7ms           10.2ms          1.0MB      1.5MB      5.85x      countingsort
262144   3.7ms           20.5ms          2.0MB      3.0MB      5.54x      countingsort
524288   7.6ms           41.5ms          4.0MB      6.0MB      5.49x      countingsort
1048576  16.3ms          83.4ms          8.0MB      12.0MB     5.11x      countingsort
2097152  30.7ms          166.7ms         16.0MB     24.0MB     5.43x      countingsort
4194304  63.6ms          344.1ms         32.0MB     47.9MB     5.41x      countingsort
8388608  123.0ms         675.9ms         64.0MB     95.8MB     5.50x      countingsort
16777216 254.9ms         1.39s           128.0MB    191.7MB    5.46x      countingsort

Very Small range (0-300)
--------------------------------------------------
Size     CountingSort    sorted()        CS_Mem     Sort_Mem   Speedup    Winner
--------------------------------------------------------------------------------
64       9.5μs           6.0μs           0.9KB      0.6KB      0.63x      sorted()
128      9.8μs           8.0μs           1.5KB      1.1KB      0.81x      sorted()
256      24.4μs          15.4μs          3.3KB      2.1KB      0.63x      sorted()
512      34.2μs          31.6μs          6.3KB      4.1KB      0.92x      sorted()
1024     66.7μs          68.3μs          12.0KB     12.1KB     1.02x      countingsort
2048     132.1μs         152.1μs         24.6KB     24.1KB     1.15x      countingsort
4096     236.4μs         319.7μs         48.3KB     48.1KB     1.35x      countingsort
8192     548.2μs         630.3μs         98.0KB     96.0KB     1.15x      countingsort
16384    1.2ms           1.5ms           191.9KB    191.9KB    1.26x      countingsort
32768    1.9ms           2.6ms           387.0KB    383.7KB    1.39x      countingsort
65536    3.7ms           5.4ms           772.0KB    767.3KB    1.46x      countingsort
131072   7.3ms           10.9ms          1.5MB      1.5MB      1.49x      countingsort
262144   15.1ms          22.1ms          3.0MB      3.0MB      1.46x      countingsort
524288   30.7ms          44.2ms          6.0MB      6.0MB      1.44x      countingsort
1048576  61.2ms          100.3ms         12.1MB     12.0MB     1.64x      countingsort
2097152  123.5ms         181.3ms         24.2MB     24.0MB     1.47x      countingsort
4194304  250.3ms         363.3ms         48.4MB     47.9MB     1.45x      countingsort
8388608  493.6ms         724.5ms         96.8MB     95.9MB     1.47x      countingsort
16777216 986.7ms         1.46s           193.5MB    191.8MB    1.48x      countingsort

Very Small range (0-400)
--------------------------------------------------
Size     CountingSort    sorted()        CS_Mem     Sort_Mem   Speedup    Winner
--------------------------------------------------------------------------------
64       13.7μs          4.2μs           1.3KB      0.6KB      0.30x      sorted()
128      20.3μs          10.3μs          2.5KB      1.1KB      0.51x      sorted()
256      34.9μs          15.4μs          4.5KB      2.1KB      0.44x      sorted()
512      64.1μs          39.9μs          8.9KB      4.1KB      0.62x      sorted()
1024     129.8μs         83.6μs          17.7KB     12.1KB     0.64x      sorted()
2048     257.4μs         150.6μs         36.5KB     24.1KB     0.58x      sorted()
4096     509.9μs         311.1μs         73.3KB     48.1KB     0.61x      sorted()
8192     1.1ms           694.1μs         144.6KB    96.1KB     0.65x      sorted()
16384    2.0ms           1.4ms           289.2KB    192.0KB    0.71x      sorted()
32768    3.9ms           2.8ms           579.5KB    383.7KB    0.73x      sorted()
65536    7.9ms           5.7ms           1.1MB      767.4KB    0.73x      sorted()
131072   15.6ms          11.4ms          2.3MB      1.5MB      0.73x      sorted()
262144   31.8ms          23.0ms          4.5MB      3.0MB      0.72x      sorted()
524288   63.4ms          47.0ms          9.0MB      6.0MB      0.74x      sorted()
1048576  127.4ms         95.1ms          18.1MB     12.0MB     0.75x      sorted()
2097152  254.6ms         191.3ms         36.1MB     24.0MB     0.75x      sorted()
4194304  510.0ms         384.3ms         72.2MB     48.0MB     0.75x      sorted()
8388608  1.02s           777.0ms         144.5MB    95.9MB     0.76x      sorted()
16777216 2.14s           1.56s           288.8MB    191.8MB    0.73x      sorted()
</pre>

As we can see, Rust CS dominated the results up to range 300, reaching close to 6.2X Speedup. At higher ranges, this version of CS lose performance due to memory access overhead, perhaps. Interestingly, its the memory usage is comparable to `sorted`'s, but growing faster as the range of elements grow.

# Conclusion

In this post, we have seen an analysis and benchmark of Counting Sort. I hope it was informative. My hope was that we could leave this with clearer use cases for this algorithm, what remains is the conventional wisdom of using it when your maximum element is not large.
