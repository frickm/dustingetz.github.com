---
layout: post
title: if monads are so awesome, why don't more people use them?
---


# {{ page.title }}

The below quotes are a response, from a friend, to my prior post on monads: (Dustin’s awesome monad tutorial for humans, in Python)[http://www.dustingetz.com/2012/04/07/dustins-awesome-monad-tutorial-for-humans-in-python.html]

> So a monad is just a function that takes another function and applies a behavior to it's processing?
> Seemed interesting, but also almost too simple...

you've only seen error-monad, you haven't seen any of the more powerful ones, nor have you seen the compounding power of combining them. when combining them you express an amazing amount of functionality in 10 extremely dense lines of code, but if you (or some other expert) can get it right, people can use your abstraction without understanding why it works. this is of course the point of an abstraction.

As an example of the difficulty (and power) of combining monads, the blog post you read is flawed when i try to use error-monad within sequence-monad. I dare you to figure out the right implementation of `bind` where the monads are combined in this stackoverflow question: [combining-maybe-and-seq-monads-confused-at-the-output](http://stackoverflow.com/questions/10059163/combining-maybe-and-seq-monads-confused-at-the-output). This is not to say that I *need* to be able to fix this type of problem, it can be provided by a library, and is indeed provided by Clojure, Haskell, Scala.

> I can show you really good examples in the product of all kinds of bullshit code that is effectively thousands of lines that could be expressed in <10 using this sort of technique. Where I was a little confused was just in this: if it's that easy, why do people have such a hard time explaining it?

if I may say so myself, the first half (non flawed) of this article is the single best monad tutorial on the Internet. If it sounds easy, it's because I distilled 30 other peope's blog post and reddit comments to their essence, and did a good job of writing about them and hand waving over the hard parts.

> I agree that languages that don't think about that in advance have a *very* hard time in dealing with that.

there is exactly one language in the world which is expressive enough to fully implement monads without prior knowledge. that would be Lisp, due to it's macros. (Lisp macros are in a different world then, say, C macros.) Haskell built in native syntax after they realized how important monads are to haskell ("no side effect" IO), and when the next computer scientist figures out how some other ridiculously theoretical math concept applies to real life software engineering problems, he's more likely to be able to express it in Lisp, until the other languages add native syntax for it. Just like the keywords `class`, `for`, list comprehensions, `call/cc`, generators, `throw`... all of these concepts can be implemented in Lisp without native syntax.

This is the beauty of Lisp: it allows me to express *previously unthinkable thoughts*. A dumb example: the combined error-seq-m, I attempted in the blog to implement the identical behavior without monads. I took two 30 minute whacks at it. Both are incorrect, I haven't been able to get it right yet! I was able to express this with monads, I needed the guy on stackoverflow to fix my combined-bind, but the combiner should be provided by the standard library along with all the individual monads. In Clojure I say "i want to combine seq-m and error-m, OK great, now here's my business logic" and bam, it works. I did something, easily, that I am barely capable of implementing without monads. Now imagine throwing in a state-monad which lets me track if a person is asking around at other banks for loans which impacts my decision to give him a loan, and now add some continuation-monad so I can implement these bank APIs asynchronously (to talk to a database or whatever) but still use them in synchronous fashion with list comprehensions and error codes. I can build this system in a few days in Clojure. I wonder how fast I could build a TMF? I wonder if the business logic in the app would be so clear from the code, because all complexity unrelated to the requirements is abstracted away, that we can just sort of build a one-off TMF app for each individual client's requirements buy sort of just assembling the pieces like legos in maybe a thousand lines of code. Fuck the reuqirements document, the code *is* the requirements document.

This is the pipe dream, and maybe it is a pipe dream, but maybe we can get closer, which is why I'm choosing to study functional programming full time for the forseeable future.

There's a final question you implied but didn't ask:

> If monads are so powerful, why haven't they caught on in mainstream?

This is an excellent question. I probably shouldn't because I am but a learner, but I'll speculate anyway.

Monads can very rapidly blow up your stack. To see this think about win32 re-entrant code. monads are implemented with macros on top of deeply nested closures. Tail-call stack elimination is basically required, and rewinding the stack in a language like javascript or python requires either clever hacks with exceptions (slow), or settimeout(0) in the context of an event loop (what if you don't want an event loop). also the bind-bind-bind-unit pattern isn't great to work with, you need macros to abstract it, lisp is the only language in the world right now who can express monads cleanly without native syntax for them. haskell has native syntax, scala resorts to creative operator overloading.

the non-technical reason is that, as far as i can tell, nobody applied monads-the-theoretical-math-concept to software until haskell's research into functional purity made them important, as well as there never being a mainstream language capable of expressing them until recent interest in functional programming via scala and haskell. Common Lisp *could* express monads but CL never mainstreamed for reasons out-of-band of this discussion. Now monads are leaking into other languages, because if you're expert enough to weild them, they make really hard problems easier. for example, parsing is ideally suited to monads (compose and combine different types of plumbing to parse a contextual grammar), as well as other difficult things like implementing continuations, exceptions, generators, list comprehensions.
