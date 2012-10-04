---
layout: post
title: Writer, Reader, and State monads in Python
---

# {{ page.title }}

Assumed prior knowledge: understand error-m, maybe-m, and identity-m.

State-m is kind of confusing in haskell for beginners due to that icky `runST` stuff. Here it is in python, if it helps. It helped me a lot.

think of state-m = reader-m + writer-m.

let's start with writer-m, it's easiest. Writer monad lets you compose functions, while accumulating information out of band (through the return values), like log messages.

    def unit(v): return (v, [])                      # monadic value is a pair: a value, and the accumulated log messages

    def bind(mv, mf):
        val, out = mv[0], mv[1]                      # unwrap the value from the log
        r_mv = mf(val)                               # invoke the function with the unwrapped value
        r_out = r_mv[1]                              # return value is a new value wrapped up with some log messages
        final_out = out + r_out if r_out else out    # accumulate the new logs with the logs we already had
        return (r_mv[0], final_out)                  # return the new value wrapped up with the accumulated logs

    def addOne(x):
        val = x+1                        # build up some result value
        logmsg = "x+1==%s" % val         # build up some log messages
        return (val, [logmsg])           # return the result, wrapped up with the logs

    >>> addThreeLogged = m_chain(addOne, addOne, addOne)   # def addThreeLogged(x): addOne(addOne(addOne(x)))
                                                           # except with bind plumbing to make the types line up
    >>> map(addThreeLogged, [10,20,30])
    [(13, ['x+1==11', 'x+1==12', 'x+1==13']),
     (23, ['x+1==21', 'x+1==22', 'x+1==23']),
     (33, ['x+1==31', 'x+1==32', 'x+1==33'])]

reader lets you define a computation which will get some read-only state, like an environment.

    def unit(v): return lambda env: v    # monadic value is a function of environment

    def bind(mv, mf):
        def _(env):                      # return a function of environment
            val = mv(env)                # which, when invoked with an environment, returns a value
            return mf(val)(env)          # apply the next function with the new value, and the same environment
        return _

    def read(key):                       # a computation in an environment, that reads a value from the environment
        def _(env):                      # return a function of environment
            return env[key]              # which, when invoked with an environment, will look up the key in the env
        return _

    >>> mv = read('a')
    <a function of env>
    >>> mv({'a': 7})
    7
    >>> mv({'a': 10, 'b': 3})
    10

    >>> computation = bind( read('a'),  lambda a:
                      bind( read('b'),  lambda b:
                            unit( a+b ))
    <a function of env>
    >>> computation({'a': 7, 'b': 3})
    10
    >>> computation({'a': 42, 'b': 1})
    43

finally, ST (aka environment) is the combination of reader and writer. computations are within an environment, but they are allowed to return a new environment for the next computation to run in.

    def unit(v):                  # monadic value is a fn of environment, which
        return lambda env: (v, env)     # returns a value wrapped up with a new environment

    def bind(mv, mf):
        def _env_mv(env):               # good luck
            val, newenv = mv(env)
            return mf(val)(newenv)
        return _env_mv

    def write(key, val):
        def _(env):
            newenv = env.copy()          # because python maps aren't immutable :(
            newenv.update([(key, val)])  # newenv has the new key:val, old env unchanged
            return None, newenv          # return no value wrapped up with the newenv
                                         # (this function is invoked for it's simulated side effect)
        return _

    def read(sym):
        def _(env):
            return env[sym], env         # return the value of that symbol, wrapped up with an unchanged env
        return _

    >>> write('a', 2)
    <a function of env>
    >>> myenv = {'a': 999, 'b': 999}
    >>> write('a', 2)(myenv)
    (None, {'a': 2, 'b': 999})           # no return value, just a simulated side effect
    >>> read('a')(myenv)
    (999, {'a': 999, 'b': 999})          # a return value, env unchanged (no side effect)

    >>> computation = bind( read('a'),        lambda a:
                      bind( read('b'),        lambda b:
                      bind( write('c', a+b),  lambda _:
                            unit(c+1)         )))
    <a function of env>
    >>> computation({'a': 7, 'b': 3})   # compute an expression along with a side effect
    (11, {'a': 7, 'b': 3, 'c': 10})

the only way to learn this stuff is to play with it in the repl. this code runs.
