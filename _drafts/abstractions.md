

## designing a language, and why abstractions are important

first we had hex. wow, we can fix hardware bugs in software holy shit look at all the money we save by dodging ridgid hardware engineering processes! hardware bugs are expensive! years -> months, dodge all that process/red tape (its there for a reason but if can avoid the reason...)

then we had assembly. wow, we don't have to write code in hex anymore! now its possible to write a device driver for a monitor in <1yr!

c was designed, to abstract away the hardware. made unix possible - can run on all different machines.

C++ was designed, because lots of good C code represented their state in structs, and there were lots of functions operating on specific structs to mutate their state. OOP is a good idea, helps organize the data and the code operating on that data.

- c++ uninitialized variables
- win32 locked file
- web display framework
- C quicksort vs haskell quicksort
- ext.ajax vs XHR

java was designed. GUIs are getting popular, and memory complexity isn't essential anymore. deployment on different OS possible due to JVM. don't have to suffer with complexities of platform and memory management. now we can have lots and lots of classes, and build more complex than ever before because we don't spend days diagnosing uninitialized variables and counting references.

## pain of 2012 is complexity of our own making

pain points are debugging ever more complex half-assed metaphors. why is java code so shitty at large scale? at trivial scales (college projects). at what point do these pain points become relevant? complex business logic in enterprise. (note that this complex business logic isn't possible in the lower level languages than java, hell it might not even be possible in python, that's interesting too! elaborate, also how to say this without flamebait)

