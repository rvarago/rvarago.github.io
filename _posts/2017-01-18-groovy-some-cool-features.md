---
layout: "post"
title:  "Groovy - Some Cool Features"
tags:   groovy oop fp
---

> Groovy has plenty of cool features, let's look at some of them.

* * *

# Groovy

Groovy [1] is a dynamic language that runs inside the Java Virtual Machine
(JVM). Groovy is very similar to Java but has some nice additions which
give to the developer interesting tools to solve daily programming tasks elegantly.

# Environment

First of all, you need to install Groovy, which can be done by following the steps at [http://www.groovy-lang.org](http://www.groovy-lang.org).
For the examples, I used Groovy at version 2.4.5.

After installed and configured, open your terminal and type `groovyConsole`.

This will open the Groovy Console, where you can write commands and see the
results one after other. Now, you are ready to start exploring some featured provided by Groovy.

# Getting Started

This example declares a _List_ of _Strings_, iterates over it and prints a
greeting text in the console.

<script src="https://gist.github.com/rvarago/53469bcd154265e04587dddff39e7907.js"></script>

Firstly, Groovy is sort of dynamically typed language with extensions for static typing.

Secondly, Groovy has an embedded syntax to declare lists, `listOfNames` is a
list of strings.

Thirdly, Groovy introduces a simple way to print data: the `println` function, which is a
shortcut to print to standard output and append a new line at the end.

Fourthly, the text printed has a special meaning too, it uses a `GString`, which is similar to `String`,
with the additional capability to interpolate values into it.

# Ranges and Closures

This example computes the sum of numbers between 1 and 5, including the end points:

<script src="https://gist.github.com/rvarago/577182001147a59429782d7eb9c7f766.js"></script>

The above code introduces new structures of the Groovy programming language
which allows us to write concise code.

Firstly, the range notation where `(a..b)` returns a _List_ of numbers between
_a_ and _b_. We iterate over this `List` and for each element, we execute the code defined in a closure.

Roughly speaking, a closure is a block of code which has
access to the surrounding environment. Here, `each` is a method that accepts a
closure with one argument: loop variable obtained from the range, and when we don't declare its name,
Groovy injects it with the default name `it`.

We then use the closure to mutate the variable `sum` declared outside the `each` block.

# Reducing

In Functional Programming, there is this operation called fold (a.k.a reduce):

<script src="https://gist.github.com/rvarago/31a763346956295175c7caf6f8001ef0.js"></script>

Folding may be achieved by applying a function to each element of a collection, accumulating them somehow at each step, and then returning the accumulated value, e.g. the sum of the elements in the collection.

# Filtering

A filter is a function applied to collections, which filters out all entries in the collection that don't meet a predicate.

In Groovy, the filter operation is represented by the `findAll` method.

<script src="https://gist.github.com/rvarago/3fedbd5043ebab438b4b537e8e02c4e6.js"></script>

In the example, we filtered out all elements that are not even.

# Bonus

Groovy supports Object-Oriented Programming, and it provides classes as basic building blocks:

<script src="https://gist.github.com/rvarago/7d726777dd391780fd26904b91a1e280.js"></script>

It also offers named-parameters, which enables us to change the order in which we supply arguments to functions, passing the argument name alongside its value. We used it in constructor syntax, by indicating at the call site which attributes we're initializing.

As well as for lists, Groovy brings syntactic sugar to define associative containers:

```groovy
 def myMap = [key: value]
```

Besides fold, another common operation in Functional Programming is the map.

Given a collection, map returns another collection, such that each element of the new collection is derived from the
application of a function to its corresponding element from the initial collection.

In Groovy, we can use the `collect` method.

In the example, we used it to build a list of ages from an associative list of `Person`.

We could go even further and use the dot spreading `*.`, replacing `collect` by:

```groovy
println mapOfPeople*.value.age
```

Where for each entry in `mapOfPeople`, we access its `value`, and then
we get the `age` from it. It's similar to access each element
individually, but the same operation "spreads over" the entire collection.

# Final Example

This final example condendes pretty everything:

<script src="https://gist.github.com/rvarago/71864120440b30e479b3d7be34f57b31.js"></script>

Here, we sum the `amount` of `House`s sold.

That resembles a common task in daily programming, where we have some data, we
filter it, transform, and accumulate it somehow.

# Conclusion

We had an overview of some cool features provided by the Groovy programming language, but there's plenty of others! And I'd
encourage you to give it a try.

# References

[1] Groovy. <http://www.groovy-lang.org/>

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
