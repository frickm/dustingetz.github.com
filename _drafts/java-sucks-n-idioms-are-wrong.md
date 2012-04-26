---
layout: post
---


> he problem with Java is the sheer amount of 'bad' code written in it

java was designed to influence code towards solving all problems with classes. the "bad code" you speak of is idiomatic java code.
Jane Street wrote about how they migrated core systems to java, abandoned it because "But somehow when coding in Java we built up a nest of classes that left people scratching their heads when they wanted to understand just what piece of code was actually being invoked when a given method was called"[1]. then, the same team moved to a functional language to great success.
Writing "good code" in java takes discipline and expertise to overcome Java's gravity towards classes. Without discipline and expertise, teams end up at the natural equlibrium you read about on thedailywtf.com. java did an awesome job of replacing C++ for the last 15 years, but there's something inherent about the design of java that doesn't scale well with the bigger, higher complexity problems we face today.
[1] http://queue.acm.org/detail.cfm?id=2038036 ~10 paragraphs in
