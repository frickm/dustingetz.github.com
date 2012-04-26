its subjective, arguably a style using ternary operator is better because it is composable - it evaluates as an expression, as opposed to a series of statements. it's something the lisp and functional programming people like - no implied dependency on order in which statements execute.
i dunno if this is valid python but it demonstrates the idea
  doSomething(
      identify_vertebrate(animal) if animal.is_vertebrate() \
          else identify_invertebrate(animal))

            doSomething( if( is_vertebrate(animal)
                               identify_vertebrate(animal)
                                                  identify_invertebrate(animal)))

                                                    (doSomething (if (is_vertebrate animal)
                                                                       (identify_vertebrate animal)
                                                                                          (identify_invertebrate animal)))
                                                                                          this is part of "referential transparency", and once you get past "OMG parens" you realize its a Good Thing. parens are caused by a style preferring composition of expressions over executing statements, not lisp.
