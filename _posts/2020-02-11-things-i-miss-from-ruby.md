---
layout: article
title: Things I Miss From Ruby
date: 2020-02-11 19:45:00
categories: programming
tags:
- programming
- ruby
- typescript
---

I've been doing a lot of work in Typescript lately and I have to admit it has me pining for the good ole days. I've been enjoying it but my developer efficiency has definitely suffered. The below is part complaint and part TODO list so I know what to tackle next time I'm looking for a weekend project.

# RSpec
I'm a pretty firm believer in thorough unit-testing when possible. I've been using Jest for Javascript testing, which has a nice DSL, but I still haven't quite memorized all of the outs and ins. I think solving these use cases will be a combination of doing some reading and just establishing some decent base foundations for future projects.

## Mocks
I still have to look up how to use Jest mocks. Inevitably I end up grepping for existing conventions and copying those. But it's enough of a drain on my efficiency that it's worth mentioning.

## Transactional Testing
In RSpec you can configure [transactional fixtures](https://relishapp.com/rspec/rspec-rails/docs/transactions), which gives you a nice, builtin way to manage your database state between tests. I've been manually setting up / tearing down state in my `beforeEach` blocks in Jest which is semantically equivalent but I wish it was a little more seamless.

## Shared Examples
I'm a little on the fence about this one. On the one hand you could argue that the use of shared examples is a code smell. If you need to test the same behavior in multiple places then maybe you should centralize that abstraction and test it in one place. On the other hand, sometimes I don't want to go down this wormhole or think it's too early to capture an abstraction so I just want to be able to re-leverage existing test code.

# An ORM I Like
ActiveRecord has a pretty elaborate DSL but eventually you get used to it. In fact, you can probably lean on a handful of query/relation builders and get pretty far. I've been using TypeORM, which is...fine, but has some conventions I don't like. As an example, you can't run individual migrations, only *all* of the migrations since the last run. I'm a big believer in fine-grained control over your system operations and being able to selectively run migrations is important. Granted, there are some process workarounds (e.g. write 1 migration, deploy it, run it, then repeat).

There's also an implicit assumption that you're using `synchronize`, even in production. `synchronize` will automatically try to get your database consistent with the current state of your application code as reflected in your entities. I won't bemoan anyone that chooses to do this, but again, it goes against my fine-grained control philosophy.

# Pry
Pry has been one of the biggest boons to my productivity EVER. Oh, is this test not working the way I expect? I can instantly inject a Pry and re-run the test. I *think* this will be somewhat easy to solve by leveraging the Node debugger and running the tests with the relevant command line flags.

# My Editor
Unfortunately, even with plugin support, I'm not a huge fan of using Vim for static languages. By the time I turn on all the plugins and learn all the new shortcuts I'm still only half as productive as I could be with a fancy IDE. So I've switched to using VSCode, which isn't terrible but has *just enough* problems that I get annoyed. Periodically, some magic incantation of keys+commands will trigger unbearable latency. The autocompletion is way too aggressive sometimes. And the undo stack seems to combine multiple changes sometimes, making undos unintuitive.

Anyways, I think I'll be burning down this list in the near future.
