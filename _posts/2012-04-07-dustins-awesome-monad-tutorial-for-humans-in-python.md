# Dustin's awesome monad tutorial for humans, in Python

*[ed: draft, need to add all the citations still. Bank account example from [tumult on hackernews](http://news.ycombinator.com/item?id=1275860)]*

> A monad is a monoid in the category of endofunctors, what's the problem?
-- [James Iry](http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html)

Right. This tutorial exists because Monads are easy, they're nothing but a design pattern. It shouldn't be necessary to understand category theory and type classes to use monads to improve our code without it being scary.

Get ready to have your mind melted :D

## rigorous error handling requires lots of 'plumbing'

consider the following API:

    def get_account(name):
        if name == "Irek": return 1
        elif name == "John": return 2
        elif name == "Alex": return 3
        elif name == "Nick": return 1
        else: return None

    def get_balance(account):
        if account == 1: return 1000000
        elif account == 2: return 75000
        else: return None

    def qualified_amount(balance):
        if balance > 200000: return balance
        else: return None

this is how we might use this API:

    def get_loan(name):
        account = get_account(name)
        if not account:
            return None
        balance = get_balance(userid)
        if not balance:
            return None
        loan = get_loan(balance)
        if not loan:
            return None
        return loan

    names = ["Irek", "John", "Alex", "Nick", "Fake"]
    loans = map(get_loan, names)

    for name, loan in zip(names, loans):
        print "%s: %s" % (name, loan)

output:

    Irek: 1000000
    John: None
    Alex: None
    Nick: 1000000
    Fake: None

I see this pattern all the time in business applications. I don't want to get into a debate about exceptions, I just want to demonstrate that exceptions are not the only way to factor out the error checking.

## factor out the plumbing

Lets take a good hard look at this. How can we factor out the error checking boilerplate in our code that uses the API? Lets factor out a function `bind`:

    def bind(val, f):
        if (val):
            return f(val)
        else:
            return None

    def qualified_amount(name):
        m_name =    unit(name)
        m_account = bind(m_name, get_account)
        m_balance = bind(m_account, get_balance)
        m_loan =    bind(m_balance, get_loan)
        return m_loan

output:

    Irek: 1000000
    John: None
    Alex: None
    Nick: 1000000
    Fake: None

That wasn't very hard, was it? This is our first monad, it's called the maybe-monad.

An astute reader will observe that the first two lines of `qualified_amount` don't actually need to be wrapped by `bind`, but its harmless if they are. Stop and try to understand why. Think about how the different types line up.

Note the unit-bind-bind-bind pattern. We'll take a good hard look at that later.

## add plumbing to handle error codes

Lets change our API to return error codes alongside the result.

    def get_account(name):
        if name == "Irek": return 1, None
        elif name == "John": return 2, None
        elif name == "Alex": return 3, None
        elif name == "Nick": return 1, None
        else: return None, "No account associated with name '%s'" % name

    def get_balance(account):
        if account == 1: return 1000000, None
        elif account == 2: return 75000, None
        else: return None, "No balance associated with account #%s" % account

    def get_loan(balance):
        if balance > 200000: return balance, None
        else: return None, "Insufficient funds for loan, current balance is %s" % balance

now we have to refactor `bind` to separate the value from the error.

    def bind(mval, mf):
        value = mval[0]
        error = mval[1]

        if not error:
            return mf(value)
        else:
            return mval

the very first value passed to `bind` needs to be in `(val, error)` format now too:

    def unit(value):
        return value, None

    def get_loan(name):
        m_name =    unit(name)
        m_account = bind(m_name, get_account)
        m_balance = bind(m_userid, get_balance)
        m_loan =    bind(m_balance, get_loan)
        return m_loan

output:

    Irek: (1000000, None)
    John: (None, 'Insufficient funds for loan, current balance is 75000')
    Alex: (None, 'No balance associated with account #3')
    Nick: (1000000, None)
    Fake: (None, "No account associated with name 'Fake'")

This is called the error-monad. Again, note the pattern of binds.

# monads are a design pattern for factoring out 'plumbing'

Monads let us compose functions who need plumbing to fit together. Monads are defined as a pair of functions, `bind` and `unit`, which work together to wrap and unwrap values. A "monadic value" is the wrapped value (the error tuple, here) and a "monadic function" is a function which returns a monadic value (returns an error tuple). This is why `bind` uses the naming convention `mval` and `mf`.

Note that we didn't specifiy `unit` for the maybe-monad - this is because python objects are already nullable, so the values don't need to be wrapped. Formally we could say `def unit(x): return x`, otherwise known as the identity function.

Monads let us think about functions that require plumbing as if they didn't have any plumbing. Its interesting that almost all of the code complexity in an enterprise business application, is plumbing. hmmmm. Lets see how deep the rabbit hole goes.

## sequence monad, aka the "functional for loop"

Suppose a person has multiple accounts at multiple banks, and we want to know their loan eligibility for each. Lets extend our API to support this.

To make things simpler, lets forget about error handling for now, so we can just return normal objects without having to worry about monads until after we understand what the code will look like.

    def get_banks(name):
        if name == "Irek": return ["Bank of America", "Chase"]
        elif name == "John": return ["PNC Bank", "Wells Fargo"]
        elif name == "Alex": return ["TD Bank"]
        else: return []

    def get_accounts(bank, name):
        if   name == "Irek" and bank == "Bank of America": return [1, 2]
        elif name == "Irek" and bank == "Chase": return [3]
        elif name == "John" and bank == "PNC Bank": return [4]
        elif name == "John" and bank == "Wells Fargo": return [5, 6]
        elif name == "Alex" and bank == "TD Bank": return [7, 8]
        else: return []

    def get_balance(bank, account):
        return 250000

    def qualified_amount(balance):
        if balance > 200000: return balance
        else: return 0

and here's how we use the API. Remember, we aren't using monads, this is just a python list comprehension. Nice and elegant, list comprehensions are beautiful.

    def get_loan(name):
        return [ qualified_amount(get_balance(bank, account))
            for bank in get_banks(name)
            for account in get_accounts(bank, name) ]

    names = ["Irek", "John", "Alex", "Fred"]
    loans = map(get_loan_amount, names)
    for name, loan in zip(names, loans):
        print "%s: %s" % (name, loan)

output:

    Irek: [250000, 250000, 250000]
    John: [250000, 250000, 250000]
    Alex: [250000, 250000]
    Fred: []

so, that works just fine, but what about errors? Fred doesn't have any bank accounts, but if our API returns error tuples like we used earlier, our sexy list comprehension doesn't have the plumbing to handle error tuples. Lets see what all the error handling plumbing looks like:

First lets add errors back into our API:

    def success(val): return val, None
    def error(why): return None, why

    def get_banks(name):
        if name == "Irek": return success(["Bank of America", "Chase"])
        elif name == "John": return success(["PNC Bank", "Wells Fargo"])
        elif name == "Alex": return success(["TD Bank"])
        else: return error("No bank associated with name %s" % name)

    def get_accounts(bank, name):
        if   name == "Irek" and bank == "Bank of America": return success([1, 2])
        elif name == "Irek" and bank == "Chase": return success([3])
        elif name == "John" and bank == "PNC Bank": return success([4])
        elif name == "John" and bank == "Wells Fargo": return success([5, 6])
        elif name == "Alex" and bank == "TD Bank": return success([7, 8])
        else: return error("No account associated with (%s, %s)" % (bank, name))

    def get_balance(bank, account):
        return 250000 #TODO abstract this datamodel to have better data

    def qualified_amount(balance):
        if balance > 200000: return success(balance)
        else: return error("Insufficient funds for loan, current balance is %s" % balance)

now the business logic. Remember, this is not a monadic implementation, we want to use error codes and see what the pluming looks like for the naive implementation.

    def get_val(m_val): return m_val[0]
    def get_error(m_val): return m_val[1]

    def get_loan(name):

        m_banks = get_banks(name)
        if get_error(m_banks):
            return m_banks
        banks = get_val(m_banks)

        for bank in banks:
            m_accounts = get_accounts(bank, name)
            if get_error(m_accounts):
                return m_accounts
            accounts = get_val(m_accounts)

            for account in accounts:
                return qualified_amount(get_balance(bank, account))

    names = ["Irek", "John", "Alex", "Fred"]
    loans = map(get_loan, names)
    for name, loan in zip(names, loans):
        print "%s: %s" % (name, loan)

output:

    Irek: (250000, None)
    John: (250000, None)
    Alex: (250000, None)
    Fred: (None, 'No bank associated with name Fred')

Side note: it took me about 45 minutes to build a working `get_loan`. I kept getting lost in the error code, whether i'm looking at a list of tuples or a list of values, and what do i do if only one of the values failed but the others are valid.

That damn plumbing, we took a nice 3 line list comprehension, and adding rigorous error handling turned everything crappy, confusing, and fragile. Its no longer obvious if our business logic is even correct, and if there are bugs, its going to be more difficult than necessary to fix. When we start to see excessive plubming, we should reach for monads.

Side note: compare what we have so far to an exception-based error handling approach. With exceptions, we need to be very careful to catch them at the lowest possible level, or they will break us out of a loop and cause an error in one of the loan requests to fail all of them! Exceptions can be dangerous.

## seq_m to the rescue

For now, lets take my word for it that sequence monad is relevant to our example, and do something simpler first.

    ranks = list("abcdefgh")
    files = list("12345678")

    def chess_squares_1():
        return [ (rank, file)
                 for rank in ranks
                 for file in files ]

    assert len(chess_squares_1()) == 64
    assert chess_squares_1()[:3] == [('a', '1'), ('a', '2'), ('a', '3')]

just a straightforward list comprehension.

Sequence-monad lets us implement a list comprehension. For now I'd like to hand-wave over the implementation of this, and just observe that it does indeed give us list comprehension.

    def seq_unit(x): return [x]
    def seq_bind(mval, mf): return flatten(map(mf, mval))

    def chess_squares_2():
        # this function will use seq-m
        unit = seq_unit
        bind = seq_bind

        return bind(ranks, lambda rank:
               bind(files, lambda file:
                       unit((rank, file))))

    assert len(chess_squares_2()) == 64
    assert chess_squares_1() == chess_squares_2()

I've changed the way the bind looks now. If you squint at it in the right light, you can see it is equivalent to the step-by-step formatting we used with prior monads, with one difference: we have access to the scope of the outer 'for loop' from the inner 'for loop'. Consider the function `lambda file: unit((rank, file))`. This is the monadic function passed to bind. Scoping our binds in this fashion (using inline closures) means that the `rank` value from the outer 'for loop' (`lambda rank: ...`) will be in the rank-loop's lexical scope.

The pattern we see here, bind-bind-...-bind-unit, is the way we will write it from here on out.

This is the point where Python fails us: we need macros to fix up this syntax. There's a clear pattern which we cannot abstract further with macros, that we cann't abstract with just higher order functions*. Clojure provides a macro called `monad` which takes something that looks like a list comprehension, and re-writes it into the nested bind form we see here. If you squint at this the right way, the usage is very similar to a native list comprehension, and in fact with macros, we would be able to have equivalent syntax to list comprehensions. Oh well.

I'm hand-waving over this a bit, you don't need to deeply understand this pattern in order to be able to wield it.

\* (TODO is this even true? can you abstract this further without macros? how would it look?)

OK, now lets apply it to our banking example.

## list comprehensions with errors: combining seq_m and error_m (this one melts your mind.)

lets recap the monads we're using so far. error monad:

    def success(val): return val, None
    def error(why): return None, why

    def error_unit(x): return success(x)

    def error_bind(mval, mf):
        #print "%s: %s" % (type(mval), mval)
        assert isinstance(mval, tuple)

        error = get_error(mval)
        value = get_val(mval)
        if error:
            return mval
        return mf(value)

sequence monad:

    def seq_unit(x): return [x]

    def seq_bind(mval, mf):
        #print "%s: %s" % (type(mval), mval)
        assert isinstance(mval, list)

        return flatten(map(mf, mval))

but, we actually want to combine their behaviors, lets build a composed monad!

    # combined monad !!
    def unit(x): return error_unit(seq_unit(x))
    def bind(mval, mf): return error_bind(mval, lambda mval: seq_bind(mval, mf))

TODO: wow, monad composition is super cool! I kind of want to handwave over this, because we can abstract it, beginners don't need to understand monad combiners ("transformers"?), Clojure provides it as a standard library.

First lets try to use the old bind pattern - step by step, no closures:

    def get_loan_wrong(name):

        m_name = unit(name)
        m_banks = bind(m_name, get_banks)

        # trouble - get_accounts needs access to the wrapped value name
        m_accounts = bind(m_banks, get_accounts)

        # trouble - get_balance needs access to unwrapped values
        balance = get_balance(bank, account)
        return qualified_amount(balance)

You can see here that each 'step' we have a reference to the monadic value, but we don't have references to the unwrapped values, which is what we want. This is another example of why we need to use closures. We need the unwrapped values to be in lexical scope.

    def get_loan(name):

        return bind(unit(name), lambda name:
               bind(get_banks(name), lambda bank:
               bind(get_accounts(bank, name), lambda account:
                        unit(qualified_amount(get_balance(bank, account))))))

output:

    Irek: [[(250000, None)], None, [(250000, None)], None, [(250000, None)], None]
    John: [[(250000, None)], None, [(250000, None)], None, [(250000, None)], None]
    Alex: [[(250000, None)], None, [(250000, None)], None]
    Fred: [None, 'No bank associated with name Fred']

i gotta say, when this worked, i got really excited :)

TODO fix up the example code to support all the other errors too.

## monadic function doesn't care which monad it's using

TODO this needs to be grounded in the example

lets define `m_map` which knows about the monadic plumbing:

    def m_map(mf, mvals):
        return map(lambda x: bind(x, mf), mvals)

    names = ["Irek", "John", "Alex", "Nick", "Fake"]
    m_names = map(unit, names)
    m_loans = m_map(get_loan, m_names)

    for name, m_loan in zip(names, m_loans):
        print "%s: %s" % (name, m_loan)

`mmap` knows about bind, but it doesn't care which `bind` we're using. This version of `mmap` will work with ANY monadic function, whether we're using the maybe-monad, the error-monad, or any other monad. The specific behavior of the monad is abstracted behind the `bind` and `unit` functions. If we write the code such that a different `bind` is in scope, `mmap` still works.


## cont-m

TODO make the bank API asynchronous, but keep composability.

We can make the underlying API asynchronous and callback-oriented, and we can abstract away all the callback plumbing with continuation monad, so we still look like function composition. wait, what? wait, yes. haha.

## other monads?

- state monad - when a bank wants to know if person is asking other banks for a loan
- parallelism, laziness?
- restarting computations, in the event of a low level error that we can recover from? continuation monad again

## references

TODO clean this up.

- http://www.intensivesystems.net/tutorials/monads_101.html
- http://khinsen.wordpress.com/2009/04/22/monad-tutorial-for-clojure-programmers/
- http://stackoverflow.com/questions/9881471/map-and-reduce-monad-for-clojure-what-about-a-juxt-monad
- http://onclojure.com/2009/03/05/a-monad-tutorial-for-clojure-programmers-part-1/
- http://stackoverflow.com/questions/3870088/a-monad-is-just-a-monoid-in-the-category-of-endofunctors-whats-the-proble
- http://stackoverflow.com/questions/7796420/pattern-to-avoid-nested-try-catch-blocks
- http://lostechies.com/derickbailey/2010/09/30/monads-in-c-which-part-is-the-monad/
- "powershell cmdlets are monads" http://msdn.microsoft.com/en-us/library/ms714395%28VS.85%29.aspx
- http://stackoverflow.com/questions/412929/creative-uses-of-monads
- http://homepages.inf.ed.ac.uk/wadler/topics/monads.html
- essense of FP (haskell monad intro): http://homepages.inf.ed.ac.uk/wadler/papers/essence/essence.ps
- http://ndanger.org/blog/2008/01/16/error-handling-in-python-monads-are-too-much-for-me/
- http://james-iry.blogspot.com/2007/11/monads-are-elephants-part-4.html
- (cont monad is god monad) http://blog.sigfpe.com/2008/12/mother-of-all-monads.html
- http://www.intensivesystems.net/tutorials/monads_101.html
- http://programmers.stackexchange.com/questions/72557/how-do-you-design-programs-in-haskell-or-other-functional-programming-languages/72562#72562
- monad site:reddit.com
- http://blog.sigfpe.com/2006/08/you-could-have-invented-monads-and.html
