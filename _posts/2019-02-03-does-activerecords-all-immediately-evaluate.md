---
layout: article
title: Does ActiveRecord's 'all' immediately execute?
date: 2019-02-03 18:45:00
categories: programming
tags:
- programming
- rails
---

In Rails, sometimes it's hard to remember which methods will immediately evaluate to SQL and which can be chained to other methods to build more complicated queries.
Let's figure out which category `all` falls into.

I've got an application with an Agency model, so let's run some queries:

```ruby
2.5.1 :003 > Agency.all
  Agency Load (1.1ms)  SELECT  "agencies".* FROM "agencies" LIMIT $1  [["LIMIT", 11]]
```

Well at first glance it looks like `all` will just evaluate. But what happens if I add a `where` clause:
```ruby
2.5.1 :004 > Agency.all.where(name: 'foo')
  Agency Load (6.6ms)  SELECT  "agencies".* FROM "agencies" WHERE "agencies"."name" = $1 LIMIT $2  [["name", "foo"], ["LIMIT", 11]]
 => #<ActiveRecord::Relation []>
```

We can see that ActiveRecord is smart enough to "chain" the `where` clause to `all`, yielding a SQL query that includes a conditional.

So whether `all` immediately evaluates depends on the context - you can "chain" additional relations (e.g. `where`), or use `all` by itself.
