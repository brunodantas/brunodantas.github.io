---
title: "Defining Mature Optimization"
description: "This is a short excerpt from the book Django QuerySet Performance, which aims at a positive definition for performance enhancement, in contrast to Knuth's famous quote."
date: 2026-02-11
categories: [performance-engineering, backend-development]
tags: [software-engineering, performance]
author: brunodantas
# image: /assets/images/cosmic.png
seo:
  type: BlogPosting
---

This is a short excerpt from the book [Django QuerySet Performance](https://a.co/d/0aLPhpKJ), which aims at a positive definition for performance enhancement, in contrast to Knuth's famous quote.

---

> Premature optimization is the root of all evil.
> 
> — Donald Knuth

The quote above is often reproduced in the context of software enhancements.

This is most often interpreted as it being useless (or worse than useless) to improve the performance of something that has not been consolidated yet, since software has a tendency to change.

In other words: first you must make your software work. Then you need stability, in some ways. Only then, it's time to optimize.

With that in mind, in this chapter we will go beyond the quote and determine what should be optimized, and why. The *how* will come in later chapters.

### Mature Optimization

Following from the "premature optimization" quote, and the previous points, we can reach some conclusions. Let's see some definitions regarding what could be called *Mature Optimization*, also based on some concepts from Performance Engineering. 

- We should have a system running in production and have a significant number of users before trying to optimize it. At that point, performance starts to become relevant in the real world.
- Performance improvements may be always welcome, but some are more important than others. This is a product of the criticality, frequency, and duration of the operation.
- Ideally, we should have Key Performance Indicators (KPI) as targets for current and future developments.

And from that, it follows that **performance in production is what matters**.

That may be obvious, but it's worth noting. Because from that, we can conclude that our system measurements must come from the production environment, or at least, an environment that replicates production as much as possible. This means using the same or similar resources, databases, data, etc. And ideally, the performance engineer should be able to simulate a number of users similar to the real number in production with certain special tools like JMeter or Locust, but that goes beyond the scope of this book. For simplicity, let's call this environment *test environment*.

> Test performance also matters
> 
> The book *Speed Up Your Django Tests* by Adam Johnson is recommended reading on this subject.

Expanding on the second point, we'll want to prioritize performance improvements by how important the operation is in the system, by how often it occurs overall, and by how long it takes to complete. These three metrics determine the relevance of an optimization effort. After all, any refactoring brings a certain amount of risk.

Finally, for Web Application KPI, we can include the following. Specific target values depend on the application domain.

- Response time (latency), especially user-facing web requests. This is the most straightforward metric, and will be our main focus.
- System capacity (throughput). It should be possible to meet 100% of the projected peak load, measured in requests per second.
- Resource utilization. Disk utilization especially, since that's our usual bottleneck.

We can generally improve all of these KPI by optimizing slow and/or frequent database queries.

<div class="page-break" style="page-break-before: always;"></div>


> See also
>
> According to author Jakob Nielsen, response times should not exceed 1 second, in order to keep the user's flow of thought uninterrupted.
> 
> See his article at:
> https://www.nngroup.com/articles/response-times-3-important-limits
