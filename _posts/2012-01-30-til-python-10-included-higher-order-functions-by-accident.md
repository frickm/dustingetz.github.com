---
layout: post
title: TIL python 1.0 included higher order functions by accident
---

# {{ page.title }}


Today I learned that python was not deliberately designed to be like Lisp -- it sort of happened by accident.

PG wrote \*1:

> Java, Perl, Python, you notice an interesting pattern. At least, you notice this pattern if you are a Lisp hacker. Each one is progressively more like Lisp. Python copies even features that many Lisp hackers consider to be mistakes. You could translate simple Lisp programs into Python line for line. It's 2002, and programming languages have almost caught up with 1958.

Peter Norvig also talks about similarities between Python and Lisp in \*2.

I was surprised to find that the most lisp-y bits of python -- higher order functions built in to the standard library, which implicitly defines the first ‘pythonic’ style guidelines -- do not appear to been by Guido’s design:

> Python reached version 1.0 in January 1994. The major new features included in this release were the functional programming tools lambda, map, filter and reduce. Van Rossum stated that "Python acquired lambda, reduce(), filter() and map(), courtesy of (I believe) a Lisp hacker who missed them and submitted working patches".

- 1 http://www.paulgraham.com/icad.html
- 2 http://www.norvig.com/python-lisp.html
- 3 http://en.wikipedia.org/wiki/History_of_Python#Version_1.0
