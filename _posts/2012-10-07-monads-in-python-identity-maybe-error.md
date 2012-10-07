---
layout: post
title: intro to monads in python. identity, maybe, error
---

Strange Loop 2012 slides in blog form. Monads by example in python - the basics.

lots of code, pay attention to the composition. here we go!

## a bank API

    def get_account(person):
        if person.name == "Alice": return Account(1)
        elif person.name == "Bob": return Account(2)
        else: return None

    def get_balance(account):
        if account.id == 1: return Balance(1000000)
        elif account.id == 2: return Balance(75000)
        else: return None

    def get_qualified_amount(balance):
        if balance.cash > 200000: return balance.cash
        else: return None


## what we want to write

    def get_loan(name):
        account = get_account(name)
        balance = get_balance(account)
        loan = get_qualified_amount(balance)
        return loan

## My boss could write this code

business logic is crystal clear. hook it up to a flowchart software or something.

    get_qualified_amount( get_balance( get_account( alice )))

    alice | get_account | get_balance | get_qualified_amount

    (-> get_account get_balance get_qualified_amount)(alice)

    alice.get_account().get_balance().get_qualified_amount()

## i love NullPointerExceptions

    def get_account(person):
        if person.name == "Alice": return Account(1)
        elif person.name == "Bob": return Account(2)
        else: return None

    >>> get_account(None)
    AttributeError: 'NoneType' object has no attribute 'id'

## what the prod code looks like :(

    def get_loan(person):
        account = get_account(person)
        if not account:
            return None
        balance = get_balance(account)
        if not balance:
            return None
        loan = get_qualified_amount(balance)
        return loan

## factor! abstract! happy!

    def bind(v, f):
        if (v):
            return f(v)
        else:
            return None

    >>> alice = Person(...)
    >>> bind(alice, get_account)
    1
    >>> bind(None, get_account)
    None

## the code we really want to write

    def bind(v,f): return f(v) if v else None

    def get_loan(name):
        m_account = get_account(name)
        m_balance = bind(m_account, get_balance)
        m_loan = bind(m_balance, get_qualified_amount) return m_loan

    >>> alice = Person(...)
    >>> get_loan(alice)
    100000
    >>> get_loan(None)
    None

## or more succinctly

    def bind(v,f): return f(v) if v else None

    def m_pipe(val, fns):
        m_val = val
        for f in fns:
            m_val = bind(m_val, f)
        return m_val

    >>> fns = [get_account, get_balance, get_qualified_amount]
    >>> m_pipe(alice, fns)
    1000000
    >>> m_pipe(dustin, fns)
    None

## lets add error handling to our API

old API

    def get_account(person):
        if person.name == "Alice": return Account(1)
        elif person.name == "Bob": return Account(2)
        else: return None

    def get_balance(account):
        if account.id == 1: return Balance(1000000)
        elif account.id == 2: return Balance(75000)
        else: return None

    def get_qualified_amount(balance):
        if balance.cash > 200000: return balance.cash
        else: return None

new API

    def get_account(person):
        if person.name == "Alice": return (Account(1), None)
        elif person.name == "Bob": return (Account(2), None)
        else: return (None, "No acct for '%s'" % person.name)

    >>> get_account(alice)
    (Account(1), None)
    >>> get_account(dustin)
    (None, "No acct for 'Dustin'")

    def get_balance(account):
        if account.id == 1: return (Balance(1000000), None)
        elif account.id == 2: return (Balance(75000), None)
        else: return (None, "No balance, acct %s" % account.id)

    def get_qualified_amount(balance):
        if balance.cash > 200000: return (balance, None)
        else: return (None, "Insufficient funds: %s" % balance.cash)

## what happens if you compose the functions now?

    def get_loan(name):
        account = get_account(name)
        balance = get_balance(account)
        loan = get_qualified_amount(balance)
        return loan

    >>> get_loan(alice)
    TypeError # can't compose anymore :(

didn't unpack the tuples, so we can't compose the functions anymore

    >>> get_qualified_amount( get_balance( get_account( alice )))
    TypeError # can't compose anymore :(

