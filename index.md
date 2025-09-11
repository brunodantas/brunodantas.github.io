---
layout: default
title: Bruno Dantas
---

# Welcome to My Blog

I'm Bruno Dantas, a software developer interested in Python and Functional Programming.

## Recent Posts

{% for post in site.posts limit:5 %}
- **{{ post.date | date: "%B %d, %Y" }}**: [{{ post.title }}]({{ post.url }})
{% endfor %}

## All Posts

[View all posts →](/blog/)

---

## About

This page is about software development
