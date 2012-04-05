#Dustin's awesome monad tutorial for humans, in Python

> A monad is a monoid in the category of endofunctors, what's the problem?
-- [James Iry](http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html)

Right. This tutorial exists because Monads are easy, they help us write cleaner code, and you don't need to understand what a endofunctor is to see how monads can help us all write good code faster with less bugs.

##Error handling is hard

consider the following API:

    def userid_from_name(person_name):
        if person_name == "Irek": return 1
        elif person_name == "John": return 2
        elif person_name == "Alex": return 3
        elif person_name == "Nick": return 1
        else: return None

    def balance_from_userid(userid):
        if userid == 1: return 1000000
        elif userid == 2: return 75000
        else: return None


    def balance_qualifies_for_loan(balance):
        if balance > 200000: return balance
        else: return None

this is how we might use this API:

    def name_qualifies_for_loan(person_name):
        mUserid = userid_from_name(person_name)
        if not mUserid:
            return None
        mBalance = balance_from_userid(mUserid)
        if not mUserid:
            return None
        mLoan = balance_qualifies_for_loan(mBalance)
        if not mUserid:
            return None
        return mLoan

    for person_name in ["Irek", "John", "Alex", "Nick", "Fake"]:
        qualified = name_qualifies_for_loan(person_name)
        print "%s: %s" % (person_name, qualified)

output:

    Irek: 1000000
    John: None
    Alex: None
    Nick: 1000000
    Fake: None


I see this pattern all the time in business applications. The traditional way is to use exceptions, which have various debates about whether this is a good pattern or not. Maybe we will forget to catch an exception which can then crash our system on accident. More or less, enterprise has accepted them, because error handling like this is so herror prone. Joel wrote about this in his infamous article. Then consider, say, win32 code, where the win32 API does not use exceptions, so we end up wrapping the darn thing in our own exceptions. (maybe bad example since win32 uses GetLastError antipattern)

##factor out the plumbing

Lets take a good hard look at this. How can we factor out the error checking boilerplate in our code that uses the API? Lets factor out a function `bind`:

    def bind(mv, mf):
        "call a function, but only if its input is valid"
        if (mv):
            return mf(mv)
        else:
            return None

    def name_qualifies_for_loan(person_name):
        mUserid =  bind(person_name, userid_from_name)
        mBalance = bind(mUserid, balance_from_userid)
        mLoan =    bind(mBalance, balance_qualifies_for_loan)
        return mLoan

output:

    Irek: 1000000
    John: None
    Alex: None
    Nick: 1000000
    Fake: None

That wasn't very hard, was it? This is our first monad, it's called the maybe-monad.

##add plumbing to handle error codes

Lets change our API to return error codes alongside the result.

    def userid_from_name(person_name):
        if person_name == "Irek": return 1, None
        elif person_name == "John": return 2, None
        elif person_name == "Alex": return 3, None
        elif person_name == "Nick": return 1, None
        else: return None, "No account associated with name '%s'" % person_name

    def balance_from_userid(userid):
        if userid == 1: return 1000000, None
        elif userid == 2: return 75000, None
        else: return None, "No balance associated with account #%s" % userid

    def balance_qualifies_for_loan(balance):
        if balance > 200000: return balance, None
        else: return None, "Insufficient funds for loan, current balance is %s" % balance

now we have to refactor `bind` to separate the value from the error.

    def bind(mv, mf):
        value = mv[0]
        error = mv[1]
        if not error:
            return mf(value)
        else:
            return mv

the very first value passed to `bind` needs to be in `(val, error)` format now too:

    def unit(value):
        return value, None

    def name_qualifies_for_loan(person_name):
        mName =    unit(person_name)
        mUserid =  bind(mName, userid_from_name)
        mBalance = bind(mUserid, balance_from_userid)
        mLoan =    bind(mBalance, balance_qualifies_for_loan)
        return mLoan

output:

    Irek: (1000000, None)
    John: (None, 'Insufficient funds for loan, current balance is 75000')
    Alex: (None, 'No balance associated with account #3')
    Nick: (1000000, None)
    Fake: (None, "No account associated with name 'Fake'")

This is called the error-monad. Are you starting to see the pattern?

Monads are a design pattern for composing functions who need plumbing to fit together. Monads are defined as a pair of functions, `bind` and `unit`, which work together to wrap and unwrap values. A "monadic value" is the wrapped value (the error tuple, here) and a "monadic function" is a function which returns a monadic value (returns an error tuple). Note that we didn't specifiy `unit` for the maybe-monad - this is because python objects are already nullable, so the values don't need to be wrapped. Formally we could say `def unit(x): return x`, otherwise known as the identity function.

##monadic function doesn't care which monad it's using

lets define `mmap` which knows about the monadic plumbing:

    def mmap(mf, mvs):
        return map(lambda x: bind(x, mf), mvs)

    names = ["Irek", "John", "Alex", "Nick", "Fake"]
    mNames = map(unit, names)
    mDecisions = mmap(name_qualifies_for_loan, mNames)

    for name, mDecision in zip(names, mDecisions):
        print "%s: %s" % (name, mDecision)

`mmap` knows about bind, but it doesn't care which `bind` we're using. This version of `mmap` will work with ANY monadic function, whether we're using the maybe-monad, the error-monad, or any other monad. The specific behavior of the monad is abstracted behind the `bind` and `unit` functions. If we write the code such that a different `bind` is in scope, `mmap` still works. Monads let us think about functions that require plumbing as if they didn't have any plumbing.

Its interesting that almost all of the code complexity in an enterprise business application, is plumbing. hmmmm.


##monads can be combined

- change error monad to not short circuit (maybe monad does that), and then compose the monads to combine them
- need python way to abstract away which monad we're "inside" of


###references


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
