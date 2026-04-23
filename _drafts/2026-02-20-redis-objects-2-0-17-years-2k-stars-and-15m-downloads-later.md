---
layout: post
title: "🔴 redis-objects 2.0: 17 years, 2k stars, and 15M downloads later"
date: 2026-02-20 12:00:00 -0800
author: "Nate Wiger"
categories: redis technology open-source
tags: redis ruby open-source
---

17 years ago, I created the Ruby gem [redis-objects](https://github.com/nateware/redis-objects) to solve a problem I was having at PlayStation. I never expected to still be maintaining it so many years later — the gem is older than my kids! I just released version 2.0, which got me reflecting on the journey.

## What's New in 2.0

Not much, actually - except one breaking change that renames the lock method to `redis_lock`. If you break the public API you have to bump the major version. I thought the history of the library would be interesting though...

## The Real-Time Leaderboard Problem

Back in 2009, I was working on online games at PlayStation. This meant massive concurrency — millions of players trying to join matches simultaneously. At the core of this was "the leaderboard problem": players wanted to see where they ranked within a few seconds of finishing their game.

Sorting at scale is a genuinely hard problem, and a bunch of PhD dissertations have been written about it. I don't have a PhD, but I still had to solve it. Sorting in a relational database completely fell apart at high transaction volumes. When I later joined AWS this was validated — Amazon heavily biases towards NoSQL solutions like [Dynamo](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf).

Luckily, [Redis](https://redis.io/) was released by the amazing [antirez](https://antirez.com/) right around the same time. It's a genius piece of software — basically network-accessible data structures with very low latency. The [Hacker News history](https://news.ycombinator.com/item?id=35871462) is a great read.

It was immediately apparent to me that [Redis Sorted Sets](https://redis.io/docs/latest/develop/data-types/sorted-sets/) were the ideal solution to sorting player scores. I'm quite confident PlayStation was one of the first users of Redis at scale, and our leaderboard implementation may have been the first one. I wrote [An Atomic Rant](/2010/02/18/an-atomic-rant/) about it, and later published [Real-time Leaderboards with ElastiCache for Redis](https://nateware.com/2013/09/08/real-time-leaderboards-with-elasticache-for-redis/) while at AWS. (I later wrote the [AWS Caching Whitepaper](/assets/docs/performance-at-scale-with-amazon-elasticache.pdf) as well)

So how does `Redis::Objects` fit in? It maps Redis data types to their equivalents in Ruby. You write Ruby code like you normally would, and Redis is used transparently behind the scenes.

```ruby
leaderboard = Redis::SortedSet.new('leaderboard')

# Set the players' scores
leaderboard['Nate']  = 15378
leaderboard['Peter'] = 17522
leaderboard['Jeff']  = 19405

# Show players in ranked order
leaderboard[0..2]          # => ["Jeff", "Peter", "Nate"]

# Find my position in the leaderboard
leaderboard.rank('Nate')   # => 2
```

Simple and a lot prettier than `redis.zadd` (granted I'm biased).

## Design by Committee (in a Good Way)

Back in 2014, [Felipe Lopes](https://github.com/felipeclopes) filed issue [#150](https://github.com/nateware/redis-objects/issues/150) asking for `connection_pool` support. Felipe opened a PR, but it was a fairly invasive rewrite, and we got stuck on whether it would break key-eval support. The thread sat.

Then a different contributor, [Max Melentiev](https://github.com/printercu), dropped in with a `method_missing`-based proxy approach. [Jared Jenkins](https://github.com/jaredjenkins) jumped in and refined it into a proper `ConnectionProxy` class. The three of them iterated, debating different design tradeoffs. I mostly nudged from the sidelines and reminded them to add tests.

What shipped as `v1.1.0` was a better design than anything I would have written alone. Plus I didn't even need a connection pool, so I wouldn't have added one! But the library got better because three strangers across different timezones argued about it in a GitHub thread.

## The Silent Marshal Corruption

I learned one lesson the hard way: once your library is in the wild, you have no idea what data already lives in your users' databases.

In `v0.9.0`, I cleaned up how values were serialized with `marshal: true`. Previously, strings and numbers were stored raw; after the change, everything ran through `Marshal.dump`. It looked like a harmless consistency fix. But it silently broke reads for anyone who upgraded with existing data.

The error was cryptic: `TypeError: incompatible marshal file format (can't be read) — format version 4.8 required; 85.114 given`. Those numbers are the ASCII codes of whatever the stored string happened to start with — the code was trying to read an unmarshaled value as a marshaled blob.

Issue [#147](https://github.com/nateware/redis-objects/issues/147) got filed in 2014. I couldn't reproduce it and closed it. It got reopened. Others confirmed it. Three years later, [Nisanth Chunduru](https://github.com/nisanthchunduru) tracked down the exact commit:

> We had upgraded redis-objects from version 0.5.2 to 0.9.1. [...] 0.5.2 doesn't marshal simple objects like String, Fixnum, Bignum and Float. 0.9.0 changed this behaviour and started marshalling all objects.

The issue thread spans nearly a decade. An [April 2025 comment](https://github.com/nateware/redis-objects/issues/147#issuecomment-2773434610) from another user in Brazil begins "For any lost soul that faces this issue" complete with reproduction steps. I wish I had those steps 10 years ago!

Still, the lesson that you can't change code that deals with data really stuck with me, and was a huge challenge for the next bug.

## The Nested Class Key Collision Bug

This time, users hit a bug that I was able to quickly root cause, but the solution stumped me for a long time. It was subtle and took years to surface ([#231](https://github.com/nateware/redis-objects/issues/231)).

The problem is the gem generates Redis keys based on class names. For a `Team` class with a `hits` counter, the key might be `team:1:hits`. But what if you have nested classes like `Dog::Behavior` and `Cat::Behavior`? Well, the original key generation logic stripped everything before `::`, so both would generate keys starting with `behavior:` — thus writing to the same Redis keys. This is bad, like [crossing the streams](https://www.youtube.com/watch?v=wyKQe_i9yyo) in Ghostbusters.

How did this happen? From [me in 2018](https://github.com/nateware/redis-objects/issues/231#issuecomment-427695149):

> Nested class names are a problem because I stole the original code from Sinatra, and unfortunately I didn't notice that it cropped the class down past the final `::`. Sorry.

I was stuck trying to release 2.0 for *years* because my initial fix broke backwards compatibility. The reality is you just can't do that when data is involved — it's too disruptive. But I couldn't figure out another way.

Luckily, in late 2025, I randomly received a [PR from Matthew Hively](https://github.com/nateware/redis-objects/pull/276) which introduced an option that affected users could turn on. Open source community to the rescue! I can confidently say I would not have come up with that idea. Finally `v2.0.0` could be shipped, almost 8 years later.

## Why I Keep Doing This

I've been a proponent of open source since I started coding in Perl in the late 90's (or "late 1900's" as my kids say). I started by releasing several Perl libraries including [CGI::FormBuilder](https://metacpan.org/pod/CGI::FormBuilder) and [SQL::Abstract](https://metacpan.org/pod/SQL::Abstract). I've since given those away to others, but I still remember fondly all the hours I spent on them.

It's fun to think that [redis-objects](https://github.com/nateware/redis-objects) is being used in games, ecommerce sites, social networks, and who knows what else out there. I'll never meet most of the people using it, but knowing it's out there and being used feels good.

I don't know if I'll be maintaining this for another 17 years (I'll be in my 60's after all) or if Ruby will even be around. Maybe AI will replace everything. But in the meantime, sincere thanks to everyone who's used it and contributed to it over the past 17 years.

And if you need to build a real-time leaderboard, try [redis-objects](https://github.com/nateware/redis-objects) 2.0!