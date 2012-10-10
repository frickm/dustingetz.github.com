---
layout: post
title: can Haskell prove that a program will not crash?
---

# {{ page.title }}

In haskell, can an arbitrary expression, that is not inside IO monad, be proved to not crash, because it passed the type checker?

(In practice all haskell programs are in IO, otherwise they don't do anything other than turn electricity into heat. Whatever.)

This question is interesting becauase it means we can use the type system to prove that our business logic will never crash, other than IO errors and unrecoverables like out of memory due to us choosing to evaluate our program on a finite register machine, rather than in our brain.

Answering this requires understanding the concept of ⊥, pronounced 'bottom'. From [wikipedia, bottom type](http://en.wikipedia.org/wiki/Bottom_type)

> In type theory, a theory within mathematical logic, the bottom type is the type that has no values. It is also called the zero or empty type, and is sometimes denoted with falsum (⊥). A function whose return type is bottom cannot return any value.

This is tightly related to the [Halting problem](http://en.wikipedia.org/wiki/Halting_problem):

> Given a description of an arbitrary computer program, decide whether the program finishes running or continues to run forever". ... Alan Turing proved in 1936 that a general algorithm to solve the halting problem for all possible program-input pairs cannot exist. A key part of the proof was a mathematical definition of a computer and program, what became known as a Turing machine; the halting problem is undecidable over Turing machines.

In haskell, all of the following are suitable definitions of ⊥:
 * `length [1..]` -- never terminates
 * `div 1 0` -- Exception: divide by zero
 * `error "hello"` -- a special form which terminates the program

evaluating ⊥ has undefined behavior but GHC will try to detect it and quit to be nice.

(Note that `1.0/0.0` and `0.0/0.0` are defined, as `Infinity` and `NaN` respectively, per the IEEE spec for doubles. DivByZero in haskell is only for Integral types.)

Thus it is not possible to generate a proof (or use the type system to prove) that an expression will not be ⊥ due to turing completeness.

We can provide `safeDiv` which returns, e.g., `Maybe`, but the haskell-provided `div` does not do this for convenience - it sucks to always be unwrapping maybes. This is not to say the type lies, because a turning complete type system permits ⊥.

so given an expression that is not ⊥ and is not inside IO, this expression cannot "crash" unless said error condition is defined on the expression's type.

so in effect, an expression of type `int -> int`, will only crash in the same ways that math can 'crash' on pencil and paper.

Which means that if we're reasonably confident that our program isn't ⊥ (do you really worry about the halting problem in your business application?), **if your program passes the type checker, it will not crash. ever.**
