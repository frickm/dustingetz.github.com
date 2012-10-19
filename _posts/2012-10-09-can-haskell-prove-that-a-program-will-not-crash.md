---
layout: post
title: can Haskell prove that a program will not crash?
---

# {{ page.title }}

In haskell, can an arbitrary expression, that is not inside IO monad -- that is, it has no side effects -- be proved to not crash, because it passed the type checker?

(In practice all haskell programs have side effects, otherwise they don't do anything other than turn electricity into heat. Whatever.)

This question is interesting becauase it means we could use the type system to prove that our business logic will never crash, other than IO errors and unrecoverables like out of memory due to us choosing to evaluate our program on a finite register machine, rather than in our brain.

Answering this requires understanding the concept of ⊥, pronounced 'bottom'. From [wikipedia, bottom type](http://en.wikipedia.org/wiki/Bottom_type)

> In type theory, a theory within mathematical logic, the bottom type is the type that has no values. It is also called the zero or empty type, and is sometimes denoted with falsum (⊥). A function whose return type is bottom cannot return any value.

This is tightly related to the [Halting problem](http://en.wikipedia.org/wiki/Halting_problem):

> Given a description of an arbitrary computer program, decide whether the program finishes running or continues to run forever". ... Alan Turing proved in 1936 that a general algorithm to solve the halting problem for all possible program-input pairs cannot exist. A key part of the proof was a mathematical definition of a computer and program, what became known as a Turing machine; the halting problem is undecidable over Turing machines.

In haskell, all of the following are suitable definitions of ⊥:
 * `length [1..]` -- never terminates
 * `div 1 0` -- Exception: divide by zero
 * `error "hello"` -- a special form which terminates the program

(Note that `1.0/0.0` and `0.0/0.0` are defined, as `Infinity` and `NaN` respectively, per the IEEE spec for doubles. DivByZero in haskell is only for Integral types.)

evaluating ⊥ can do various things - for example, it might not terminate, or it might throw an exception. But it's not going to evaluate to a defined value.

Really, we're talking about partial functions. (These are a math concept, not specific to Haskell.)

> a partial function from X to Y is a function ƒ: X' → Y, where X' is a subset of X. It generalizes the concept of a function by not forcing f to map every element of X to an element of Y (only some subset X' of X). -- [Partial function](http://en.wikipedia.org/wiki/Partial_function)

If an expression in our program contains a partial function, then it is not defined over it's entire domain - or rather, it may evaluate to ⊥.

A turing complete type system is capable of expressing ⊥. Thus a turing complete type system cannot be used to generate a proof (or use the type system to prove) that an expression will not be ⊥.

[Here is a list of partial functions in haskell.](http://www.haskell.org/haskellwiki/List_of_partial_functions)

We can provide safe versions of all of these functions. For example, a `safeDiv` which returns, e.g., `Maybe`. The haskell-provided `div` does not do this for convenience - it sucks to always be unwrapping maybes. This is not to say the type lies, because a turning complete type system permits ⊥; that is, permits partial functions.

so given an expression that is not ⊥ and is not inside IO, e.g. an expression of type `int -> int`: this expression cannot "crash" unless said error condition is defined on the expression's type. But haskell allows ⊥ expressions to match this type, so this assertion isn't useful.

It does mean that we can choose to define our business exceptions in a way that impact the result type signature, which is roughly analogous to checked exceptions in Java, except more useful in practice. (Checked exceptions lose their value when all your dependencies aren't using them.)
