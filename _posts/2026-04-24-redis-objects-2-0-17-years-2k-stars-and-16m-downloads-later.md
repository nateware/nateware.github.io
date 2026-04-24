---
layout: post
title: "💎 redis-objects 2.0: 17 years, 2k stars, and 16M downloads later"
date: 2026-04-23 0:00:00 -0800
author: "Nate Wiger"
categories: redis technology open-source
tags: redis ruby open-source
---

17 years ago, I created the Ruby library [redis-objects](https://github.com/nateware/redis-objects) to solve a problem I was having at PlayStation. I never expected to still be maintaining it so many years later — it's older than my kids! I just released version 2.0, which got me reflecting on the journey and lessons learned.

## What's New in 2.0

Not much, actually - except one breaking change that renames the lock method to `redis_lock`, and the fix for an 8 year old bug that breaks data compatibility if you enable it. I thought the history of the library would be interesting though...

## The Real-Time Leaderboard Problem

Back in 2009, I was working on online games at PlayStation. This meant massive concurrency — millions of players trying to join matches simultaneously. At the core of this was "the leaderboard problem": players wanted to see where they ranked right after finishing their game.

Sorting at scale is a genuinely hard problem, and a bunch of PhD dissertations have been written about it. I don't have a PhD, but my team still had to solve it. Sorting in a relational database completely fell apart at high transaction volumes. When I later joined AWS this was validated; Amazon heavily biases towards NoSQL solutions like [Dynamo](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf).

Luckily, [Redis](https://redis.io/) was released by the amazing [antirez](https://antirez.com/) right around the same time. It's a genius piece of software — basically network-accessible data structures with very low latency. The [Hacker News history](https://news.ycombinator.com/item?id=35871462) is a great read.

It was immediately apparent to me that [Redis Sorted Sets](https://redis.io/docs/latest/develop/data-types/sorted-sets/) were the ideal solution to sorting player scores in real-time. I'm quite confident PlayStation was one of the first users of Redis at scale, and our Redis leaderboard implementation may have been the first one. I wrote [An Atomic Rant](/2010/02/18/an-atomic-rant/) about it, and later published [Real-time Leaderboards with ElastiCache for Redis](https://nateware.com/2013/09/08/real-time-leaderboards-with-elasticache-for-redis/) while at AWS. (I later wrote the [AWS Caching Whitepaper](/assets/docs/performance-at-scale-with-amazon-elasticache.pdf) as well)

So how does `Redis::Objects` fit in? It maps Redis data types to their equivalents in Ruby. You write Ruby code like you normally would, and Redis is used transparently behind the scenes.

```ruby
leaderboard
  = Redis::SortedSet.new('leaderboard')

# Set the players' scores
leaderboard['Nate']  = 15378
leaderboard['Peter'] = 17522
leaderboard['Jeff']  = 19405

# Find my position in the leaderboard
leaderboard.rank('Nate') # 2
```

Simple and a lot prettier than `redis.zadd` (granted I'm biased).

## Design by Committee (in a Good Way)

In 2014, [Felipe Lopes](https://github.com/felipeclopes) filed issue [#150](https://github.com/nateware/redis-objects/issues/150) asking for `connection_pool` support. Felipe opened a PR, but it was a fairly invasive rewrite, and we got stuck as it would break some core functionality.

Then [Max Melentiev](https://github.com/printercu) jumped in with a `method_missing`-based approach, and [Jared Jenkins](https://github.com/jaredjenkins) refined it into a proper `ConnectionProxy` class. The three of them iterated, debating different design tradeoffs. My big contribution was reminding them to write tests.

What shipped as `v1.1.0` was a better design than anything I would have written alone. Plus I didn't even need a connection pool, so I wouldn't have added one in the first place! But the library got better because three strangers across different timezones argued about it in a GitHub thread.

## The Silent Marshal Corruption

I learned one lesson the hard way: once your library is in the wild, you have no idea what data already lives in your users' databases.

In `v0.9.0` (Sep 2017), I cleaned up how values were serialized with `marshal: true`. Previously, strings and numbers were stored raw; after the change, everything ran through `Marshal.dump`. It seemed like a harmless consistency fix, and solved some other edge cases. But it silently broke reads for anyone who upgraded with existing numbers and strings in Redis.

The error was cryptic: `TypeError: incompatible marshal file format (can't be read) — format version 4.8 required; 85.114 given`. Turns out, those numbers are the ASCII codes of whatever the stored string happened to start with — the code was trying to read an unmarshaled value as a marshaled blob.

Issue [#147](https://github.com/nateware/redis-objects/issues/147) got filed in 2014. I couldn't reproduce it and closed it. But three years later, [Nisanth Chunduru](https://github.com/nisanthchunduru) tracked down [the exact commit](https://github.com/nateware/redis-objects/issues/147#issuecomment-252233167):

> We had upgraded redis-objects from version 0.5.2 to 0.9.1. [...] 0.5.2 doesn't marshal simple objects like String, Fixnum, Bignum and Float. 0.9.0 changed this behaviour and started marshalling all objects.

The issue thread spans nearly a decade. A recent [April 2025 comment](https://github.com/nateware/redis-objects/issues/147#issuecomment-2773434610) from another user in Brazil humorously begins "For any lost soul that faces this issue" complete with a solution. I wish I had that 10 years ago!

Still, the lesson that you can't change code that deals with data stuck with me, and was a huge challenge for the next bug.

## The Nested Class Key Collision Bug

This time, users hit a bug that I was able to easily root cause, but the solution stumped me for a long time. The bug was subtle and took years to surface ([#231](https://github.com/nateware/redis-objects/issues/231)).

The problem is the gem generates Redis key names based on Ruby class names. For a `Team` class with a `hits` counter, the key might be `team:1:hits`. But what if you have nested classes like `Dog::Behavior` and `Cat::Behavior`? Well, the key generation logic stripped everything up to `::`, so both would generate keys starting with `behavior:` — thus writing to the same Redis keys! This is bad, like [crossing the streams](https://www.youtube.com/watch?v=wyKQe_i9yyo) in Ghostbusters.

How did this happen? From [me in 2018](https://github.com/nateware/redis-objects/issues/231#issuecomment-427695149):

> Nested class names are a problem because I stole the original code from Sinatra, and unfortunately I didn't notice that it cropped the class down past the final `::`. Sorry.

I was stuck trying to release 2.0 for _years_ because my initial fix broke backwards compatibility. As I had learned, you can't do that when data is involved, it's too disruptive. But I couldn't figure out another way.

Luckily, in late 2025, I received an out-of-the-blue [PR from Matthew Hively](https://github.com/nateware/redis-objects/pull/276) which introduced an option that people with nested classes could use. Open source community to the rescue! I can confidently say I would not have come up with that idea. Finally `v2.0.0` could be shipped, almost 8 years later.

## Why Do I Keep Doing This?

I no longer code much Ruby - nowadays it's mostly React and Node via Claude for my side projects. But I've been a proponent of open source since I started coding in Perl in the late 90's (or "late 1900's" as my kids say). I started by releasing several Perl libraries including [CGI::FormBuilder](https://metacpan.org/pod/CGI::FormBuilder) and [SQL::Abstract](https://metacpan.org/pod/SQL::Abstract). I've since given those away to others, but I still remember fondly all the hours I spent on them.

It's fun to think that [redis-objects](https://github.com/nateware/redis-objects) is being used in games, ecommerce sites, social networks, and who knows what else. I'll never meet most of the people using it, but knowing it's out there and being used feels good.

I don't know if I'll be maintaining this for another 17 years (I'll be in my 60's after all) or if Ruby will even be around. Maybe AI will replace everything. But in the meantime, sincere thanks to everyone who's used it and contributed to it over the past 17 years.

And if you need to build a real-time leaderboard, try [redis-objects](https://github.com/nateware/redis-objects) 2.0!
