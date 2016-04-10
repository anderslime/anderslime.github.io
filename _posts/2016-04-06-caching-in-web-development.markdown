---
layout: post
title: "Caching in Web Development"
description: Describing the caching basics with important characteristics and properties.
teaser: Describing the caching basics with important characteristics and properties in caching required to be able to find the caching technique you need for your use case.
date: 2016-04-06
categories:
- caching
- cache invalidation
- cache management
- caching properties
---

While writing my Master's thesis i find a lot of interesting stuff i would like
to share. The thesis is about the design and implementation of a caching
system that updates automatically when the data it depends on are changed.

In this first post i will cover the important characteristics and properties
you should consider when you want to apply caching in your web application. In
later post i will cover specific techniques, how you pick a caching technique
for your use case and some open source implementations of caching and last
cover the results of my thesis.

But first - let's start with a motivational speech about caching:

## Why should i use caching?

There are multiple advantages of using caching in your web application, but
**performance** and **reduced CPU usage** are two of the bigger ones. For the
few non-programmers reading this: this means we get a faster application while
using our servers less.

But like many other programming techniques, caching also comes with challenges
and thereby trade-offs. I have found caching to be a hard challenge that often
comes with a gotcha here and there. And this is something even <a href="http://martinfowler.com/bliki/TwoHardThings.html" target="_blank">Martin Fowler agrees on this</a>.
The challenges and trade-offs depends on the caching techniques used, but
managing **cache invalidation**, **staleness** and **consistency** are
challenges often described in the context of caching.

## Caching Basics

Most web applications works by having a web server that receives an
HTTP-request, which is handled by some code that probably fetches some data in
the a storage system. To get the most out of caching, the web application is
often extended with a in-memory key-value store for caching that has the easy
task of storing arbitrary content identified by some key. Popular choices for a
cache storage are: Redis and Memcached. This results in the architecture below.

<img src="/images/cache_architecture.png" />

In overall the Web Application will implement different caching techniques that
decides which value to cache and when the cached value is too old to use. The
technique you decide to use depends on each use case.

When you want to pick a caching technique you have to decide how much of your content
you want to cache - also called the **granularity**. In web applications identifying itselfs with the figure above, this scale goes
from the finest granularity found in the raw data to the most coarse granularity
in the HTTP-request sent from the web server.

Optimally you want to cache until you achieve a satisfactory performance gain, but some content is easier
to cache than others and some granularities has certain advantages when it
comes to handling invalidation, which is another important choice. The
**invalidation technique** decides the **consistency** and **freshness** guarantees.

In overall there exists three categories of invalidation techniques: **key**, **trigger** or
**expiration**-based invalidation.

### Key-based Invalidation
In key-based invalidation, you calculate a key that is ensured to change when
the cached content changes. This technique has the advantage that the value
returned is always fresh, but it requires the developer to deduce a key
calculation that is guaranteed to change when the cached value changes. The
key-based invalidation technique is described well in
<a href="https://signalvnoise.com/posts/3113-how-key-based-cache-expiration-works" target="_blank">a blog post by David Heinemeier</a>.

### Trigger-based invalidation

The cache can also be invalidated using pre-defined triggers. This could
be triggers placed manually in the code or callbacks/events from when some data
is updated. Trigger-based invalidation also ensures to invalidate stale values
immediately, but it requires the developer to ensure that the triggers are
invoked every time a given cached value has to be invalidated.

### Expiration-based invalidation

If a short period of staleness can be tolerated, the cache can also be managed
using an expiration-based invalidation technique. This technique works by
defining some kind of expiration interval after which the cached value should
be invalidated. This technique makes cache management simple, but it requires
the sacrifice of staleness and potentially inconsistent data.

With a little knowledge about the important characteristics and properties on
application-level caching, we are now capable of evaluating caching techniques
in web applications.
