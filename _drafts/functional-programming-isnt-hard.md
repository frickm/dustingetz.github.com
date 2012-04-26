
http://news.ycombinator.com/item?id=3723080
ook, Python's audience isn't Haskell's audience, and all these scary words like "curring", "incremental binding", "just like haskell", "left association" aren't things that most Python people care about. the article makes this stuff look like voodoo which is frustrating because functional programming doesn't have to be voodoo.
currying is really really easy and everyday python programmers probably do it on accident.
  def divisible(n, x): return n % x == 0

    def divisors(n):
          possible_divisors = range(2, int(sqrt(n))+1)
                return filter(lambda x: divisible(n, x), possible_divisors) # OMG curry !

                  assert divisors(24) == [2, 3, 4]
