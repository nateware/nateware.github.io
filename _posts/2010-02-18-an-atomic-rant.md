---
layout: post
title: "☢️ An Atomic Rant"
date: 2010-02-18 12:00:00 -0800
author: "Nate Wiger"
categories: redis technology
tags: atomicity redis ruby
---

You are probably not handling atomic operations properly in your app, and probably have some nasty lurking race conditions. The worst part is these will get worse as your user count increases, are difficult to reproduce, and usually happen in your most critical pieces of code. (And no, your unit tests can't catch them either.)

**Spoiler:** If you're part of the ADHD generation and want to skip learning and go straight to the punchline, use [Redis](http://redis.io) and [redis-objects](http://github.com/nateware/redis-objects) for all your atomic data needs.

## Brush Up Your Resume

Let's assume you're writing an app to enable students to enroll in courses. You need to ensure that no more than 30 students can sign up for a given course. In your enrollment code, you have something like this:

```ruby
@course = Course.find(1)
if @course.num_students < 30
  @course.course_students.create!(:student_id => 101)
  @course.num_students += 1
  @course.save!
else
  # course is full
end
```

You're screwed. You now have 32 people in your 30 person class, and you have no idea what happened.

"Well no duh," you're saying, "even the ActiveRecord docs mention locking, so I'll just use that."

```ruby
@course = Course.find(1, :lock => true)
if @course.num_students < 30
  # ...
```

Nice try, but now you've introduced other issues. Any other piece of code in your entire app that needs to update anything about the course -- maybe the course name, or start date, or location -- is now serialized. If you need high concurrency, you're screwed (still).

You think, "ah-ha, the problem is having a separate counter!"

```ruby
@course = Course.find(1)
if @course.course_students.count < 30
  @course.course_students.create!(:student_id => 101)
else
  # course is full
end
```

Nope. Still screwed.

## The Root Down

It's worth understanding the root issue, and how to address it.

Race conditions arise from the difference in time between _evaluating_ and _altering_ a value. In our example, we fetched the record, then checked the value, then changed it. The more lines of code between those operations, and the higher your user count, the bigger the window of opportunity for other clients to get the data in an inconsistent state.

Sometimes race conditions don't matter in practice, since often a user is only operating on their own data. This has a race condition, but is probably ok:

```ruby
@user = User.find(params[:id])
@post = Post.create(:user_id => @user.id, :title => "Whattup")
@user.total_posts += 1  # update my post count
```

But this would be problematic:

```ruby
@blog = Blog.find(params[:id])
@post = Post.create(:blog_id => @blog.id, :title => "Whattup")
@blog.total_posts += 1  # update post count across all users
```

As multiple users could be adding posts concurrently.

In a traditional RDBMS, you can increment counters atomically (but not return them) by firing off an update statement that self-references the column:

```sql
update users set total_posts = total_posts + 1 where id = 372
```

You may have seen ActiveRecord's `increment_counter` class method, which wraps this functionality. This solves half the problem of updating the counters atomically. But this has the significant side effect that your object is no longer in sync with the DB, so you get other issues:

```ruby
@blog = Blog.find(params[:id])
Blog.increment_counter :total_posts, @blog.id
if @blog.total_posts == 1000
  # the 1000th poster - award them a gold star!
```

The DB says 1000, but your `@blog` object still says 999, and the right person doesn't get their gold star. Sad faces all around.

## A Better Way

Bottom line: **Any operation that can alter a value must also return that value in the same operation for it to be atomic.** If you do a separate get then set, or set then get, you're open to a race condition. There are only a few systems that support an "increment and return" type operation, and [Redis](http://redis.io) is one of them (Oracle sequences are another, and Postgres supports "update returning").

When you think of the specific things that you need to ensure, many of these will reduce to numeric operations:

- Ensuring there are no more than 30 students in a course
- Getting more than 2 but less than 6 people in a game
- Keeping a chat room to a max of 50 people
- Correctly recording the total number of blog posts
- Only allowing one piece of code to reorder a large dataset at a time

All except the last one can be implemented with counters. The last one will need a carefully placed lock.

The best way I've found to balance atomicity and concurrency is, for each value, actually create two counters:

- A counter you base logic on (eg, `slots_taken`)
- A counter users see (eg, `current_students`)

The reason you want two counters is you'll need to change the value of the logic counter first, before checking it, to address any race conditions. This means the value can get wonky momentarily (eg, there could be 32 `slots_taken` for a 30-person course). This doesn't affect its function -- indeed, it's part of what makes it work -- but does mean you don't want to display it.

So, taking our Course example:

```ruby
class Course < ActiveRecord::Base
  include Redis::Objects

  counter :slots_taken
  counter :current_students
end
```

Then:

```ruby
@course = Course.find(1)
@course.slots_taken.increment do |val|
  if val <= @course.max_students
    @course.course_students.create!(:student_id => 101)
    @course.current_students.increment
  end
end
```

Race-condition free. Why? Because we're checking the direct result of the increment operation against a value. The set and get operations are one and the same, which is the crucial piece. If that code block returns false, the counter is rewound, and no animals were harmed in this atomic op.

Then, due to the `current_students` counter, your views get consistent information about the course, since it will only be incremented on success. There is still a race condition where `current_students` could be less than the real number of `CourseStudent` records, but since you'll be displaying these values in a view (after that block completes) you shouldn't see this manifest in real-world usage.

Now you can sleep soundly, without fear of getting fired at 3am via an angry phone call from your boss. (At least, not about this...)
