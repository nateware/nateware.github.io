---
layout: post
title: "✍🏼 Converting from Wordpress to GitHub Pages with Claude"
date: 2026-02-09 12:00:00 -0800
categories:
tags:
---

I've used Wordpress for many many years, but I got to the point where I was tired of paying $100/year for a blog that gets a few hundred visits per month. I've played around with static site generators like [Jekyll](https://jekyllrb.com/) but have always dreaded the amount of time it would take to convert my existing Wordpress blog. Then I realized one evening: Why would I do this myself in the age of AI? Enter Claude Code.

## The Prompt

Converting a blog is a major PITA. You have to download the existing Wordpress articles, port them to Markdown, change all the image links, make sure code is formatted correctly, etc, etc. Certainly I could get Claude to do this for me? What I didn't expect was it would do a much better job than me in a fraction of the time.

I followed the [GitHub Pages Quickstart](https://docs.github.com/en/pages/quickstart) to setup a fresh repo for my new blog. At this point it was just a skeleton with no content. Then I opened it in Claude Code and issued this prompt:

> this is a personal blog written in jekyll. i want to convert my existing wordpress blog into jekyll posts. my blog is located at: nateware.com . for each post on my blog, please create a corresponding markdown file. respect any of the formatting in my existing blog - headers, code segments, links, etc. for each existing blog post, create a new jekyll markdown file in the \_posts directory formatted as YYYY-MM-DD-[slug from existing blog]. For example my blog article located at https://nateware.com/2010/02/18/an-atomic-rant/ should be downloaded, converted to markdown, and saved as \_posts/2010-02-18-an-atomic-rant.md. Create a plan

Claude then generated a plan so that for each post it would:

1. Fetch the full page content via WebFetch
2. Extract title, categories, and tags
3. Convert HTML body to clean Markdown
4. Write file to `_posts/YYYY-MM-DD-slug.md`

It even added this step that I didn't ask for:

- Add `permalink: /:year/:month/:day/:title/` to `_config.yml` to match existing WordPress URL structure for SEO/link preservation

When it came to images, Claude came up with this strategy:

- Download images referenced in posts to `assets/images/`
- Update markdown references to `/assets/images/filename.ext`
- 3 older images (replacing-macbook-ssd post) recovered from Wayback Machine since originals no longer on WordPress **(WHAT?!)**

The last one blew my mind - Claude was "smart" enough to detect that an image link was broken, and rather than giving up, it knew to go to the wayback machine and find it from a previous snapshot. That's crazy. I don't think I would have even thought of that.

## The Results

You're seeing it for yourself. Other than editing this blog post, I did no coding or conversion steps myself. Everything was automated via Claude Code. You can view the full repo at: [nateware.github.io](https://github.com/nateware/nateware.github.io)

Up next is writing a full app "hands off the wheel" style. More to come...
