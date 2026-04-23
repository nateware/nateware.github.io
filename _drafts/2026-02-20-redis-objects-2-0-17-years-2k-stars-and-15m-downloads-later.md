---
layout: post
title: "🔴 redis-objects 2.0: 17 years, 2k stars, and 15M downloads later"
date: 2026-02-20 12:00:00 -0800
author: "Nate Wiger"
categories: redis technology open-source
tags: redis ruby open-source
---

17 years ago, I created the Ruby gem [redis-objects](https://github.com/nateware/redis-objects) to solve a problem I was having at PlayStation. I never expected to still be maintaining it this many years later — the gem is older than my kids! I just released version 2.0, which got me reflecting on the journey.

## The Real-Time Leaderboard Problem

Back in 2009, I was working on online games at PlayStation. This meant massive concurrency — millions of players trying to join matches simultaneously. At the core of this was "the leaderboard problem": players wanted to see where they ranked within a few seconds of finishing their game.

Sorting at scale is a genuinely hard problem, and a bunch of PhD dissertations have been written about it. I don't have a PhD, but I still had to solve it. Sorting in a relational database completely fell apart at high transaction volumes. When I later joined AWS this was validated — Amazon heavily biases towards NoSQL solutions like [Dynamo](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf).

Luckily, [Redis](https://redis.io/) was released by the amazing [antirez](https://antirez.com/) right around the same time. It's a genius piece of software — basically network-accessible data structures with very low latency. The [Hacker News history](https://news.ycombinator.com/item?id=35871462) is a great read.

It was immediately apparent to me that [Redis Sorted Sets](https://redis.io/docs/latest/develop/data-types/sorted-sets/) were the ideal solution to sorting player scores. I'm quite confident PlayStation was one of the first users of Redis at scale, and our leaderboard implementation may have been the first one. I wrote [An Atomic Rant](/2010/02/18/an-atomic-rant/) about it, and later published [Real-time Leaderboards with ElastiCache for Redis](https://nateware.com/2013/09/08/real-time-leaderboards-with-elasticache-for-redis/) while at AWS. (I wrote the original [AWS Caching Whitepaper](/assets/docs/performance-at-scale-with-amazon-elasticache.pdf) as well)

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

(I've always loved how beautiful Ruby is, and I've always been surprised it didn't catch on as widely as Python, given it can do all the same stuff. Oh well.)

## What Open Source Has Taught Me About Coding

I've been a big proponent of open source since I started coding in Perl in the late 90's (or "late 1900's" as my kids say). Before `Redis::Objects` I released several Perl libraries including [CGI::FormBuilder](https://metacpan.org/pod/CGI::FormBuilder) and [SQL::Abstract](https://metacpan.org/pod/SQL::Abstract).

Designing a library so it can be open sourced fundamentally changes how you approach a problem. It forces the code to be configurable, and keeps you from accidentally embedding business logic only relevant to your current company (or project!). You have no idea who's using it, what they're doing with it, or what weird edge cases they'll hit.

It also gives you a chance to learn from passionate coders you'd never meet otherwise. `Redis::Objects` actually has 85 contributors from around the world (so far). I've met some really cool people over the years who have taught me new approaches and better ways to structure code.

The other thing about maintaining open source is that you can't just build something and walk away. Ruby upgrades, Redis changes, new conventions — keeping the library current has been a great forcing function to keep up with the latest tech trends.

## The Nested Class Key Collision Bug

I hit a bug that I was able to quickly diagnose, but the solution stumped me for many years. It was subtle and took years to surface ([#213](https://github.com/nateware/redis-objects/issues/213)).

The problmen is the gem generates Redis keys based on class names. For a `Team` class with a `hits` counter, the key might be `team:1:hits`. Simple enough. But what if you have nested classes like `Dog::Behavior` and `Cat::Behavior`? The original key generation logic stripped everything before `::`, so both would generate keys starting with `behavior:` — thus writing to the same Redis keys. This is bad, like crossing the streams in Ghostbusters.

How did this happen? From me in 2018:

> Nested class names are a problem because I stole the original code from Sinatra, and unfortunately I didn't notice that it cropped the class down past the final ::. Sorry. I am open to a new option something like `full_prefix: true` but unfortunately we can't change the code since that would be backwards incompatible.

I was stuck trying to release 2.0 for years because my initial fix broke backwards compatibility, despite my comment above. The reality is you just can't do that when data is involved — it's too disruptive. But I couldn't figure out another way.

Luckily, I randomly received a [PR from Matthew Hively](https://github.com/nateware/redis-objects/pull/276) which introduced an option flag that affected users could turn on. Open source community to the rescue! I can confidently say I would not have come up with that idea.

## Why I Keep Doing This

There's something about having built something that's still useful after all this time. Somewhere out there, redis-objects is handling atomic operations for games, e-commerce sites, social networks, and who knows what else. I'll never meet most of the people using it, but knowing it's out there doing useful work feels good.

I'm also incredibly thankful for the people who have contributed over the years. Pull requests, bug reports, issue discussions — all of it has made this gem better than I could have made it alone.

Version 2.0 fixes some longstanding bugs and makes the API more consistent. If you're using redis-objects, check out the [upgrade guide](https://github.com/nateware/redis-objects#important-20-changes) — there are a few breaking changes, but they're worth it.

I don't know if I'll be maintaining this for another 17 years, but I'll probably keep at it for a while longer. It's become part of who I am as an engineer, and I'm not ready to let that go yet.
