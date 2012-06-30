---
layout: post
title: Talk- use continuations for something useful
---

# {{ page.title }}

TLDR: A continuation is an object that represents the control state of a process at a particular moment (instruction pointer and call stack). We can save this state and later restore it, to manipulate the flow of execution in unusual ways. We will start with a trivial example and build up to a technique for using a blocking API, concurrently, without threads.

I gave this lightning talk at Philly RedSnake 2012.

- [slides](https://docs.google.com/present/view?id=dv9r8sv_82cn7dngcq)
- [video](http://www.ustream.tv/recorded/20611452) (starts at 1:50, sorry about the ad)
- [source](https://github.com/dustingetz/sandbox/blob/22aa8ee7ddb802423733c9a488d49f7e6129ed9a/contuations/reactor.rb)
