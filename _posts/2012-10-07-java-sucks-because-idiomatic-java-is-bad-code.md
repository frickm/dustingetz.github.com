---
layout: post
title: Java sucks because idiomatic java is bad code
---

# {{ page.title }}


I'm sure you've seen the [AbstractSingletonProxyFactoryBean](http://static.springsource.org/spring/docs/2.5.x/api/org/springframework/aop/framework/AbstractSingletonProxyFactoryBean.html). This hit HN the other day, titled *Everything that's wrong with Java in a single class*.

This triggered a bunch of comments, saying things like

> it's not java's fault, it's the architecture astronauts who think this is good abstraction

> The problem with Java is the sheer amount of 'bad' code written in it

**This is wrong. It's definitely java's fault.**

## Languages have opinions

Java is designed around the principle of "everything should be a class" which makes everyone's first instinct to have lots of mutable instances with lots of mutable member variables. All the language-provided data structures are mutable, and it's difficult to write expression-oriented code, which means the order in which statements execute will always be important, and always be a source of bugs. Turns out that this gravity towards mutable state leads to some fragile code that everyone is afraid to touch in the large.

Compare to Clojure, which supports mostly all the same features as Java, but emphasizes them differently. You can use mutable data, but by default everything is immutable. You can have methods and instances, but its easier to just use a function, and provides the tools to scope the state as tightly as possible: to the single expression where the state is meant to be used. Java can do higher order functions and closures, but they're so verbose that its better to choose imperative for/while loops over map/reduce/filter, which don't even come in the standard library. Clojure is designed to make higher order functions and expression-oriented thinking idiomatic, java makes this so difficult as to be not worth pursuing even with a team who already understands how to think this way.

Java's design leads people towards code that you're afraid to touch for fear of disturbing state somewhere else which causes failures unrelated to your change. That's why when you need to fix a bug, in Java, people will expose a hook here and there to make a minimum change that they are sure doesn't break code somewhere else. A decade of changes designed to expose little hooks to not break existing code, you end up with a AbstractSingletonProxyFactoryBean with intercepters and proxies and loaders. Nobody is claiming that AbstractSingletonProxyFactoryBean is an abstraction. It came into existence over time because it was the easiest way to fix bugs and adapt to changing requirements while not breaking existing code.

## But can't an expert team overcome Java's negative gravity?

Someone responded to these comments:

> You can do all the functional stuff in Java, Clojure is doing exactly that. It will be more verbose, but once you learn the ropes it just looks different, it is not different.

This is a good point; Java has gravity towards bad code, can an informed team overcome the gravity towards implementation inheritance
and mutable state?

Here's a paper that describes a team who migrated a bunch of excel macros to Java, found that they had built an unmaintainable mess, then took the same team and built it in OCaml, and have been using OCaml ever since. [OCaml for the masses, Yaron Minsky, Jane Street](http://queue.acm.org/detail.cfm?id=2038036)

> But somehow when coding in Java we built up a nest of classes that left people scratching their heads when they wanted to understand just what piece of code was actually being invoked when a given method was called.

The same team moved to a functional language to great success.

## It's the language desginer's job to establish idioms, ours is to ship business code

Our job is to ship code that works on schedule. All the younger Java hires are going to follow the idioms of the language, because they are still learning. Teams that start with a few college grads using Java, product is successful, profit is generated, new hires are made but always less senior than those whos tarted ("A begets A and B begets C"), and the natural equilibrium is what you read about on thedailywtf.com.

here is some code i wrote last week.

{% highlight java %}
public static Map<String, Collection<InboxEntry>> partitionByTaskName(Collection<InboxEntry> delegated)
    // unfortunately generalizing this requires a higher order function and java makes me not really want to bother
    // implementing default-dict correctly also needs a higher order function (default value constructor)
{
    Map<String, Collection<InboxEntry>> taskMap = new HashMap<String, Collection<InboxEntry>>(); // this is a default-dict
    for (InboxEntry inboxEntry : delegated)
    {
        // the partition key
        String task_name = inboxEntry.getQueueItemAttributes().fetchSimpleValue("task_name");

        Collection<InboxEntry> inboxEntries = taskMap.get(task_name);
        if (null == inboxEntries)
        {
            inboxEntries = new ArrayList<InboxEntry>();
            taskMap.put(task_name, inboxEntries);
        }

        inboxEntries.add(inboxEntry);
    }
    return taskMap;
}
{% endhighlight %}

This is a method i extracted from some old code somewhere else while i fixed a bug in it.

The functional programmer in me sees opportunities to generalize (and prevent future bugs here), so i try to refactor it into a defaultdict and a higher order function, using guava. 20 minutes later I still don't have working code, i revert, curse the gods because this is a 30 second refactor even in python let alone clojure, and move on, i don't have time for this shit.

Its not about it looking different. it doesn't matter that closures and objects are turing equivalent. its a syntax problem.writing functional java code has too much friction and is not worth it.

This is not even getting into how any hardcore functional code needs persistent data structures to be fast, which means
* you need to write your code against a 3rd party collections library instead of JDK provided implementations,
* any interop with 3rd party code may need to be marshalled into a mutable interface at the boundary which implies a copy,
* in the absense of built-in laziness or macros, pure-functional code which is a series of data transformations - think like a unix pipe - will expand all datastructures in memory along the way, it is prohibitively difficult to make the pipeline logic lazy (stream-oriented).

## Can you write the proper factoring of the above?

For those interested, HN commenter [anakanemison](http://news.ycombinator.com/user?id=anakanemison) did supply a proper Java refactoring. The functional programmers out there, would you have arrived at this solution in Java? How long would it take, would you have bothered in practice? ... what about in your favorite language?

{% highlight java %}
public static <T, K> Multimap<K, T> partitionBy(
    final Iterable<? extends T> inputList,
    final Function<? super T, ? extends K> partitionFunc)
{
    final Multimap<K, T> multimap = HashMultimap.create();
    for (T t : inputList) {
        multimap.put(partitionFunc.apply(t), t);
    }
    return multimap;
}
{% endhighlight %}
