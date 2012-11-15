---
layout: post
title: notes on McCarthy seminal paper on LISP
---

# {{ page.title }}

*Notes from a Philly software meetup, Functional Fall, where we studied the seminal LISP paper: [Recursive Functions of Symbolic Expressions and Their Computation by Machine, Part I](http://www-formal.stanford.edu/jmc/recursive.pdf)*

moderator: mike b

## introduction

punch line to the paper - top p2 -- "next we describe s-expr and s-finctions, describe universal fn apply"

s-expressions, is the entirety of the syntax for lisp, and m-expressions, which are syntax for function application, and we discuss how to transform between s- and m-. we discuss the apply function, which turns out to be a lisp interpreter. apply can be implemented in a few thousand lines of assembly, and that was the original lisp. note that he didn't write this then do lisp, he wrote lisp, then he reduced that to this paper. of course, he probably made a grad student actually implement lisp ;)

narrative of this paper -- starting with the math, taking just far enough to get the math into something executable. McCarthy's goal is to say, hey we actually implemented this. we can do stuff with this language.

notice that we take for granted all the things that exist now. p7. note, this is describing like the second high level language ever invented.

mccarthy takes us just far enough -- which is why lisp purists may be uncomfortble with clojure, because clojure is adding impure things, native types. [ed: pretend we were a lisp purist, why would we believe that?]

## section 2: strictly define a function

these are mathemeticians -- to us, fn is something in a program, but this is a formal definition of functions. the point of this section is to be strict aboutwhat a function is, and the notation for a function. a function is defined over part of a domain (talked about in lambda calculus -- in fact, lisp is almost the same -- basically an impl of lambda calc, with minor extensions, such as named labels for recursion). order of operations IS defined (just like in math), but its the order at which the parameters are bound, which lambda notation lets us define, to be explicit/consistent with our substutitions (thanks dan mead). lambda notation expresses explicitly how the args are substituted. matters when you have bound variables vs free variables. "unambiguous treatment of functions of functions" p7 this is just a statement meaning, how to keep your bound variables straight. ad-hoc form notation ( f = x^2+y ) doesn't achieve this explicit parameter binding. this is one difference between an algebraic expression, and a lambda calculus expression.

`predicate` is a fn whose range is T/F. means always returns either true or false. a boolean fn. you see these a lot in higher order fns in lisp. Q: what is a higher order fn? A: a function that takes a fn as an argument, or returns a fn. FIlter is the quintessential example of a predicate.
List filter(List collection, Fn predicate)

conditional expression -- we provide an interesting notation for conditionals, mike compares to lisp's `cond` -- looks like P implies E (`P -> E`) means go from left to right, evaluate them in order. if P1 then E1, else if P2 then E2, else if P3, then E3, ... else: undefined. its exactly the same as in math, and evaluated like if/elseif/else in imperative languages. evaluation of Propositions (the conditional) in order, and when we find a true proposition, evaluate his Expression (and only his expression, no other expressions).

in math:

    |x| = (x<0 -> -x, True-> x)

True-> as the last element, means `else if TRUE then return x`. note, `T` means `True`.

in python:

    def abs(x):
        if x<0: return -x
        elif True: return x

is else being undefined important? in a procedural language, else doesn't necessarily have a value. confsion about else -- have to say "else if TRUE then blah" -- its an else if -- the else, is not defined. you have to define it as else if true. [ed: so else is not even really a necessarily concept. just say 'else if T' to define it, and else is always undefined]. undefined means, in the math sense e.g. div-by-zero, not in the undefined behavior sense in C which means the program may or may not crash or have unpredictable side effects.

see recursive factorial definition in the paper. Divergence about y-combinators, which are a very tricky function in lambda calculus that allows a function to self-recurse without naming the function. It works because effectively, the y-combinator makes a copy of the function to evaluate later. mike: y combinator is cool, but it is of no practical use as a programming construct. labeling things buys you so much in simplicity of understanding. Turns out that Curry discovered/defined y-combinator, Church didn't realize it was necessary (cite?). y-combinator was added to lambda-calc after the fact.

p8 -- lambad contexts, always clear the context a variable is bound in -- mcc. never goes into this with labels. maybe this is just relevant to the implementation of this stuff, not relevant to the formalism. dan says "but label is a real function in lisp"

remember, lambda-calculus is important because it gives us vigour in reasoning and proving the behavior of a program. as discussed in prior weeks, you can't prove a program with iteration, because there's no formal calculus dscription of iteration. We can formalize recursion, so we implement iteration in terms of recursion. NB: asm has only goto, no recursion, no iteration. [ed: but asm doesn't have a theoretical math foundation, its based on machine implementation details]

We extend lambda calculus with "label notation" such we can easily define recursion. McCarthy (and we) feel lambda notation is "inadequate" or "unweildy" to define recursion without naming the functions. too impractical to reason about. y combinatro needs a macro for practical usage. so mccarthy invents this label construct.

## seciton 3: s-expressions

class of symbolic expressions, ordered pairs and lists. define 5 elementary functions, which can compose into anything? to build anything.

note, the label stuff, was pure math, it wasn't an s-expression.

definition of an s-expression, p9 middle.

s-expr is a rigorous application of logic (thanks Hunter(Texas)). lisp is an impl of lambda calc (almost), lambda calc is an application of math. function, is an algebraic function.

mccarthy introduces shorthand, so we don't have to do this dotted notation of nested tree of pairs. note, its synonymous. syntactic sugar. its a series of dotted pairs. note, its a recursive definition. mcc. is showing us how to represent a list, as an s-expression. (A,B,C,D) is sugar for (A*(B*(C*(D*NIL))))

funny how mcc says "notation is writable and somehwat readable" -- they never thought to indent! ha!

mike: side note for non-lisp programmers: everyone knows it looks funny, all the parens (length 1(1 2 3)) which is "prefix notation" for a fn call -- pass this list to this function. rules for evaluation order. evaluate inner things first. quote operator prevents the list from being evaluated.  otherwise it would apply (2,3) to the thing called 1 and say 1 is not a function.

p 10b: functions of s-expr
we introduce a second notation. M-expr is a function of s-expr. mike: this was never implemented in the original lisp, it was never necessary, it got in the way of macros. is this a formal crutch along the way, for translating the math into pure s expr? mike: no, they thought it would be a more conveiniient representation for programmers to use. Mccarthy expected M-expressions to be the dominant practical way to express a program, because it is familiar to mathemeticians, but turns out the keyboards in use those days had parens but not brackets, so s-expressions won. at time paper was written: m-expr is the term/syntax for making a fn call (thanks mike).

this is the first time in this paper we have not had function application in s-expressions. [ed: is it acruate to say, that we've defined functions with s-expr, but haven't applied them?] s-expr as a func call possible, due to special forms? (hunter)

some things aren't functions. special forms. "things built into the language" (small set of stuff) -- car, cdr, cons, apply (?), all teh c/d stuff. you can stack a/d e.g. caaaddr to get the n'th thing in the list. "special form" basically means, native functions, you can't implement these functions in lisp. special froms can do different things to the evaluation order of their arguments. ' (read as: quote-operator), just turns off evaluation of its functions. special forms "muck with" the evaluation order of their arguments.

dustin: so, m-expr can be translated into an s-expr fairly simply. so m is just sugar on top of s expr. mcc doesn't actually say this in the paper, but we think this is the case.

right now, we don't have a notation for applying fns to s-expressions, we have m-expr, but we will get there to where we can translate them.

define equality (NB: always hairy in OO languages). T if same symble, else false. "is this the same symbol or not".

car is the first thing in a dotted pair, cdr is the second (last) thing in a dotted pair. which of course can be an s-expression (a tree).

recursive s-expressions -- so far, no recursion (thus no iteration). how do we form new functions ? add conditionals and recursion. ff[x] -- stands for "first flattened"? -- takes first atom that it comes to, no matter how deeply nested recurses down the first until it gets to an atom, then takes it. so its a recursive car that finds an atom. ff[x] = [ atom[x] -> x, T -> ff[car[x]] ] recursively finds first element until we get an atom.

[ed: we skip the other interesting fn definitions, using these 5 forms. subst, etc.]

p15: equivalence of m-expr and s-expr. rules for translation.

"if youre in an m-expr, and see an s-xpr, quote the s-expr, to prevent it from being evaulated." (ed: ahhhh, so quote is critical for fn evaluation)

prefix notation (with brackets) just translated to infix notation with round braces. note, he just introduced quote out of nowhere. just defined it. but michael defined it earlier. it means, "dont evaluate this s-expr". apparently this is common, to define syntax before defining meaning, in papers like this.

p15.e.4: mathy equivalent of COND. COND is of course a special form, you can't implement a conditional with other atoms. e.g. in other lisps, 'if' is a macro around cond, which is a native fn. "normal form" vs "special form"

p16.f: apply, universal fn mike: apply taks two args: a fn, and the list of arguments to that fn. (apply fn '(1 2 3)) is like invoking fn(1,2,3).

p18: apply vs eval... this defines step by step how apply works? this is explaining step by step the implementation of apply. notes on how apply works. the list of rules of what apply does. how to implement this in assembly. of course, once you have apply in asm, you have a lisp interpreter, and we can implement apply in lisp.

note any time you pass something to a fn, you short circuit eval, but what if a && b, but b is undefined or never terminates?

when you're passing arguments into a function, you must eval the args first. discussion of what happens if you get undefined. apply, itmust be defined. this is for this impl of apply, but not fundamental to lisp (?)

still in p18.4: cond can short circuit what is the significance of this? soon as we find true P, eval E, return that as the value. cond -- doesn't eval all its args, and in this impl is literally the only thing that allows short circuiting.

ed: i had confusion here about undefined and short cirtuiting. i think it is due to thinking imperatively e.g. in python, instead of in s-expr and lisp.

p18.6: we don't understand this, is it the same dificulties as the lambda calculus?

skip 2 pages of lol.

p21: higher order functions maplist, is "map" which inspired mapreduce. fundamental functional construct. mike: is this the first definition of map? takes an s-expr, returns an s-expr.

dustn: confused about NIL -- nil is list terminator, an atom. all lists end in nil. null is predicate that returns T if nil.

ed: im missing something big with and/or and prepositions. begining of the paper we defined algebra of expressions. but how do you write an s-expression taht says "if a>b and b>c then 0, else 1"? is it part of the cond operator? is it a prefix native operator which is impl with a truth table? how can you impl AND prefix function with lisp atoms?

ed: am confused. i think its imprptant to distinguish expresson vs function vs equation just like in math. not sure how to implement propositions as s-expr, only as algebra. how does this stuff relate? how does proposition relate to s-expression?

note: "cdr" pronounced "kuh-dur"

ed: the derivative s-expr was worth looking at, todo

## section 4: impl on an ibm 704

this stuff is specific to 704. so old, lol, that all memory is register! registers are the main memory. this is how the physical stuff is put together.

this is where "car" "cdr" came from -- address, decrement. they come from the, like, bouncing around of different instructions -- the computer may not execute instructions sequentially? from the audience

p24: goes over ways to represent s-expr in memory. memory sharing (structure sharing), etc. later there became eq and eql (from dan) which this is the difference? like in jave, are the objects equal, or are the references equal?

p25: he describes using a linked list in memory to store s-expressions. this co[uld have ben represented as an arrya, more in line with the times, but they didn't -- they used linked lists, he explains why. size/number of expressions cannot be predicted in advance, cann't arrange fixed blocks to store them (array). registers put back on free storage list when no longer needed. he talks sort of about a heap here.

talks about garbage collection. 15k registers, ha. 15k words (36 bits). 6 bits to get a character (47 characters). paragraph somehwere is the first desc of a gc algorithm, ever. 26.c free storage list.

"kidn of like finding out cave men knew how to make cars"

27.d elementary s-functions in the computers atom, eq, car, cdr, cons, what about cond? atom -- each atom has a special constant address part of a register (ed: reread...) eq -- check the address equality (impl detail based on atoms being structure shared) car -- ibm 704 impl details

each register has an address part and a decrement part. decrement i guess is a pointer to the next pair.

discusses how recursive fns are implemnted, and discusses the stack. "public push-down list"

p30.6: "hipster quote" (haha, thanks mike) -- "program feature allows use of goto statements" lol

section 5: "lol someone was drinking" -- mike L-expressions, aother way to define functions of symbolic expressions which are similar to lisp. basically with strings, and with fns that build new strings, and predicates on strings, you can build lisp.

p31: "yeah, its called writing a parser"

seciton7 : ycombinator reference about recursion - extension to lambda notation (someone else made it) is label.

michael (mashion) -- formal reference to open vs closed subroutine. open: everything inlined. closed: refer to fn once in the code, all references are a goto. ed: where was this?
