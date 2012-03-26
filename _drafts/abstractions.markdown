- abstractions
- missing level of abstraction in win32, joel cruft code
- leaky abstractions happen, fix the abstraction at the right layer
- many engineers never learn how to find the right abstraction, trained to patch things at the wrong layer
- ext.ajax vs XHR
- how can we turn this around? language which encourages good abstractions - clojure
- abstractions weren't as important because applications weren't as complex - not as many layers of abstractions. you can afford to skip a layer, it wasn't so complex that you couldn't keep it all in your head, win32 programs could be successful
- javascript provides the tools to create good abstractions, but it requires a surgeon to build the foundations for the team to use. javascript itself does not have opinions about what is a good thing to do or what is a bad thing to do, it just provides a set of tools for the team to figure out what is good and bad
- once a product is shipped, it is difficult to go back and fix things due to business concerns. java, javascript can't be fixed in a timely manner, even if there are surgeons on the committee now. the poor abstractions get baked into the product, and we can't align short term business interests ($) with the need to fix the abstractions. the technical debt becomes not worth it to fix. 
- javascript: you can ship a MVP with code that is aware of the DOM. it is easier this way. but as it gets more complex, we need to work with higher level metaphors, but the foundations are already baked. famous jquery tangle. "When working on a web application that involves a lot of JavaScript, one of the first things you learn is to stop tying your data to the DOM. It's all too easy to create JavaScript applications that end up as tangled piles of jQuery selectors and callbacks, all trying frantically to keep data in sync between the HTML UI, your JavaScript logic, and the database on your server. For rich client-side applications, a more structured approach is often helpful." http://documentcloud.github.com/backbone/
- how can we shift the equlibrium such that we are training engineers to build better abstractions? clojure, languages built by a surgeon, with strong opinions of what abstractions should look like. becomes a self-teaching circle.
- haskell - wrong level of abstraction. we don't care about types and purity, at least to the degree that haskell & ML forces us to think at. maybe the complexity of our problems isn't high enough yet, maybe someday, but right now, everyday business problems, don't need haskell & ML. some problems do - compilers, quant trading (jane street) - but not the majority of software businesses.
- ruby on rails - crud biz apps - they are at the right level of abstraction for the task at hand, until the task gets bigger. how do we line up the business incentives such that money is driving quality?


- c++ uninitialized variables
- win32 locked file
- web display framework
- C quicksort vs haskell quicksort