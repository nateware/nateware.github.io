---
layout: post
title: "🔴 redis-objects 2.0: 17 years, 2k stars, and 15M downloads later"
date: 2026-02-20 12:00:00 -0800
author: "Nate Wiger"
categories: redis technology open-source
tags: redis ruby open-source
---

17 years ago, I created the Ruby gem [redis-objects](https://github.com/nateware/redis-objects) to solve a problem I was having at PlayStation. I never expected to still be maintaining it so many years later. The gem is older than my kids! I just released version 2.0, and that got me reflecting on the journey.

## The Real-Time Leaderboard Problem

Back in 2009, I was working at PlayStation. The real-time nature of online games meant we needed to handle massive concurrency — millions of players trying to join matches simultaneously. At the core of this was "the leaderboard problem": Players wanted to instantly see where they ranked compared to everyone else, within a few seconds of finishing their game.

This is actually a pretty difficult sorting problem at scale, and a bunch of PhD dissertations have been written aboutit. I don't have a PhD, but I still had to solve it. I found that sorting in a relational database completely fell apart at high transaction volumes. When I later joined AWS this was validated - there's not a relational DB anywhere to be found.

Luckily, [Redis](https://redis.io/) was released by the amazing coder [antirez](https://antirez.com/) right around the same time. It's a genius piece of software - basically network accessible data structures with very low latency. The [Hacker News History](https://news.ycombinator.com/item?id=35871462) is a great read. 

It was immediately apparent to me that [Redis Sorted Sets](https://redis.io/docs/latest/develop/data-types/sorted-sets/) were the ideal solution to sorting player scores. I'm quite confident PlayStation was one of the first users of Redis at scale, and our Redis leaderboard solution may have been the first one. I wrote [An Atomic Rant](/2010/02/18/an-atomic-rant/) about it, and later published an article about [Real-time Leaderboards with ElastiCache for Redis](https://nateware.com/2013/09/08/real-time-leaderboards-with-elasticache-for-redis/) while at AWS.

So how does `Redis::Objects` fit in? Basically it maps Redis data types to their equivalent in Ruby. The result is you write Ruby code like you normally would, and Redis is transparently used behind the scenes. 

```ruby
leaderboard = Redis::SortedSet.new('leaderboard')

# Set the player's score
leaderboard['Nate']  = 15378
leaderboard['Peter'] = 17522
leaderboard['Jeff']  = 19405

# Show players in ranked order
leaderboard[0..2]           # => ["Nate", "Jeff", "Peter"]
leaderboard[0, 2]           # => ["Nate", "Jeff"]

# Find my rank in the leaderboard
leaderboard.rank('Peter')   # => 2
leaderboard.rank('Jeff')    # => 1
```

I've always loved how beautiful Ruby is, and have always been surprised it didn't catch on as widely as something like Python, given it can do all the stame stuff. Oh well.

## What Open Source Has Taught Me About Coding

I've been a big proponent of open source since I started coding in Perl in the late 90's (or "late 1900's" as my kids say). Before `Redis::Objects` I released Perl libraries including [CGI::FormBuilder](https://metacpan.org/pod/CGI::FormBuilder) and [SQL::Abstract](https://metacpan.org/pod/SQL::Abstract). 

I think designing a library so it can be open sourced fundamentally changes how you approach a problem. It forces you to make sure the code is configurable, and you don't accidentally embed business logic only relevant to your current company (or project!) into the code. With open source, you have no idea who's using your library, what they're doing with it, or what weird edge cases they'll hit.

It also gives you an opportunity to learn from other passionate coders who you would never meet otherwise. I've met some really cool people over the years who have contributed some really unique ideas. And learned a ton about how to structure code.

The other thing about maintaining open source software is that you're forced to stay current with current technology. You can't just build something and walk away. Over the years there have been Ruby upgrades, Redis changes, and so on, and having to keep my library up to date has been a great forcing function to stay current.

## The Nested Class Key Collision Bug

This one was subtle and took years to surface ([#213](https://github.com/nateware/redis-objects/issues/213)). The gem generates Redis keys based on class names. For a `Team` class with a `hits` counter, the key might be `team:1:hits`. Simple enough.

But what if you have nested classes? Like `Dog::Behavior` and `Cat::Behavior`? The original key generation logic stripped everything before `::`, so both would generate keys starting with `behavior:...`. Both classes would be writing to the same Redis keys. This is bad, like crossing the streams in Ghostbusters.

What was the root cause? As I explained in 2018:

> Nested class names are a problem because I stole the original code from Sinatra, and unfortunately I didn't notice that it cropped the class down past the final ::. Sorry. I am open to a new option something like full_prefix: true but unfortunately we can't change the code since that would be backwards incompatible.

I was stuck trying to release 2.0 for years because my initial solution broke backwards compatibility. The reality is you just can't break backwards compatibility. It's too disruptive. But I couldn't figure out a different way.

Luckily, I randomly received a [PR from Matthew Hively](https://github.com/nateware/redis-objects/pull/276) that introduced a new flag that affected people could use. Open open source community to the rescue. I can confidently say I would not have come up with that idea.

## Why I Keep Doing This

But also, there's something about having built something that's still useful after all this time. Somewhere out there, redis-objects is handling atomic operations for games, e-commerce sites, social networks, and who knows what else. I'll never know most of the people using it, but knowing it's out there being useful work feels good.

I'm also incredibly thankful for the people who have contributed over the years. Pull requests, bug reports, discussions on issues — all of it has made this gem better than I could have made it alone. Open source is a collaborative effort, and I'm grateful to everyone who's helped along the way.

Version 2.0 fixes some longstanding bugs and makes the API more consistent. If you're using redis-objects, check out the [upgrade guide](https://github.com/nateware/redis-objects#important-20-changes) — there are a few breaking changes, but they're worth it.

I don't know if I'll be maintaining this for another 17 years. But I'll probably keep at it for a while longer. It's become part of who I am as an engineer, and I'm not ready to let that go yet.
