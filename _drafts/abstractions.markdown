##why are abstractions important
- hoare property, correct code, maintenance cycles from mythical man month. this will make software cheaper!
- missing level of abstraction in win32, joel cruft code
- leaky abstractions happen, fix the abstraction at the right layer
- many engineers never learn how to find the right abstraction, trained to patch things at the wrong layer
- abstractions weren't as important because applications weren't as complex - not as many layers of abstractions. you can afford to skip a layer, it wasn't so complex that you couldn't keep it all in your head, win32 programs could be successful
- javascript provides the tools to create good abstractions, but it requires a surgeon to build the foundations for the team to use. javascript itself does not have opinions about what is a good thing to do or what is a bad thing to do, it just provides a set of tools for the team to figure out what is good and bad
- once a product is shipped, it is difficult to go back and fix things due to business concerns. java, javascript can't be fixed in a timely manner, even if there are surgeons on the committee now. the poor abstractions get baked into the product, and we can't align short term business interests ($) with the need to fix the abstractions. the technical debt becomes not worth it to fix. 
- c++ uninitialized variables
- win32 locked file
- web display framework
- C quicksort vs haskell quicksort
- ext.ajax vs XHR

##what makes a language popular
- php, python, rails, javascript/jquery - easy!
- java - OOP is a good idea
- need a language which says "OOP is good at encapsulating mutable state, but mutable state as a default is a bad idea, use data structures and functions instead"

##baking technical debt is a really big deal
- how does agile relate to all of this?
- regulatory compliance in very large industries
- QA processes for validation
- red-tape exists to protect from large-scale, company-breaking events, like security disasters, lawsuits
- processes are un-agile, but they are important, and they aren't going to go away any time soon. we need to address technical debt before it gets baked in a shipped release.
- many startups don't have this problem, and a many of the best surgeons thus are attracted to startups, where they have product control, they can a) build a system without losing control of the technical debt before its baked, or b) not worry about technical debt because they're the one causing it, and make it someone elses problem. (this is the state of industry right now)

##what economic factors are keeping us at a local maxima
- javascript: you can ship a MVP with code that is aware of the DOM. it is easier this way. but as it gets more complex, we need to work with higher level metaphors, but the foundations are already baked. famous jquery tangle. "When working on a web application that involves a lot of JavaScript, one of the first things you learn is to stop tying your data to the DOM. It's all too easy to create JavaScript applications that end up as tangled piles of jQuery selectors and callbacks, all trying frantically to keep data in sync between the HTML UI, your JavaScript logic, and the database on your server. For rich client-side applications, a more structured approach is often helpful." http://documentcloud.github.com/backbone/
- look at all the one-off javascript libraries, all trying to fix the problems differently? need a surgeon to just choose for us. jquery, sencha/extjs, YUI, backbone.js, sugar.js, underscore.js, which one do I use?
- baking technical debt. complex branching required. cvs vs git.
- how do we fix it? we have to flip the equlibrium at the language level. we need a language, that can get adoped widespread into enterprise problems, which normal, human, college grad, enterprise workers can use, which has self-training aspects that encourage us to build abstractions from the lowest level. "learning lisp changes the way you think about code" this is the effect we want, just probably not in a lisp?

##how can we design a system such that the equlibrium state is a global maxima?
- generational problem - university, enterprise
- the people who are getting work done, their business economics are ship, not 
- how can we shift the equlibrium such that we are training engineers to build better abstractions? clojure, languages built by a surgeon, with strong opinions of what abstractions should look like. becomes a self-teaching circle.
- haskell - wrong level of abstraction. we don't care about types and purity, at least to the degree that haskell & ML forces us to think at. maybe the complexity of our problems isn't high enough yet, maybe someday, but right now, everyday business problems, don't need haskell & ML. some problems do - compilers, quant trading (jane street) - but not the majority of software businesses.
- ruby on rails - crud biz apps - they are at the right level of abstraction for the task at hand, until the task gets bigger - look at all the people complaining about rails 3.0. how do we line up the business incentives such that money is driving quality?
- clojure is a great language - interactive development inherent in lisps - but it probably isn't the answer either, the masses may not be ready to accept a lisp. also the tooling behind lisp may not be ready (?) for large codebases, which have the deepest abstration pain. i can navigate a large codebase in eclipse far better than i can navigate one in emacs. maybe better abstractions can prevent codebases from growing to this size, but the transitional period is relevant.
- coffeescript isn't helping. coffeescript made things linearly better, but does not provide any opinions to reinforce good abstractions and discourage bad ones. its too much like javascript - it lets you get away with too much. 

