---
layout: post
title: "Caching Approaches in Web Development"
description: An overview of different caching approaches in academic litteraure and the open source world.
teaser: An overview of different caching approaches in academic litteraure and the open source world.
date: 2016-04-03
categories:
- caching
- HTTP-caching
- ruby
- ruby on rails
---

While writing my Master's thesis i find a lot of interesting stuff i would like
to share. The thesis is about the design and implementation of a caching
system that updates automatically when the data it depends on are changed.

In this first post i want to share my overview of current caching techiques, which
are covered in academic litterature as well as in the world of open source projects.
But first - let's start with a motivational speach about caching:

## Why should i use caching?

There are multiple advantages of using caching in your web application, but
**performance** and **reduced CPU usage** are two of the bigger ones. For the
few non-programmers reading this: this means we get a faster application while
using our servers less.

But like many other programming techniques, caching also comes with challenges
and thereby trade-offs. I have found caching a a hard challenge that often
comes with a gotcha here and there. And this is something even <a href="http://martinfowler.com/bliki/TwoHardThings.html" target="_blank">Martin Fowler agrees on this</a>.
The challenges and trade-offs depends on the caching techniques used, but
managing **cache invalidation**, **staleness** and **consistency** are
challenges often described in the context of caching.

## Caching Approaches - A Brief Overview

Caching techniques can be divided into the **granuarity** of the value you
want to cache and the **invalidation technique** used.

In dynamic web applications this scale goes from the finest granuarlity found in
the raw data to the most coars granularity in the HTTP-request send from the
web server.

The staleness and consistency of the caching technique are desided by how
you invalidate the cached values. The cached values can be invalidated using
a key identifying the cached value, some trigger or an invalidation interval.
