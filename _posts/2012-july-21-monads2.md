---
layout: post
title: monads for normal people in python and clojure
---

# {{ page.title }}

## dealing with null

here we have a series of functions with similar signatures: each takes one non-null argument, and each returns a value which may be null.

    def get_account(name):
        if name == "Irek": return 1
        elif name == "John": return 2
        else: return None

    def get_balance(account):
        if account == 1: return 1000000
        elif account == 2: return 75000
        else: return None

    def get_qualified_amount(balance):
        if balance > 200000: return balance
        else: return None

this is what business logic might look like that uses these functions:

    def get_loan(name):
        account = get_account(name)
        if not account:
            return None
        balance = get_balance(account)
        if not balance:
            return None
        loan = get_qualified_amount(balance)
        return loan

note that we're just applying these functions in order. data flow is like a unix pipe:

    name | get_account | get_balance | get_qualified_amount

which is the same thing as function composition:

    get_qualified_amount(get_balance(get_account("Irek"))) => 1000000
    get_qualified_amount(get_balance(get_account("John"))) => TypeError

both of these ways of expressing the business logic have maximum clarity, except in the case of an error, which causes something like a TypeError, NullPointerException, or segfault.

here's a way to abstract the repeated null checks. The m_ prefix on variables is to indicate that they are not a raw value, but they are a value which are specifically allowed to be null.

    def bind(v, f):
        if (v):
            return f(v)
        else:
            return None

    def qualified_amount(name):
        m_account = get_account(name)
        m_balance = bind(m_account, get_balance)
        m_loan =    bind(m_balance, get_loan)
        return m_loan

now we can get similarly clear "pipelined" business logic without explicit null checks:

    def m_pipeline(val, fns):
        m_val = val
        for f in fns:
            m_val = bind(m_val, f)
        return m_val

    >>> m_pipeline("Irek", [get_account, get_balance, get_qualified_amount])
    1000000
    >>> m_pipeline("John", [get_account, get_balance, get_qualified_amount])
    None

or with varargs for nicer syntax:

    def m_pipeline(val, *fns):
        m_val = val
        for f in fns:
            m_val = bind(m_val, f)
        return m_val


    >>> m_pipeline("Irek", get_account, get_balance, get_qualified_amount)
    1000000
    >>> m_pipeline("John", get_account, get_balance, get_qualified_amount) # => None

So that's cool right there, we can pipeline our business logic even though we need to inject error-handling plumbing between each function application. This pattern lets us compose functions that otherwise couldn't be composed because the types don't line up.

## error codes

A powerful effect of structuring our code in this way is that all the logic that packs and unpacks values into a type that may be null is not in the business logic. This means we can change the way we indicate errors to a more complex type without changing or adding complexity to the business logic.

lets refactor our API to return an error code explaining why a result is null.

    def success(val): return (val, None)
    def failure(err): return (None, err)

    def get_account(name):
        if name == "Irek": return success(1)
        elif name == "John": return success(2)
        else: return failure("No account associated with name '%s'" % name)

    def get_balance(account):
        if account == 1: return success(1000000)
        elif account == 2: return success(75000)
        else: return failure("No balance associated with account #%s" % account)

    def get_qualified_amount(balance):
        if balance > 200000: return success(balance)
        else: return failure("Insufficient funds for loan, current balance is %s" % balance)

so now our API signals success and failure via a complex return value:

    >>> get_account("Irek")
    (1, None)

    >>> get_balance(1)
    (1000000, None)

lets change `bind` to understand how to unpack our complex return value:

    def bind(mval, mf):
        value = mval[0]
        error = mval[1]

        if not error:
            return mf(value)
        else:
            return mval


    >>> bind((1, None), get_balance)
    (1000000, None)

note that `bind` requires us pass it a complex value, which it knows how to unpack into a simple value and optionally apply it to a function. When we use `chain`, the value we're passing in is typically a simple value, so chain needs to know how to turn a simple value into a complex value.

    def unit(val): return success(val)

    def m_pipeline(val, fns):
        m_val = unit(val)
        for f in fns:
            m_val = bind(m_val, f)
        return m_val


    >>> m_pipeline("Irek", get_account, get_balance, get_qualified_amount)
    (1000000, None)
    >>> m_pipeline("John", get_account, get_balance, get_qualified_amount)
    (None, 'Insufficient funds for loan, current balance is 75000')