## fix bind to unpack the tuples for us

    def bind(mval, mf):
        value = mval[0]
        error = mval[1]
        if not error:
            return mf(value)
        else:
            return mval

    >>> mval = (1, None)
    >>> bind(mval, get_balance)
    (100000, None)

    >>> mval = (None, "insuf funds")
    >>> bind(mval, get_balance)
    (None, "insuf funds")

## business logic didn't change

original business logic is the composition of functions.

    def bind(mv, mf): mf(mv[0]) if mv[0] else mv
    def unit(v): return (v, None)

literally copy paste the same business logic

    def get_loan(name):
        m_account = get_account(name)
        m_balance = bind(m_account, get_balance)
        m_loan =    bind(m_balance, get_qualified_amount) return m_loan

    >>> get_loan(alice)
    (1000000, None)

or the same business logic written as a pipe

    def m_pipe(val, *fns):
        m_val = unit(val)
        for f in fns:
            m_val = bind(m_val, f)
        return m_val


    >>> m_pipe(alice, get_account, get_balance, get_qualified_amount)
    (1000000, None)

or generated from the same flow chart software

    alice | get_account | get_balance | get_qualified_amount

## here be monads

    # maybe monad
    def bind(mv, mf): return mf(mv) if mv else None
    def unit(v): return v

    # error monad
    def bind(mv, mf): mf(mv[0]) if mv[0] else mv
    def unit(v): return (v, None)

## list comprehensions

visualize what this returns

    >>> ranks = list("abcdefgh")
    >>> files = list("12345678")

    >>> [(x,y) for x in ranks for y in files]

    [('a', '1'), ('a', '2'), ('a', '3') ... ]

how can you implement the list comprehension? how does it work?

    # list monad
    def unit(v): return [v]
    def bind(mv,mf): return flatten(map(mf, mv))

its actually not important that you understand why bind is implemented like this. you can assume that it has been supplied by a library written by someone who cares. it's the subject of many other monad tutorials if you are interested.

    >>> bind( ranks,
              lambda rank: bind( files,
                                 lambda file: unit( (rank, file) )))

    [('a', '1'), ('a', '2'), ('a', '3') ... ]

lets rearrange whitespace to make it nicer to read

    >>> bind( ranks,    lambda rank:
        bind( files,    lambda file:
              unit((rank, file))))

picture some macros that hide the binds and you will recognize list comprehension syntax in python

    >>> [ (x,y) for x in ranks for y in files]

can you see it? or write it like this:

    >>> [ (x,y)
          for x in ranks
          for y in files]

and in clojure

    repl> (for [rank (seq “abcdefgh”)
                file (seq “12345678”)]
             [rank, file])

clojure of course has real macros, so we can really implement for comprehensions this way and they will really work. go ahead and try it.

you now know enough to add list comprehensions to javascript, though you need to use the bind syntax because we don't have macros.

each bind returns a new monadic value, which is suitable to be passed to bind again. note that using anonymous functions inline like this leaves the unwrapped variables in lexical scope, which means we can use them in later nested bind expressions. this pattern of chaining together binds is called a monad comprehension.

## the simplest monad: identity monad

    # identity monad
    def bind(mv,mf): return mf(mv)
    def unit(v): return v

lets get familiar with monad comprehension syntax and use it with identity monad

    >>> bind( 7,
              lambda x: bind( 3,
                              lambda y: unit(x + y)))
    10

    >>> bind( 7,    lambda x:
        bind( 3,    lambda y:
              unit(x + y)))
    10

clojure provides native syntax for this monad too, it's called `let`. go figure.

    repl> (let [x 7
                y 3]
             (+ x y)
    10

## so wtf is a monad

* a design pattern for composing functions that have incompatible types, but can still logically define composition
* by overriding function application
* to increase modularity and manage accidental complexity
* by making the business logic read like a flow chart out of your boss's powerpoint slides

building a language that lets you write a requirements document that is both human and machine readable is something that a lot of people are interested in doing. if only they studied lambda calculus...
