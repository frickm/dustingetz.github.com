---
layout: post
title: implement python generators without yield
---

# {{ page.title }}

here are some [python examples of how to actually implement generators](https://github.com/dustingetz/sandbox/blob/master/etc/lazy.py) as if python did not provide syntactic sugar for them (or in a language without native syntax, like javascript). snippets from that link below. This code runs, type it into a repl.

## fib as a python generator:

    from itertools import islice

    def fib_gen():
        a, b = 1, 1
        while True:
            yield a
            a, b = b, a + b

    assert [1, 1, 2, 3, 5] == list(islice(fib_gen(), 5))

## lexical closures

Python doesn't have proper lexical closures, but Python3 has the `nonlocal` keyword, which is close. Languages with proper lexical scope don't need a special keyword.

    ## python3, untested
    def fib_gen4():
        a, b = 0, 1
        def _():
            nonlocal a, b   # lift vars to enclosing scope
            a, b = b, a+b
            return a
        return _

    assert [1,1,2,3,5] == ftake(fib_gen2(), 5)


## using object closures instead of generators

Because [ClosuresAndObjectsAreEquivalent](http://c2.com/cgi/wiki?ClosuresAndObjectsAreEquivalent). This is really the same thing as the above.

    def ftake(fnext, N):
        "take the next N elements from a sequence"
        return [fnext() for _ in xrange(N)]

    class fib_gen3:
        def __init__(self):
            self.a, self.b = 1, 1

        def __call__(self):
            r = self.a
            self.a, self.b = self.b, self.a + self.b
            return r

    assert [1,1,2,3,5] == ftake(fib_gen3(), 5)


## python functions are just objects

    def fib_gen2():
        #funky scope due to python2.x workaround
        #for python 3.x use nonlocal
        def _():
            _.a, _.b = _.b, _.a + _.b
            return _.a
        _.a, _.b = 0, 1  # this state is shared across all calls to the fn
        return _

    assert [1,1,2,3,5] == ftake(fib_gen2(), 5)
