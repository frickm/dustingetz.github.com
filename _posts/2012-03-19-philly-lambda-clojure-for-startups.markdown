layout: post
title: Philly Lambda presents: Paul deGrandis on Clojure Powered Startups
-

Philly Lambda notes - 12 march 2012 
sponsored by Comcast Interactive Media; cimlife.com
{{ page.title }}
========
[ed: this is my personal take on Paul's lecture, the discussion has been organized, re-ordered, edited and clarified by me. My own opinions have inevitably leaked into this. ]

speaker: Paul deGrandis
- VP Engineering at Tutorspree, worked briefly at CIM, lives in NYC now
- giving this talk for clojure/west on march 16

a CTOs job is to manage risk
--------------

if clojure adoption and clojure success is on the rise, where are the success stories? whats the best way to apply clojure?

Neal Ford talk at Conj last autumn - [master plan for enterprise domination (slides)](http://nealford.com/downloads/conferences/Clojure_Masterplan(Neal_Ford).pdf) [ed: Neal's talk focuses on out-of-band factors - why do some technologies become popular? how can we leverage this to make clojure popular in enterprise?] 

Neal Ford: CTOs are driven by fear, make decisions worried about making the wrong decisions. evaluate risk, unknowns, failure changes. true of large enterprises, true for startups. we make decisions for evaluation risk. in order to do this, need objective data, rapid adaptation - so i can instantly realize what is wrong, i need tools that allow me to see and switch real fast. spiral iteration. risk driven architectures. want to take the unknowns, how can we find more info about them, navigate the space as flexible as we possibly can. success is - winnning at the risks, beating the odds.

Paul works with startups - small teams ~5. As a startup, Paul has the freedom to choose his stack, we only need to convince ourself it's the right choice, not everyone else. 

If clojure actually does manage risk better than other choices, this is our tagline: "Ill show you four cases to use clojure to manage risk better than you are now " - this is the killer argument for clojure adoption. To back this up, we need a "success protocol". examine people using clojure, what techniques can we use to get success? mine the success stories, distill them into one toolbox.

"building a team using clojure was scariest thing ive ever done, borderline irrisponsile". only risk is getting tyhe team to use clojure.ma

Clojure manages risk better than any other technologies
-------------

what are the reasons this was making him personally more productive?
- web layer at tutorspree (php) - small, who cares
- reranking search order

adapt quickly. need machine learning. statistical problem. deliver as fast as i can to look at data. to iterate on hard problem - how to detect learning styles.

culture surrounding clojure and that community - don't need a huge team, need a small team. a pro, and someone taking legos . but need a lego factory. taking about surgical team per fred brooks MMM - 

community is infectious - winning product needs winning team - winning team needs winning culture (attract great team), then product comes for free. 

clojure forces you to think problems in a bottom up way. build legos to solve his learning problem - in other langages, you have to actively try harder to do this. other startp collective intelligence problem. he did it in python, extremely funtional python - but he had to try real hard, had 

note both Paul and Kyle (mobile banking) are using Rails as their web layer. reasons cited: maturity of platform. [ed: this surprised me, I'd like to know more about this, as I think clojure would be ideal for the enterprise vertical app i work on, including middle tier. I think there is a mismatch in approaches - the enterprise frontend is heavily data oriented with ajax services side by side with business state routines, as opposed to Tutorspree's frontend which is mostly presenting content as markup? Kyle's web layer, is it a frontend to his mobile banking infrastructure? I wonder if his web layer is only relevant to the internet-facing website, thus there's no complex backend logic, which makes Rails a better fit? this is purely speculative. Tutorspree's frontend certainly has core value, Paul talks of monitoring user actions like click locations in order to figure out learning styles and match up with tutors]

Q (Dustin): Clojure vs python. Why is clojure so much better?
A (Paul): clojure as a community, toolset, language - forces you to be bottom up. build on the smallest nugget, building blocks, that you can build with. a bucket of legos. [ed: Paul implies this is something that python is less good at - i agree - needs more thought]. 

python: write a reduce that grows a collection - due to native list datastructure being mutable, this implies a copy. [ed: also no official support for tail recursion - Guido van Rossom says ["I don't want TRE in the language... short answer, it's simply unpythonic."](http://neopythonic.blogspot.com/2009/04/tail-recursion-elimination.html)] 

"python's datastructures just aren't designed to be functional from the bottom up." Paul(?) at one point was writing highly functional python for a machine learning business, and became frustrated by this - if you want to take a functional style all the way, you want language support. Rich Hickey quote: https://docs.google.com/document/pub?id=1Vgbw3hGCye2rtmnZ6iSGy0WfY9aElE3SX3WbXn2-bAA

[ed: here are a few python implementations of reduce and map to see what he means - the naive recursive implementation ("the functionally pure way")  can overflow the stack, and using reduce to implement map implies a concatenation operator, which in python is a copy. we could implement our own persistent data structures that are immutable so they don't need this copy, and copy this to and from native lists, but when you start bending the language in a way that hte language doesn't like, the community hates it, teams will revolt, etc.]  https://github.com/dustingetz/sandbox/blob/master/etc/map.py

Same with Java - you can write functional-style Java, but its not batteries included, need third party libraries like Google Guava which is full of hacks to short circuit reflection, allow you to do reduce, etc [ed: I don't have a source on this quote and I don't recall who said it - in my experience with functional java, I just implement the basic functional operations imperatively and ignore the cost of copies because we're a webapp and copies will never be a bottleneck. the boilerplate surrounding function objects in Java is the real barrier to adoption, makes things look much scarier than they really are to teammates such that they prefer to filter with a for loop. All this said, I still feel a functional style helps us write code faster with less bugs, its just much less awesome than with native language support.] 


Ben: clojure is not the only conceivable language to do these things: first class functions,  sane mutability, etc. We talked a bit about haskell but I don't have great notes - 

Dan Mead: "i dont write haskell for performance" 
Nick C: trying to get a lot of hands on haskell can be a problem.
Dan Mead: haskell was very academicy things in our brain. SPJ (creator of haskell) says its supposed to have a trickle down effect to other languages. haskel - first language to study monads, they have trickled into clojure now. [ed: and scala, and perhaps even javascript: [jQuery is a Monad](http://importantshock.wordpress.com/2009/01/18/jquery-is-a-monad/)]

Hunter H - Clojure is on jvm, and CLR sooner or later, and javscript (clojurescript) which is maturing quickly

Trevor - jruby is one of the fastest impl of ruby. hh is it true of jython? dunno. 



why is clojure more popular than scala? big theme at clojure con - "stop fighting scala". 1. want to get as fasr away from java s i can, the toolkit and mindset isn't what i want - really dig into functional scala and convince team as well. "what level of scala programmer are you" "noob level - you don't use oo features. 2. you know how to do reduce and foldl" haha. 

clojure has nifty features to make things persistant
clojure wrapping lbiraries discussion. wrapping war

"language matches my style of problem solving"
- 

hh: interactive development.

factor - a language that isn't a lisp, not a paren language, smalltalk - adanced interactive development environments. super interactive languages - smalltalk. leverages 30 years of lispers doing interactive development. lego analogy. not going at the problem head on. lego building happens interactively.

DM: languages rooted in lambda calculus are better at decomposing problems. dustin said something stupid about haskell and got headshakes from kyle. about using type system to build legos interactively instead of using the interactive emacs.

thems:clojures opinions are what i find attractive.
problem domain and language domain. interactive development.

trev: clojure matches your thought process, but how does your thought process work?
feedback: transitioning from java - java devs afraid of functional.

martin: didn't use clojure for his web layer - very thin - firs stuff to rabit mq - easiest way to manage risk is to hire cheap developers. kyle  -the libraries for clojure are not mature, so uses rails for web layer. ring - crappy error handling. this is a negative aspect of the community. kyle - community emphasizes core clojrue community valus - lead you to writing cde that doesn't have to be debugged. paul - 3 versons of tutorspree, third is in clojure in web layer.


matin: why did it take so long for this to happeb? history of FP and lisps in general:
lisp machines - first time raw performance died, because commodity computer won. "inferior tech won". cl could have won, but it got so close to AI.;

OO - paradigms change with the level of complexity - procedural to non -o, non-p to OO. had a hard time using a different type of abstraction. now bigdata and concurrency. stream processing, sequence. Guis. managing objects in an environment - this is OO. sort of difficult to do functional. i"nonsense". fp is about applying manipulations on data, a gui doesn't . web request from response DOES map well.

collective intelligence problem in a web framewrok - super functional python. wow, the way im solving every problem is this way. the abstractions you arrive at other people independently arrived at.

nck c- surgical team metaphor

baked, clojurescript, importance of forms, designer lorem text on client with code to do transforms on the client


how to evangelize
-------------
once we have convinced ourselves that Clojure helps us manage risk, we do these things to help spread awareness - evangelism:

1. prototype - get lots of eyes on clojure. increased awareness for everybody.
2. toolchain - lots of points of contacts with clojure. they don't have to understand, but we want them touching it
3. crux - solving the hardest part we can with clojure - lots of minds.

1. prototyping: maximize awareness, demonstrate value, allow others to "hold". two possible soltuons to this problem, heres the research, here's the impl. "you solved an unknown, w/ clojure, in one week"

2. toolchain - Pushbutton Labs built a compiler for this language to script his video game, and got it onto all the customers of the games.


Paul seems to be speaking from the perspective of maximizing clojure adoption, not why clojure.
//
after you stimulate team usage, team picks up, groks it. biggest risk - can i get enough talent, and can i teach my team how to program in clojure effectively? Case stides World Singles - Sean COrfield (a clojure contributor)  he used coldfusion before, switched to clojure. risk: we have to switch platforms, CF s dead, how towe do it?

so if we can sell them on so easy, they're going to adopt it naturally. just need that first hump. get stuck in the feedback hump on Crux slide which is great - start kicking ass - "kick ass circle" haha. (ed zone baby) clojure lets you hit thiz zone better than any other technology.
//



Paul. are we really winning? adoption, %growth of jobs is rocketing up. manages risk really well.

we questioned his graph of clojure % growth.


after talk, two projects using clojure - 
- how to build a static analyser for a functiona, dynamic language - which is hard for lisps because you can redefine the whole lang
- doing for clojure w core.logic to reason aobut programs. logic programming (like prolog) - paul is new to this. ex problem: "how do i optimize calendars?" show a practical use for logic programming

- "baking websites is fucking badass" - "bake" ?? completely because its secure
- serve static page but make remote calls etc, demo of this? i missed what he said here

Dustin Getz - 19 March 2012