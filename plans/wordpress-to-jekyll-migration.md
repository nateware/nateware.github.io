# Plan: Convert WordPress Blog (nateware.com) to Jekyll Posts

## Prompt

The following prompt was provided to Claude:

> this is a personal blog written in jekyll. i want to convert my existing wordpress blog into jekyll posts. my blog is located at: nateware.com . for each post on my blog, please create a corresponding markdown file. respect any of the formatting in my existing blog - headers, code segments, links, etc. for each existing blog post, create a new jekyll markdown file in the \_posts directory formatted as YYYY-MM-DD-[slug from existing blog]. For example my blog article located at https://nateware.com/2010/02/18/an-atomic-rant/ should be downloaded, converted to markdown, and saved as \_posts/2010-02-18-an-atomic-rant.md. Create a plan

## Context

The user has a personal blog at nateware.com running WordPress and wants to migrate all 11 blog posts into their new Jekyll site at `nateware.github.io`. Each post needs to be fetched, converted to clean Markdown with proper Jekyll front matter, and saved into `_posts/`. All formatting (headers, code blocks, links, images, embedded media) must be preserved.

## Posts to Convert (11 total)

| #   | Filename                                                          | Source URL                                                       |
| --- | ----------------------------------------------------------------- | ---------------------------------------------------------------- |
| 1   | `2010-02-18-an-atomic-rant.md`                                    | `/2010/02/18/an-atomic-rant/`                                    |
| 2   | `2010-06-14-atomic-rant-redux.md`                                 | `/2010/06/14/atomic-rant-redux/`                                 |
| 3   | `2012-12-01-replacing-macbook-hd-with-an-ssd.md`                  | `/2012/12/01/replacing-macbook-hd-with-an-ssd/`                  |
| 4   | `2013-04-06-linux-network-tuning-for-2013.md`                     | `/2013/04/06/linux-network-tuning-for-2013/`                     |
| 5   | `2013-09-08-real-time-leaderboards-with-elasticache-for-redis.md` | `/2013/09/08/real-time-leaderboards-with-elasticache-for-redis/` |
| 6   | `2014-03-21-game-analytics-with-aws-at-gdc-2014.md`               | `/2014/03/21/game-analytics-with-aws-at-gdc-2014/`               |
| 7   | `2015-03-28-advanced-game-analytics-with-aws-at-gdc-2015.md`      | `/2015/03/28/advanced-game-analytics-with-aws-at-gdc-2015/`      |
| 8   | `2019-02-04-how-amazon-ended-up-in-san-diego.md`                  | `/2019/02/04/how-amazon-ended-up-in-san-diego/`                  |
| 9   | `2019-04-08-donut-based-security.md`                              | `/2019/04/08/donut-based-security/`                              |
| 10  | `2019-07-09-nates-stock-market-theory-of-management.md`           | `/2019/07/09/nates-stock-market-theory-of-management/`           |
| 11  | `2019-07-24-the-job-title-director-sucks.md`                      | `/2019/07/24/the-job-title-director-sucks/`                      |

## Implementation Steps

### Step 1: Update `_config.yml`

- Add `permalink: /:year/:month/:day/:title/` to match existing WordPress URL structure for SEO/link preservation

### Step 2: Create assets directory

- `mkdir -p assets/images/` for any downloaded post images

### Step 3: Fetch and convert all 11 posts (3 parallel batches)

**For each post:**

1. Fetch the full page content via WebFetch
2. Extract title, categories, and tags
3. Convert HTML body to clean Markdown
4. Write file to `_posts/YYYY-MM-DD-slug.md`

**Batch A - Text/essay posts (4 posts, parallel):**

- `how-amazon-ended-up-in-san-diego` (leadership, no code)
- `donut-based-security` (leadership, has images)
- `nates-stock-market-theory-of-management` (leadership, no code)
- `the-job-title-director-sucks` (leadership, no code)

**Batch B - Technical posts with code (4 posts, parallel):**

- `an-atomic-rant` (Redis/Ruby code blocks)
- `atomic-rant-redux` (Redis/Ruby code, images)
- `linux-network-tuning-for-2013` (heavy config/bash blocks)
- `real-time-leaderboards-with-elasticache-for-redis` (Ruby/Redis code)

**Batch C - Posts with embedded media (3 posts, parallel):**

- `replacing-macbook-hd-with-an-ssd` (multiple screenshots)
- `game-analytics-with-aws-at-gdc-2014` (embedded SlideShare + YouTube)
- `advanced-game-analytics-with-aws-at-gdc-2015` (embedded video)

### Step 4: Handle images

- Download images referenced in posts to `assets/images/`
- Update markdown references to `/assets/images/filename.ext`
- Posts with known images: donut-based-security, atomic-rant-redux, replacing-macbook-hd-with-an-ssd, how-amazon-ended-up-in-san-diego, stock-market-theory, the-job-title-director-sucks
- 3 older images (replacing-macbook-ssd post) recovered from Wayback Machine since originals no longer on WordPress

### Step 5: Verify

- Run `bundle exec jekyll build` to confirm all posts compile
- Spot-check rendered output

## Front Matter Format

```yaml
---
layout: post
title: "Post Title Here"
date: YYYY-MM-DD 12:00:00 -0800
categories: category1 category2
tags: tag1 tag2
---
```

## Conversion Rules

- **Headers**: `<h2>`-`<h6>` to `##`-`######`; skip `<h1>` (title is in front matter)
- **Code blocks**: `<pre>/<code>` to fenced triple-backtick blocks with language hints (ruby, bash, text, redis)
- **Links**: `<a>` to `[text](url)`; internal nateware.com links become relative paths
- **Images**: Download locally, use `![alt](/assets/images/file.ext)`
- **Embedded media**: SlideShare/YouTube iframes pass through as raw HTML
- **Lists**: `<ul>/<li>` to `- item`; `<ol>/<li>` to `1. item`
- **Formatting**: `<strong>` to `**bold**`, `<em>` to `*italic*`, `<blockquote>` to `>`
- **Entities**: Convert `&amp;`, `&lt;`, `&gt;`, `&nbsp;` to plain characters

## Files Modified

- `_config.yml` - add permalink setting
- `_posts/` - 11 new `.md` files
- `assets/images/` - new directory with 11 downloaded images