- complexities of web development (look at all the layers in play in web, are they any more or worse than like writing a game, or a windows app?
- cite article form ryan dhal about shitty state of development, vs article about dizzying complexity. our abstractions are doing just fine, they only break down at all at the very tippy top, the bleeding edge of complexity, modelling soft processes on top of 100 layers of abstractions. only the top few layers leak.

i wonder what an enterprise bizapp complexity looks like compared to a car, or a missle, or a fighter plane. plane we have well defined mechanical systems so the software is naturally well defined also. requirements are obvious.

what would a language look like, to address the pain of accidental complexity when modelling soft processes? why is naming abstractions (designing metaphors really) so hard? ideas, not code. we are now at the point where we have this complex set of requirements. so high level things, requirements are sooo meta. now our pain points are extracting the nuances of requirements (cite "how to read math/code"), and these nuances are getting lost in the noise, not caught till they're already baked into releases, etc. why are nuances lost? they're lost in complexities of half-assed abstractions, because we're so high level that we're making up our own metaphors. our pain points are our inability to abstract.

why aren't python etc good enough? is C# good enough? it  actually might be, i wish i had experience in large C# codebases.

## what are the good ideas on how to address complexity of our own making?

something about functional programming here, but without using the word functional programming. cue nostrademons article.

"OOP is good at encapsulating mutable state, but mutable state as a default is a bad idea because it causes bugs at really high levels and interacting complexity, use data structures and functions instead"

clojure? scala? not haskell. what is it about them that addresses this pain? why aren't other languages doing a good job?

scala is nice because its a fairly gentle transition, you can integrate with your OO team but it self-teaches towards other, better, ways to abstract. but the scalaz (fake haskell) stuff is a bit over the top, people get bitten and regret scalaz a lot, see hackernews.

## at current abstraction level, why are abstractions important
(this whole section is irrelevant or should be combined above, maybe this can be about)

- hoare property, correct code, maintenance cycles from mythical man month. this will make software cheaper!
- missing level of abstraction in win32, joel cruft code
- leaky abstractions happen, fix the abstraction at the right layer
- many engineers never learn how to find the right abstraction, trained to patch things at the wrong layer
- abstractions weren't as important because applications weren't as complex - not as many layers of abstractions. you can afford to skip a layer, it wasn't so complex that you couldn't keep it all in your head, win32 programs could be successful
- javascript provides the tools to create good abstractions, but it requires a surgeon to build the foundations for the team to use. javascript itself does not have opinions about what is a good thing to do or what is a bad thing to do, it just provides a set of tools for the team to figure out what is good and bad
- once a product is shipped, it is difficult to go back and fix things due to business concerns. java, javascript can't be fixed in a timely manner, even if there are surgeons on the committee now. the poor abstractions get baked into the product, and we can't align short term business interests ($) with the need to fix the abstractions. the technical debt becomes not worth it to fix.

## baking technical debt is a really big deal

- how does agile relate to all of this?
- regulatory compliance in very large industries
- QA processes for validation
- red-tape exists to protect from large-scale, company-breaking events, like security disasters, lawsuits
- processes are un-agile, but they are important, and they aren't going to go away any time soon. we need to address technical debt before it gets baked in a shipped release.
- many startups don't have this problem, and a many of the best software "surgeons" (MMM) thus are attracted to startups, where they have product control, they can a) build a system without losing control of the technical debt before its baked, or b) not worry about technical debt because they're the one causing it, and make it someone elses problem. (this is the state of industry right now)

baking technical debt. complex branching required due to baking debt. cvs vs git to manage this debt - past clients can benefit downstream because managing changes across branches doesn't need nearly as much human intervention. not sure if this is relevant.

## what economic factors are keeping us at a local maxima
cite graham hill buddy in VC industry: how do maturing startups handle their technical debt from demo-driven success criteria? they import older veterans to fix their mess. can you do this without a rewrite in a technology stack targetted at higher complexity?

targetting levels - you don't need `class MyApp{ public static void main(){}}` for a 100 line algorithm, or a blog. but this pays off for harder problems, clearly, hocky stick adoption. what about even harder than that, the next level?

javascript w/ jquery: you can ship a MVP with code that is aware of the DOM. it is easier this way. but as it gets more complex, we need to work with higher level metaphors, but the foundations are already baked. famous jquery tangle. "When working on a web application that involves a lot of JavaScript, one of the first things you learn is to stop tying your data to the DOM. It's all too easy to create JavaScript applications that end up as tangled piles of jQuery selectors and callbacks, all trying frantically to keep data in sync between the HTML UI, your JavaScript logic, and the database on your server. For rich client-side applications, a more structured approach is often helpful." http://documentcloud.github.com/backbone/

look at all the one-off javascript libraries, all trying to fix the problems differently? need a surgeon to just choose for us. jquery, sencha/extjs, YUI, backbone.js, sugar.js, underscore.js, which one do I use?

how do we fix it? we have to flip the equlibrium at the language level. we need a language, that can get adoped widespread into enterprise problems, which normal, human, college grad, enterprise workers can use, which has self-training aspects that encourage us to build abstractions from the lowest level.

## barriers to overcome

- generational problem - university, enterprise - training aligns with business incentives, follows the money because thats the fitness criteria. the fitness criteria MUST align with minimizing the complexity of our own making, the highest level stuff.
- the people who are getting work done, their business economics are ship, not
- how can we shift the equlibrium such that we are training engineers to build better abstractions? clojure, languages built by a surgeon, with strong opinions of what abstractions should look like. becomes a self-teaching circle.
- haskell - wrong level of abstraction. we don't care about types and purity, at least to the degree that haskell & ML forces us to think at. maybe the complexity of our problems isn't high enough yet, maybe someday, but right now, everyday business problems, don't need haskell & ML. some problems do - compilers, quant trading (jane street) - but not the majority of software businesses.
- ruby on rails - crud biz apps - they are at the right level of abstraction for the task at hand, until the task gets bigger - look at all the people complaining about rails 3.0. how do we line up the business incentives such that money is driving quality?
- coffeescript isn't helping. coffeescript made things linearly better, but does not provide any opinions to reinforce good abstractions and discourage bad ones. its too much like javascript - it lets you get away with too much. clojurescript is a better idea, has super strong opinions.

## what makes a language popular

and popular in their domains? hockystick adoption. (combine with intro section, popular means removes friction, removes pain)

enterprise languages: c, c++, java

fun languages: php, python, javascript, SO EASY to get started. funny how none of these languages were designed! python was guido's christmas project, javascript was hacked together for some reason or another cite, php was a kid project

perl, ruby? ruby is designed "to be beautiful" but how did he do? ruby is so dynamic that it can bite yourself hard, see rails 3.0, rails/ruby is great at complexity of a simple biz web site (removes alllll the pain, can built a blog in 5 min, CMS in a few weeks), but its pretty terrible at superhigh complexity, there's too much dynamic magic happening to reason about what is going on.

jquery - was certainly designed, to make dom manipulation easy. and its so easy, until you tie yourself into knots over your dom because its at the wrong level of abstraction as the app gets bigger.

## how can we design a system such that the equlibrium state is a global maxima?

(this is the meat of the post)

php sooo easy to get started but incremental change gradually becomes "oh god what have we done and its too late to change". alrady baked the debt, once you bake debt you've already lost. (is this even true? nice theorem not sure how i feel)

 is this even solvable with a language? can you make a language that is as easy to get into as it is to scale it to high complexity? (i think so)

 "learning lisp changes the way you think about code" this is the effect we want, just probably not in a lisp? or is the things "mere mortals" fear about lips irrational, and will be fixed with training (over generations) and time? interactive development is pretty nuts, and it seems to only happen in paren (expression-based/mathematical) languages. statements (and thus side effects aka void functions) make interactive development really hard and/or unfeasible.

clojure is a great language - interactive development inherent in lisps makes it extremely approachable for newbies - but it probably isn't the answer either, the masses may not be ready to accept a lisp. also the tooling behind lisp may not be ready (?) for large codebases, which have the deepest abstration pain. i can navigate a large codebase in eclipse far better than i can navigate one in emacs. maybe better abstractions can prevent codebases from growing to this size, but the transitional period is relevant.

do we even want novices working on high level complexity problems? no clojure shops hire newbies. you need to know your datastructures. clojure teaches you them (mine were weak, im getting decently good now). i actually think clojure can teach a novice how to think properly, and clojure can fix your broken-by-oop brain, if you have an open mind.

## what's the next level after this one?

jim G will love this. advanced research. strong ai (if its even metaphysically possible).
