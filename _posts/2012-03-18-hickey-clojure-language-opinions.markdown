---
layout: post
title: Clojure encourages immutable style, Common Lisp does not
---

# {{ page.title }}
(from video [Expert to Expert: Rich Hickey and Brian Beckman - Inside Clojure](http://channel9.msdn.com/shows/Going+Deep/Expert-to-Expert-Rich-Hickey-and-Brian-Beckman-Inside-Clojure/), 
transcribed and slightly edited for clarity by me)

**Brian Beckman**: Now, in what sense is Clojure functional? you said at the beginning that Clojure is functional, but not pure, can you tell me exactly what you mean?

**Rich Hickey**: "Functional has been applied, you know, for a whole spectrum of things. I think there’s the haskell meaning of functional, which is sort of, the pure notion of function applied to absolutely everything in the language. And then there’s another notion of functional... lisps claimed to be functional early on, only for the simple reason that functions were first class objects or values that you can pass around. Those are the two ends. Somewhere in the middle, i think, is a style of programming that emphasizes programming with pure functions, in other words functions that are free of side effects, that operate only upon their arguments, which are values, and that only produce values as results. So that’s a pure function. Clojure emphasizes the style of programming using pure functions, where lisps, traditionally, have not. You could do it, but it was a choice. The key thing in forcing you to make that choice, is to have the core data structures be immutable, so they now can be treated as values, in a way that mutable things cannot. So Clojure emphasizes functional programming, by providing a set of immutable core data structures, including those ones that you use to represent code. They’re also persistent, which is a special characteristic we could talk about. Because of that, it encourages the functional style of programming, because you cannot change the core data structures, so you have to write a function that produces a new value as a result. but there’s no enforcement. if you write a function and that does IO, well that’s not a pure function anymore, and I don’t preclude it, and there’s no type system to preclude it, its a dynamically typed language, which just respects the programmer’s right to do whatever they please."

Dustin Getz - 19 March 2012