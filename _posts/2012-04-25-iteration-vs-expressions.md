---
layout: post
title: higher-level iteration
---

# {{ page.title }}

this is inspired by and loosely based on [Ned Batchedler's talk about iteration in python](http://nedbatchelder.com/text/iter.html).

once upon a time, there was a while loop.

    items = [1, 2, 3, 4, 5]
    dev visit(item): print item

    i = 0
    while i < len(items]:
        item = items[i]
        visit(item)

while loops suck because its super easy to screw up and loop forever. so we abstracted, and then there where for loops.

    for i in range(len(items)):
        item = items[i]
        visit(item)

for loops also suck, because its easy to have off by one errors, so we abstracted higher, and then there was foreach (aka iteration).

    for item in items:
        visit(item)

iteration also sucks, because there is more code about iteration than there is for the business logic. so we abstracted higher, and then there was

    map(visit, items) #even higher level than iteration

    def is_odd(item): return item % 2 == 0
    assert [1,3,5] == filter(is_odd, items)

    def square(item): return item * item
    assert [1, 4, 9, 16, 25] == map(square, items)

    def add(accumulator, item): return accumulator + item
    assert 15 == reduce(add, items)

notice we're doing harder things in less lines of code, and everything is obviously correct (no bugs) to a glance.

each of these can be implemented on top of foreach, and i haven't yet seen a loop that can't be decomposed into a combination of map, filter, reduce. also note that filter/map/reduce are expressions, not statements, so they are more "mathy" and thus easier to reason about.

ggchappell from hackernews brings up an interesting example:

> I agree with your basic point: that thinking of loops in terms of high-level loop primitives, is a Good Thing. but, the "zip" operation gives loops that seem to me to be unable to be decomposed into map, filter, reduce. E.g., how would you print a file with line numbers using only map, filter, reduce?

reflexively i bang out one answer:

    def zip(*seqs): return map(lambda *items: items, *seqs)
    assert [(1, 'a'), (2, 'b'), (3, 'c')] == zip([1,2,3],['a','b','c'])

ggchappell brings up an excellent point:

> Well, strictly speaking, you have answered my objection. But I guess I was thinking in Haskell. If Python's map takes multiple iterables, then, yes, zip is a special case. But writing zip by mapping a function without side effects, using a "map" that works only on a single iterable (as in Haskell) strikes me as a rather different thing. So I suppose that the truth of "this loop can be written using map" depends on exactly what one means by "map".

that's a great objection. you can't implement a map that works with multiple sequences without dropping down an abstraction layer and using iteration. is there another way?

    def head(seq): return seq[0]
    def rest(seq): return seq[1:]
    def transpose(mtx):
        a = map(head, mtx)
        b = map(rest, mtx)
        return [a] + ([transpose(b)] if b[0] else [])

    def zip2(xs, ys): return transpose((xs, ys))
    def zipN(*seqs): return transpose(seqs)

this works but is sort of icky in python, any ideas? this is also where functional python starts to fall apart - no tail call recursion, and default data structures aren't persistent. based on a [haskell solution](http://stackoverflow.com/questions/2578930/understanding-this-matrix-transposition-function-in-haskell) which is nicer.

challenge: find a loop that can't be decomposed into filter, map, reduce. is it provable that we can decompose an arbitrary loop into high level transformations? is it generally agreed that for all loops, simple and complex, the higher level version is easier to prove correct, less vulnerable to bugs? (given adequate language support, this will never be idiomatic python.)
