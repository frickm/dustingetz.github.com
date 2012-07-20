---
layout: post
title: clojurescript internals: implementing `doc` and other dynamic functionality
---

# {{ page.title }}

It's open source week here at hackerschool, and five of us are contributing to ClojureScript, an implementation of Clojure hosted in a browser. ClojureScript doesn't have 'doc' (see issue: [CLJS-138: make docstrings accessible from the REPL](http://dev.clojure.org/jira/browse/CLJS-138)). We've implemented it, here's a deep-dive.

This is the product of a collaborative effort from six hackerschoolers:

* https://github.com/zachallaun
* https://github.com/julienfantin
* https://github.com/maryrosecook
* https://github.com/oskarth
* https://github.com/jamak
* https://github.com/dustingetz

## why doesn't ClojureScript have doc?

So what's `doc` anyway? Here it is in regular Clojure (henceforth, Clojure-JVM)

    ;; Clojure-JVM repl
    user> (defn get-answer
    "the answer to life, the universe, and everything"
    []
    42)
    #'user/get-answer

    user> (doc get-answer)
    -------------------------
    user/get-answer
    ([])
      the answer to life, the universe, and everything
    nil

Why doesn't ClojureScript have it? Well, that's complicated. Lets look at [`doc`'s implementation](https://github.com/clojure/clojure/blob/clojure-1.4.0/src/clj/clojure/repl.clj#L120), and dive into how ClojureScript works.

```
(defmacro doc
  "Prints documentation for a var or special form given its name"
  {:added "1.0"}
  [name]
  (if-let [special-name ('{& fn catch try finally try} name)]
    (#'print-doc (#'special-doc special-name))
    (cond
      (special-doc-map name) `(#'print-doc (#'special-doc '~name))
      (resolve name) `(#'print-doc (meta (var ~name)))                       ;; <-----
      (find-ns name) `(#'print-doc (namespace-doc (find-ns '~name))))))
```

the interesting line tells us that docstrings come from metadata associated with a `var`:

```
user> (defn foo "hello docstring" [] 42)
#'user/foo
user> (meta (var foo))
{:arglists ([]), :ns #<Namespace user>, :name foo, :doc "hello docstring", :line 1, :file "NO_SOURCE_FILE"}
user> (:doc (meta (var foo)))
"hello docstring"
```

how do `meta` and `var` work in Clojure-JVM? [impl of meta](https://github.com/clojure/clojure/blob/clojure-1.4.0/src/clj/clojure/repl.clj#L120):

```
(def
 ^{:arglists '([obj])
   :doc "Returns the metadata of obj, returns nil if there is no metadata."
   :added "1.0"
   :static true}
 meta (fn ^:static meta [x]
        (if (instance? clojure.lang.IMeta x)    ;; <---- dependency on Java code
          (. ^clojure.lang.IMeta x (meta)))))
```

[IMeta](https://github.com/clojure/clojure/blob/clojure-1.4.0/src/jvm/clojure/lang/IMeta.java ) is a Java interface that describes how to get metadata from things that have it - functions, objects, references, maybe other stuff -

```
public interface IMeta {
    IPersistentMap meta();
}
```

`var` is a special form implemented in Java, and its implementation is more complicated, but basically it is the thing that `def` builds and returns. functions, values, objects are all vars, vars are invokable and bindable and have metadata. `(def x 1)` doesn't build a java.lang.Long; it builds a var which refers to a value which is a Long. This `var` abstraction is how Clojure implements repl functions like `doc`, `type`, `source`.

All these dependencies on Java code requires a JVM, and not compatible with ClojureScript, and have to be re-written or avoided. For performance reasons, ClojureScript tries to compile down to native JavaScript types. Clojure-JVM functions turn into JavaScript functions, and Clojure-JVM vars compile down to raw Javascript code: `(def x 1)` will (roughly) compile to `var x = 1`, with various contexts around it to ensure Clojure's scoping rules are respected, etc. A consequence of this optimization is that the whole concept of Clojure-JVM `var`s does not exist in ClojureScript. The metadata associated with Clojure-JVM vars is ommitted from the javascript output, including docstrings. Bad ClojureScript.

We built a ClojureScript core API which allows ClojureScript code to query the compiler environment, allowing us to implement things like `doc` and `macroexpand`.

## lets do it

lets open a compiler repl and find the doc metadata.

here is a sample cljs file to play with:

```
(ns example.hello
  (:require
    [example.crossover.shared :as shared]))

(defn ^:export say-hello []
  (js/alert (shared/make-example-text)))

(defn add-some-numbers [& numbers]
  (apply + numbers))
```

we modified `analyze-file` to support loading from the filesystem instead of just the classpath. lets anayze a file.

```
(require '[cljs.analyzer :as a]) ;=> nil
nil
(require '[cljs.compiler :as c]) ;=> nil
nil
user> (def f "file:///Users/dustingetz/src/cljs-bootstrap/src-cljs/example/hello.cljs")
#'user/f
user> (a/analyze-file f)
nil
user> (pprint @a/namespaces)
{example.hello
 {:defs
  {add-some-numbers
   {:method-params ([numbers]),
    :max-fixed-arity 0,
    :variadic true,
    :protocol-inline nil,
    :protocol-impl nil,
    :fn-var true,
    :line 8,
    :file
    "/Users/dustingetz/src/cljs-bootstrap/src-cljs/example/hello.cljs",
    :name example.hello/add-some-numbers},
   say-hello
   {:method-params ([]),
    :max-fixed-arity 0,
    :variadic false,
    :protocol-inline nil,
    :protocol-impl nil,
    :fn-var true,
    :line 5,
    :file
    "/Users/dustingetz/src/cljs-bootstrap/src-cljs/example/hello.cljs",
    :name example.hello/say-hello}},
  :requires-macros {},
  :uses-macros nil,
  :requires {shared example.crossover.shared},
  :uses nil,
  :excludes #{},
  :name example.hello},
 cljs.core {:name cljs.core},
 cljs.user {:name cljs.user}}
nil
```

as the analyzer processes forms, it adds them to `a/namespaces`. each namespace maps its vars to various compiler information. at first glance this info looks like a superset of the info returned by (meta (var x)):

```
user> (defn ^{:a 1} foo "hello docstring" [] 42)
#'user/foo
user> (meta (var foo))
{:arglists ([]), :ns #<Namespace user>, :a 1, :name foo, :doc "hello docstring", :line 1, :file "NO_SOURCE_FILE"}
```

but note that the docstring is missing, as well as the arbitrary metadata we attached. We modified the analyzer to add this information to the analyzer output, which the CLJS team confirmed was a bug.

* expose this as a web service to query from an arbitrary javascript environment -- rhino, node, etc



ClojureScript has a few parts:

* compiler: emits javascript files
* analyzer: intermediate step during compilation to transform Clojure forms into a sort of AST
* re-implementation of certain parts of clojure.core which have JVM dependencies (for example, Clojure's persistent data structures are built on top of Java's data structures)
* a repl to evaluate ClojureScript in a javascript environment (e.g. a browser, or Rhino)

The repl is where we are focusing.





The ClojureScript repl needs some implementation of 'eval' which can compile a user-entered ClojureScript form into javascript so the browser can understand it. The ClojureScript compiler is not self-hosting: it written in Clojure-JVM (as opposed to ClojureScript), uses pieces of Clojure-JVM which aren't implemented in ClojureScript, and has many JVM dependencies. At this stage of the project it is not a priority to remove these dependencies, thus it is presently impossible to implement 'eval' in pure ClojureScript and not possible to have a pure ClojureScript browser repl.

Instead, the ClojureScript repl is hosted on a Clojure-JVM backend, which pushes compiled javascript to a browser environment over a HTTP connection. This lets us type in ClojureScript forms (or evaluate specific forms from within an IDE, e.g. emacs), which are compiled to javascript and sent to the browser. In this way we can evaluate forms which manipulate the DOM, display message boxes, log to the browser developer console, etc.


So lets take a look at how Vars work

show how doc and meta work with vars

how to fix it

changes needed