The `unit` function for our first example, values that might be null, is trivial: python objects are already allowed to be null, so they are technically already a complex type of `value | None`. `unit` for maybe-null values is simply the identity function.

## generalized utility functions that work with all monads

A monad is a design pattern for abstracting out the plumbing necessary to compose functions of complex types. A monad is defined by the set of functions `bind` and `unit`.

We can design general functions that understand the monad interface without depending on the implementation of the monad.

Here is some code that demonstrates that the same utility functions can be reused independent of the monad they operate in. Don't worry about how this `m_chain` works, I ported it from the Clojure standard library.

    def m_chain(*fns):
        """returns a function of one argument which will perform the monadic
        composition of fns. Like m_pipeline, but chains the functions together
        for later evaluation. Ported from clojure.algo.monads/m-chain."""

        def m_chain_link(chain_expr, step):
            return lambda v: bind(chain_expr(v), step)
        return reduce(m_chain_link, fns, unit)

Here we redefine `maybe-monad` and `error-monad` from above, as well as the trivial `identity-monad`, and show that the same definition of `m_chain` can be shared with all of them. the `with monad` syntax is provided by a library and out of scope, but you can see the implementation here, all it does is put the desired `unit` and `bind` into lexical scope so we can use them.

TODO: use the get-loan example here

identity-monad:

    identity_m = {
        'bind': lambda v,f:f(v),
        'unit': lambda v:v
    }

    with monad(identity_m):
        assert m_chain(lambda x:2*x, lambda x:2*x)(2) == 8

maybe-monad:

    maybe_m = {
        'bind': lambda v,f:f(v) if v else None,
        'unit': lambda v:v
    }

    with monad(maybe_m):
        assert m_chain(lambda x:2*x, lambda x:2*x)(2) == 8
        assert m_chain(lambda x:None, lambda x:2*x)(2) == None

error-monad:

    error_m = {
        'bind': lambda mv, mf: mf(mv[0]) if mv[0] else mv,
        'unit': lambda v: (v, None)
    }

    with monad(error_m):
        success = lambda val: unit(val)
        failure = lambda err: (None, err)

        assert m_chain(lambda x:success(2*x), lambda x:success(2*x))(2) == (8, None)
        assert m_chain(lambda x:failure("error"), lambda x:success(2*x))(2) == (None, "error")
        assert m_chain(lambda x:success(2*x), lambda x:failure("error"))(2) == (None, "error")

TODO: this would be a good place to explore a few more generic methods from a monad utility library

## list-monad

I'm going to handwave over the implementation of the more complex monads, there are plenty of tutorials that explain the implementation in great detail.

list-monad:

    list_m = {
        'unit': lambda v: [v],
        'bind': lambda mv, mf: flatten(map(mf, mv))
    }

*handwaving* `bind` here does the same type of plumbing as for loops. map a function over a list. each function is of type `x -> [x]`. An example is the range function: `range(5) => [0, 1, 2, 3, 4]`.

To build a chessboard, we can't chain or pipeline our functions - we need the 'outer loop' variables to be in lexical scope to the inner loop. Here, we nest our bind expressions inside each other, so we have access to values from earlier expressions at the end of the comprehension.

    def chessboard():
        ranks = list("abcdefgh")
        files = list("12345678")

        with monad(list_m):
            return bind(ranks, lambda rank:
                   bind(files, lambda file:
                           unit((rank, file))))

    assert len(chessboard()) == 64
    assert chessboard()[:3] == [('a', '1'), ('a', '2'), ('a', '3')]

TODO: add get-loan example

## monad transformers

TODO

This is where it starts to get interesting.

List comprehension, where individual leaves can fail. Exceptions would ruin the entire list comprehension; we want the exception to be per-leaf. Show this work both ways. Show how nasty the code is without monads.

## writer-monad, reader-monad

    - deterministic logging
    - dependency injection

## state-monad

    - state machines
    - streams

## parser combinators

    (def parser-m (state-t maybe-m))
    - backtracking

## continuation-monad

    callbacks

    call-cc

    (def beat-m (state-t cont-m))

    (def async-m (seq-t maybe-t cont-m))
