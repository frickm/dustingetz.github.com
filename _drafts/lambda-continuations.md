July 17th, 6:30 - Aaron Mansheim - Continuations: Primitive, Practical, Proof

This month's meeting will be on July 17th, 6:30 at the Comcast center.

RSVP at the doodle poll here - http://www.doodle.com/7rn37rixc8fitysb

Aaron Mansheim will be presenting 'Continuations: Primitive, Practical, Proof'.
Abstract as follows.

All levels welcome - we'll reserve half our time for questions. This
talk aims to prepare you for these online tutorials:
matt.might.net/articles and www.cis.upenn.edu/~bcpierce/sf/

We'll examine an interpreter for a primitive functional programming
language (untyped lambda calculus). The interpreter is written in a
practical functional programming language (Scala).

Then we'll introduce continuations. We'll use continuations in place of
any loops, recursion, or exception handling in order to re-implement a
part of the interpreter in a functional programming language similar to
ML (Coq).

Coq facilitates writing proofs about programs. We'll demonstrate using
Coq to prove that the recursive implementation and the continuations
implementation produce the same results

## background speaker
aaron manshein
tomshon royers - query parser for a search product
continuations, how to create flow control using them. use of coq also

scala parsing combinator aexample

continuation - ability and information to continue on should something exceptional occur? dunno if this is what he said

"golena" means hen. language like ML

some stuff on the screen about some proof? wall of code. coq code. coq = "cook"

"fixed point in this ML like language" - y combinator - can use the name of the function in the function itself

q: is that a constraint of coq itself? a: yes

valexp - expressions in the lmada calculus. code traversal of the tree of the expression.

he's written a grammar which parses lambda calculus. or, its the definition of said grammar.

logical statement of equivalence of the thing we just defined

"continuation "u" is an explicit state, or "thats how ithink of continuations anyway:

this is coq ide. it keeps score as we go through the proof. we give it a goal for the proof (and subgoals)

q mbl: what are we trying to prove? version of simple recursive function is equivalent in its result to the original simple recursive function.

expression
environment = mapping of variable names to expression
c is continuation we carry around "more or less like a first aid box"

mathew might of university of utah - papers online about lambda calculus interpreters and continuations

"small step interpreter" allthough the lambda calculus is equiv to turning machine (means has halting problem), each step in the progress of the interpreter is guaranteed to complete

church encoding: encode "true", "false", numbers as lambda expressions
"if"
interesting because we haven't defined the successor function \s - turns out it doesn't matter
what we need is objects - express relationship between functions and objects

original lambda calculus - no such thing as a fn of more than one variable. you nest lambdas
