# Dustin's awesome monad tutorial for humans, in Python

*[ed: draft, need to add all the citations still. Bank account example from [tumult on hackernews](http://news.ycombinator.com/item?id=1275860)]*

> A monad is a monoid in the category of endofunctors, what's the problem?
-- [James Iry](http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html)

Right. This tutorial exists because Monads are easy, they're nothing but a design pattern. It shouldn't be necessary to understand category theory and type classes to use monads to improve our code without it being scary.

## rigorous error handling requires lots of 'plumbing'

consider the following API:

    def userid_from_name(name):
        if name == "Irek": return 1
        elif name == "John": return 2
        elif name == "Alex": return 3
        elif name == "Nick": return 1
        else: return None

    def balance_from_userid(userid):
        if userid == 1: return 1000000
        elif userid == 2: return 75000
        else: return None

    def balance_qualifies_for_loan(balance):
        if balance > 200000: return balance
        else: return None

this is how we might use this API:

    def name_qualifies_for_loan(name):
        userid = userid_from_name(name)
        if not userid:
            return None
        balance = balance_from_userid(userid)
        if not balance:
            return None
        loan = balance_qualifies_for_loan(balance)
        if not loan:
            return None
        return loan

    names = ["Irek", "John", "Alex", "Nick", "Fake"]
    qualified = map(name_qualifies_for_loan, names)
    for name, qualified in zip(names, qualified):
        print "%s: %s" % (name, qualified)

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

    def name_qualifies_for_loan(name):
        m_name =    unit(name)
        m_userid =  bind(m_name, userid_from_name)
        m_balance = bind(m_userid, balance_from_userid)
        m_loan =    bind(m_balance, balance_qualifies_for_loan)
        return m_loan

output:

    Irek: 1000000
    John: None
    Alex: None
    Nick: 1000000
    Fake: None

That wasn't very hard, was it? This is our first monad, it's called the maybe-monad.

## add plumbing to handle error codes

Lets change our API to return error codes alongside the result.

    def userid_from_name(name):
        if name == "Irek": return 1, None
        elif name == "John": return 2, None
        elif name == "Alex": return 3, None
        elif name == "Nick": return 1, None
        else: return None, "No account associated with name '%s'" % name

    def balance_from_userid(userid):
        if userid == 1: return 1000000, None
        elif userid == 2: return 75000, None
        else: return None, "No balance associated with account #%s" % userid

    def balance_qualifies_for_loan(balance):
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

    def name_qualifies_for_loan(name):
        m_name =    unit(name)
        m_userid =  bind(m_name, userid_from_name)
        m_balance = bind(m_userid, balance_from_userid)
        m_loan =    bind(m_balance, balance_qualifies_for_loan)
        return m_loan

output:

    Irek: (1000000, None)
    John: (None, 'Insufficient funds for loan, current balance is 75000')
    Alex: (None, 'No balance associated with account #3')
    Nick: (1000000, None)
    Fake: (None, "No account associated with name 'Fake'")

This is called the error-monad. Are you starting to see the pattern?

Monads are a design pattern for composing functions who need plumbing to fit together. Monads are defined as a pair of functions, `bind` and `unit`, which work together to wrap and unwrap values. A "monadic value" is the wrapped value (the error tuple, here) and a "monadic function" is a function which returns a monadic value (returns an error tuple). This is why `bind` uses the naming convention `mval` and `mf`.

Note that we didn't specifiy `unit` for the maybe-monad - this is because python objects are already nullable, so the values don't need to be wrapped. Formally we could say `def unit(x): return x`, otherwise known as the identity function.

Monads let us think about functions that require plumbing as if they didn't have any plumbing. Its interesting that almost all of the code complexity in an enterprise business application, is plumbing. hmmmm. Lets see how deep the rabbit hole goes.

## seq monad

Suppose a person has multiple accounts at multiple banks, and we want to know their loan eligibility for each. Lets extend our API to support this. To make things simpler, lets forget about error handling for now.

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

and here's how we use the API. Note, we aren't using monads, this is just a python list comprehension.

    def get_loan_amount(name):
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

API first:

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


## list comprehensions with errors: combining seq_m and error_m (this one melts your mind.)

(TODO: figure out an example that doesn't need lexical scope to work?)

    # error monad
    def error_unit(x): return success(x)

    def error_bind(mval, mf):
        """unpack monadic value from error monad into (val, error).
        invoke mf(val), but only when there is not an error.
        returns a result in error-monad."""
        error = get_error(mval)
        value = get_val(mval)
        if error:
            return mval
        return mf(value)

    def flatten(listOfLists):
        """Flatten one level of nesting
        http://docs.python.org/library/itertools.html#recipes"""
        return chain.from_iterable(listOfLists)

    # seq monad
    def seq_unit(x): return [x]

    def seq_bind(mval, mf):
        """unpack a list of values, invoking mf(val) for each value. Each result is
        a list of values, collect these lists into a single list.
        returns a list of results in seq-monad."""
        return flatten(map(mf, mval)) # not provided by python

    # combined monad !!
    def unit(x): return error_unit(seq_unit(x))
    def bind(mval, mf): return error_bind(mval, lambda mval: seq_bind(mval, mf))

TODO: wow, monad composition is super cool! I kind of want to handwave over this, because we can abstract it, beginners don't need to understand monad combiners ("transformers"?), Clojure provides it as a standard library.

    def get_loan_wrong(name):

        m_name = unit(name)
        m_banks = bind(m_name, get_banks)

        # trouble - get_accounts needs access to the wrapped value name
        m_accounts = bind(m_banks, get_accounts)

        # trouble - get_balance needs access to unwrapped values
        balance = get_balance(bank, account)
        return qualified_amount(balance)

what if we do it with closures? try again:

        balance = bind(unit(name), lambda name:
                  bind(get_banks(name), lambda bank:
                  bind(get_accounts(bank, name), lambda account:
                           get_balance(bank, account))

        return qualified_amount(balance)

    def get_loan(name):

        return bind(unit(name), lambda name:
               bind(get_banks(name), lambda bank:
               bind(get_accounts(bank, name), lambda account:
                        unit(qualified_amount(get_balance(bank, account))))))


this is the point where Python fails us: we need macros to fix up this syntax. There's a clear pattern which we cannot abstract further with just higher order functions! wow, powerful stuff. If you squint at this the right way, the usage is very similar to a native list comprehension, and in fact with macros, we would be able to have equivalent syntax to list comprehensions. Oh well.

anyway, the output:

    Irek: [[(250000, None)], None, [(250000, None)], None, [(250000, None)], None]
    John: [[(250000, None)], None, [(250000, None)], None, [(250000, None)], None]
    Alex: [[(250000, None)], None, [(250000, None)], None]
    Fred: [None, 'No bank associated with name Fred']

i gotta say, when this worked, i got really excited :)


## monadic function doesn't care which monad it's using

lets define `m_map` which knows about the monadic plumbing:

    def m_map(mf, mvals):
        return map(lambda x: bind(x, mf), mvals)

    names = ["Irek", "John", "Alex", "Nick", "Fake"]
    m_names = map(unit, names)
    m_decisions = m_map(name_qualifies_for_loan, m_names)

    for name, m_decision in zip(names, m_decisions):
        print "%s: %s" % (name, m_decision)

`mmap` knows about bind, but it doesn't care which `bind` we're using. This version of `mmap` will work with ANY monadic function, whether we're using the maybe-monad, the error-monad, or any other monad. The specific behavior of the monad is abstracted behind the `bind` and `unit` functions. If we write the code such that a different `bind` is in scope, `mmap` still works.


## monads can be combined

- change error monad to not short circuit (maybe monad does that), and then compose the monads to combine them
- need python way to abstract away which monad we're "inside" of

## cont-m

- make the bank API asynchronous, but keep composability

## TODO

I'm not sure where to go after that, and how to integrate this story with monad transformers. is there a use case for cont-m here? what benefit might we get from that, parallelism? laziness? falling back when a low-level error (like connection interrupted) occurs? state-monad for when the bank wants to know if someone is asking other banks for a loan? i'll want to compare the monadic code to traditional versions with only first-order functions, to demonstrate (or not!) that the monadic code leads to writing better code faster with less bugs.



## references


- http://www.intensivesystems.net/tutorials/monads_101.html
- http://khinsen.wordpress.com/2009/04/22/monad-tutorial-for-clojure-programmers/
- http://stackoverflow.com/questions/9881471/map-and-reduce-monad-for-clojure-what-about-a-juxt-monad
- http://onclojure.com/2009/03/05/a-monad-tutorial-for-clojure-programmers-part-1/
- http://stackoverflow.com/questions/3870088/a-monad-is-just-a-monoid-in-the-category-of-endofunctors-whats-the-problem
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
-
- http://programmers.stackexchange.com/questions/72557/how-do-you-design-programs-in-haskell-or-other-functional-programming-languages/72562#72562

monad site:reddit.com
http://blog.sigfpe.com/2006/08/you-could-have-invented-monads-and.html
