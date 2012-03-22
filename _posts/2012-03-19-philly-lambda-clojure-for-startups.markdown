---
layout: default
title: Philly Lambda presents Paul deGrandis on Clojure Powered Startups
---

# {{ page.title }}

Philly Lambda notes - 12 march 2012 
sponsored by Comcast Interactive Media; cimlife.com

*ed: this is my personal take on Paul's lecture, the discussion has been organized, re-ordered, edited and clarified by me. My own opinions have inevitably leaked into this.*

speaker: Paul deGrandis
- VP Engineering at Tutorspree, worked briefly at CIM, lives in NYC now
- giving this talk for clojure/west on march 16

## a CTOs job is to manage risk
if clojure adoption and clojure success is on the rise, where are the success stories? whats the best way to apply clojure?

Neal Ford talk at Conj last autumn - [master plan for enterprise domination (slides)](http://nealford.com/downloads/conferences/Clojure_Masterplan(Neal_Ford).pdf) *ed: Neal's talk focuses on out-of-band factors - why do some technologies become popular? how can we leverage this to make clojure popular in enterprise?*

Neal Ford: CTOs are driven by fear, make decisions worried about making the wrong decisions. evaluate risk, unknowns, failure changes. true of large enterprises, true for startups. we make decisions for evaluation risk. in order to do this, need objective data, rapid adaptation - so i can instantly realize what is wrong, i need tools that allow me to see and switch real fast. spiral iteration. risk driven architectures. want to take the unknowns, how can we find more info about them, navigate the space as flexible as we possibly can. success is - winnning at the risks, beating the odds.

Paul works with startups - small teams ~5. As a startup, Paul has the freedom to choose his stack, we only need to convince ourself it's the right choice, not everyone else. 

If clojure actually does manage risk better than other choices, this is our tagline: "Ill show you four cases to use clojure to manage risk better than you are now " - this is the killer argument for clojure adoption. To back this up, we need a "success protocol". examine people using clojure, what techniques can we use to get success? mine the success stories, distill them into one toolbox.

Paul: "building a team using clojure was scariest thing ive ever done, borderline irrisponsile". only risk is getting tyhe team to use clojure.

## Clojure manages risk better than any other technologies
what are the reasons this was making him personally more productive?

kyle - community emphasizes core clojrue community valus - lead you to writing code that doesn't have to be debugged.

adapt quickly. Paul needs machine learning, he has a statistical problem. deliver as fast as i can to look at data. to iterate on hard problem - how to detect learning styles. (*ed: but what about the people solving problems of complexity, not as intuitively suited to the mathematical nature of functional programming?*)

the realization that "wow, the way im solving every problem is this way". the abstractions you arrive at other people independently arrived at.

culture surrounding clojure and that community - don't need a huge team, need a small team. a pro, and someone taking legos . but need a lego factory. taking about surgical team per fred brooks MMM - the "surgeon" does the critical, core infrastructure work - he acts as a multiplier for his team. community is infectious - winning product needs winning team - winning team needs winning culture. attract a great team, then product follows naturally. (*ed: clojure also tends to teach surgeons-in-training - the language has strong opinions which influence our code for the better. the opinions teach us.*)

note both Paul and Kyle (mobile banking) are using Rails as their web layer. reasons cited: maturity of platform. (*ed: this surprised me, I'd like to know more about this, as I think clojure would be ideal for the enterprise vertical app i work on, including middle tier. I think there is a mismatch in approaches - the enterprise frontend is heavily data oriented with ajax services side by side with business state routines, as opposed to Tutorspree's frontend which is mostly presenting content as markup? Kyle's web layer, is it a frontend to his mobile banking infrastructure? I wonder if his web layer is only relevant to the internet-facing website, thus there's no complex backend logic, which makes Rails a better fit? this is purely speculative. Tutorspree's frontend certainly has core value, Paul talks of monitoring user actions like click locations in order to figure out learning styles and match up with tutors*)

Q (Dustin): Clojure vs python. Why is clojure so much better?
A (Paul): clojure as a community, toolset, language - forces you to be bottom up. build on the smallest nugget, building blocks, that you can build with. a bucket of legos. *ed: Paul implies this is something that python is less good at - i agree - needs more thought.* 

clojure forces you to think problems in a bottom up way. build legos to solve his learning problem - in other langages, you have to actively try harder to do this. other startp collective intelligence problem. he did it in python, extremely funtional python - but he had to try real hard, python's inherent opinions and idioms don't naturally lead code towards a functional style.

python: write a reduce that grows a collection - due to native list datastructure being mutable, this implies a copy. *ed: also no official support for tail recursion - Guido van Rossom says ["I don't want TRE in the language... short answer, it's simply unpythonic."](http://neopythonic.blogspot.com/2009/04/tail-recursion-elimination.html)*

"python's datastructures just aren't designed to be functional from the bottom up." Paul(?) at one point was writing highly functional python for a machine learning business, and became frustrated by this - if you want to take a functional style all the way, you want language support. Rich Hickey quote: https://docs.google.com/document/pub?id=1Vgbw3hGCye2rtmnZ6iSGy0WfY9aElE3SX3WbXn2-bAA

*ed: here are [a few educational python implementations of reduce and map](https://github.com/dustingetz/sandbox/blob/master/etc/map.py) to see what he means - the naive recursive implementation ("the functionally pure way")  can overflow the stack, and using reduce to implement map implies a concatenation operator, which in python is a copy. we could implement our own persistent data structures that are immutable so they don't need this copy, and copy this to and from native lists, but when you start bending the language in a way that hte language doesn't like, the community hates it, teams will revolt, etc. We can implement our primitives (filter, map, reduce) imperatively, but now we're not functional all the way down. its unclear to me how much it matters for most problems - but functional all the way down does seem to vlend itself towards a deep understanding of fundamentals.*

Same with Java - you can write functional-style Java, but its not batteries included, need third party libraries like Google Guava which is full of hacks to short circuit reflection, allow you to do reduce, etc *ed: I don't have a source on this quote and I don't recall who said it - in my experience with functional java, I just implement the basic functional operations imperatively and ignore the cost of copies because we're a webapp and copies will never be a bottleneck. the boilerplate surrounding function objects in Java is the real barrier to adoption, makes things look much scarier than they really are to teammates such that they prefer to filter with a for loop. All this said, I still feel a functional style helps us write code faster with less bugs, its just much less awesome than with native language support.*

Ben: clojure is not the only conceivable language to do these things: first class functions,  sane mutability, etc. We talked a bit about haskell but I don't have great notes.

Factor is a language that isn't a lisp, not a paren language, has a functional style. smalltalk had adanced interactive development environments.

Clojure has interactive development which is really helpful for building legos. (*ed: here's a great example of interactive development - [video: Quick Intro to Live Programming with Overtone](http://vimeo.com/22798433)*). interactive development keeps you square in the "zone". clojure lets you hit the "zone" better than any other technology.
  
Clojure leverages 30 years of lispers doing interactive development. lego analogy. not going at the problem head on. lego building happens interactively.

Dan M: languages rooted in lambda calculus are better at decomposing problems. 

Dustin asked if haskell can be like legos also - instead of building interactively by evaluating code as we edit it in emacs, we can build interactively by typechecking as we edit. Got negative reactions from the audience (notably Kyle and Dan). (*ed (dustin) - i think there is something to think about here, some of my best java code i can refactor & add features without running the code at all - the type checker tells me its correct, and it works the first time. It just takes discipline to decompse your java code into legos, it doesn't happen naturally like in haskell and clojure.*)
 
Dan Mead: "i dont write haskell for performance" 
Nick C: trying to get a lot of hands on haskell can be a problem.
Dan Mead: haskell was very academicy things in our brain. SPJ (creator of haskell) says its supposed to have a trickle down effect to other languages. haskel - first language to study monads, they have trickled into clojure now. *ed: and scala, and perhaps even javascript: [jQuery is a Monad](http://importantshock.wordpress.com/2009/01/18/jquery-is-a-monad/)*

Hunter H - Clojure is on jvm, and CLR sooner or later, and javscript (clojurescript) which is maturing quickly

Trevor - jruby is one of the fastest implementations of ruby. if you have a problem with ruby performance, running on the JVM is the typical first step. Hunter H asks if this is the case for jython, we don't know.

why is clojure more popular than scala? big theme at clojure conj - "stop fighting scala". some people want to get as far away from java as they can, the toolkit and mindset isn't what they want, perhaps the community is harmful. Scala community seems to really dig into functional scala which makes it a good transitional language to convince team as well. It's almost elitest - the Scala community projects a tone of: entry level scala = "don't use the the oo features", next level "knows how to do a reduce"

clojure has nifty features to make things persistant
clojure wrapping lbiraries discussion. wrapping war

Seems a lot of smart people say "Clojure as a language matches my style of problem solving" (*ed: Many of the best programmers I've met are very careful to be humble - they speak of what works for them and their problem, and are careful not to generalize that to everyone, but of course these are the people we stand to learn the most from.*)


Trevor: clojure matches your thought process, but how does your thought process work?
feedback: transitioning from java - java devs afraid of functional.

Kyle didn't use clojure for his ("very thin") web layer. (*ed: im not sure how accurately my notes here capture what Kyle said.*) Kyle's web layer just sends stuff to RabbitMQ. Clojure's web layer libraries aren't mature, so uses rails for web layer. e.g. Ring - crappy error handling. this is a negative aspect of the community. Paul - 3 versons of tutorspree.com web layer, third is in clojure (not in production) - its an experiment, but he got it running in a week.

Q (Martin): why did it take so long for functional programming to start mainstreaming?

history of FP and lisps in general: lisp machines - first time raw performance died, because commodity computer   hardware won. (imperative von-neumann architecture which revolves aroundmanipulating state in registers). "inferior tech won". Common Lisp could have won, but it got so close to AI. (*ed: CL seems to have weaker opinions than clojure - it doesn't help people write stateless code - it lets the developer do whatever he wants.*)

OO - paradigms change with the level of complexity - procedural to non -o, non-p to OO. had a hard time using a different type of abstraction. now bigdata and concurrency. stream processing, sequences. GUIs. managing objects in an environment - managing state, the state is essential to the problem - OOP's nature of encapsulating state is perfectly suited to building GUIs. FP is about applying manipulations on data, a gui isn't. web request from response DOES map well to FP.


baked, clojurescript, importance of forms, designer lorem text on client with code to do transforms on the client


## how to evangelize
once we have convinced ourselves that Clojure helps us manage risk, we do these things to help spread awareness - evangelism:

1. prototype - get lots of eyes on clojure. increased awareness for everybody.
2. toolchain - lots of points of contacts with clojure. they don't have to understand, but we want them touching it
3. crux - solving the hardest part we can with clojure - lots of minds.

1. prototyping: maximize awareness, demonstrate value, allow others to "hold". two possible soltuons to this problem, heres the research, here's the impl. "you solved an unknown, w/ clojure, in one week"

2. toolchain - Pushbutton Labs built a compiler for this language to script his video game, and got it onto all the customers of the games.

after you stimulate team usage, team picks up, groks it. biggest risk - can i get enough talent, and can i teach my team how to program in clojure effectively? Case stides World Singles - Sean Corfield (a clojure contributor)  he used coldfusion before, switched to clojure. risk: we have to switch platforms, ColdFusion s dead, now what do we use?

so if we can sell them on "clojure makes things so easy", they're going to adopt it naturally. just need that first hump. get stuck in the feedback loop, the zone.

Paul. are we really winning? adoption, %growth of jobs is rocketing up. social proof that clojure manages risk really well. we questioned his graph of clojure % growth, the chart was misleading, but certainly clojure is experiencing sharp growth in terms of jobs, we just can't compare it to other technologies without better normalization.

## post talk demos
after talk, two Tutorspree demos using clojure.

how to build a static analyser for a function, dynamic language - which is hard for lisps because you can redefine the whole language. Clojure (as a lisp) can trivially process its own AST (abstract syntax tree), we can use logic programming to declare patterns that we think are code smells, and automatically flag them, or translate them into something better. Logic programming lets you solve problems like N-Queens by declaring constraints, and then search the solution space for things that meet the constraints.

"baking websites is fucking badass" - Paul demoed using clojure script to "bake" a website - a few of us hadn't seen this and had "holy shit" reactions. We get "lorem ipsum" mocked markup from a designer. Without modifying the mocked markup, we write functional Clojure transforms to substitute our content into the mocked markup. Then instead of rendering html or producing static html, we just send the client the mocked markup bundled with the transforms. Becomes trivial to interact with designers, they retain full control over their markup. (*ed: There was conversation about how this is only possible in Clojurescript (not javascript) due to the parenthetical nature of the language - we can encode transforms as forms (expressions) - in retrospect, this is not obvious to me.*)


Dustin Getz - 19 March 2012