> So a monad is just a function that takes another function and applies a behavior to it's processing?
> Seemed interesting, but also almost too simple...

If you think individual monads are powerful, imagine the power of combining him. The blog post you read is flawed: I tried to combine seq-m (list comprehensions) with error-m. List comprehensions with free error processing. I dare you to figure out the right implementation of `bind` where the monads are combined in this stackoverflow question: [combining-maybe-and-seq-monads-confused-at-the-output](http://stackoverflow.com/questions/10059163/combining-maybe-and-seq-monads-confused-at-the-output). This is not to say that I *need* to be able to fix this type of problem - a "software surgeon" should provide a library that abstracts these things so I can say "just build a new monad which is the combination of these other two monads" - but the implementation of this is, frankly, at the very edge of my competence, and I devote about half of my waking, non-working hours to studying this stuff!

> I can show you really good examples in the DocWay Server of all kinds of bullshit code that is effectively thousands of lines that could be expressed in <10 using this sort of technique. Where I was a little confused was just in this: if it's that easy, why do people have such a hard time explaining it?

if I may say so myself, the first half (non flawed) of this article is the single best monad tutorial on the Internet. If it sounds easy, it's because I took a hundred hours of studying monads and did a good job of writing about them and hand waving over the hard parts. And you've only touched the tip of the iceberg (individual monads in isolation).

In a strongly typed language you drown in the types, I don't even get monads in Haskell yet.

> I agree that languages that don't think about that in advance have a *very* hard time in dealing with that.

there is exactly one language in the world which is expressive enough to fully implement monads without prior knowledge. that would be Lisp, due to it's macros. (Lisp macros are in a different world then, say, C macros.) Haskell built in native syntax after they realized how important monads are to haskell ("no side effect" IO), and when the next computer scientist figures out how some other ridiculously theoretical math concept applies to real life software engineering problems, he's more likely to be able to express it in Lisp, until the other languages add native syntax for it. Just like the keywords `class`, `for`, list comprehensions, `call/cc`, generators, `throw`... all of these concepts can be implemented in Lisp without native syntax.

This is the beauty of Lisp: it allows me to express *previously unthinkable thoughts*. A dumb example: the combined error-seq-m, I attempted in the blog to implement the identical behavior without monads. I took two 30 minute whacks at it. Both are incorrect, I haven't been able to get it right yet! I was able to express this with monads, I needed the guy on stackoverflow to fix my combined-bind, but the combiner should be provided by the standard library along with all the individual monads. In Clojure I say "i want to combine seq-m and error-m, OK great, now here's my business logic" and bam, it works. I did something, easily, that I am barely capable of implementing without monads. Now imagine throwing in a state-monad which lets me track if a person is asking around at other banks for loans which impacts my decision to give him a loan, and now add some continuation-monad so I can implement these bank APIs asynchronously (to talk to a database or whatever) but still use them in synchronous fashion with list comprehensions and error codes. I can build this system in a few days in Clojure. I wonder how fast I could build a TMF? I wonder if the business logic in the app would be so clear from the code, because all complexity unrelated to the requirements is abstracted away, that we can just sort of build a one-off TMF app for each individual client's requirements buy sort of just assembling the pieces like legos in maybe a thousand lines of code. Fuck the reuqirements document, the code *is* the requirements document.

This is the pipe dream, and maybe it is a pipe dream, but maybe we can get closer, which is why I care so much about functional programming.

There's a final question you implied but didn't ask:

> If monads are so powerful, why haven't they caught on in mainstream?

This is an excellent question. I probably shouldn't because I am but a learner, but I'll speculate anyway.

Monads can very rapidly blow up your stack.
