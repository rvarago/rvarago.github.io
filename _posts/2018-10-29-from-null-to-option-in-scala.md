---
layout: "post"
title:  "From null to Option in Scala"
tags:   adt fp scala
---

> NULL, nullptr, null, etc. have been used to represent the absence of values for many years, but there are better alternatives to model this scenario. In Scala, the answer is usually Option.
 
* * *

|![Scala.](/assets/img/2018-10-29-from-null-to-option-in-scala_0.png)|
|:--:| 
| *Scala. Source: <https://github.com/OlegIlyenko/scala-icon/blob/master/README.md>.*|

# Introduction

When we call a piece of code (e.g. a function) we expect that it'll return a **meaningful** value, for example:

  1. Fetch a user from some storage by name..
  2. Access the fetched user's role
  3. Print the name of the retrieved roles.

Sounds pretty decent, right?

In Scala, we might come up with the following solution as our first attempt:

<script src="https://gist.github.com/rvarago/cdd35607dc3c62cc0d9b07bea33c64ad.js"></script>

What do you think? Is it right? The answer is: well, it depends! What?? :S

  * Is it expected to call `findUser` with a `name` that doesn't correspond to any `User`?

If you answered "yes" to the above questions, then this solution may have some drawbacks and would lead to surprises.

On the happy path, `findUser` returns an instance of `User` that will then be consumed by `getRoles` and subsequently by `printRolesFromUser`.

However, if we didn't find a user, what should `findUser` return? That's a failure, or disappointment, and how should we to model it? The common, and perhaps not so idiomatic solution in Scala, is to represent such absence of a value with `null`, which is value accepted by all references. We can think of `null` as a reference pointing to nowhere, like or an empty box.

From the `printRolesFromUser`'s perspective, by inspecting the signature of the method, it can't tell for sure what happens in the case of a failure.

  1. Maybe `findUser` returns an "empty" `User`.
  2. Maybe it returns `null`. However, accessing a property from `null` will trigger a `NullPointerException` (NPE).

Our interface might be violating the [Principle of Least Astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment). Unless it, at least, explicitly document this behaviour in a comment, therefore clients have to read the documentation. Moreover such documetation has to maintained over time and we must be careful to keep it up-to-date, few things are worse than a comment that lies.

Suppose that we do return `null`, what handle it then?

Then, we have our second attempt:

<script src="https://gist.github.com/rvarago/ee4a2fcbd61f29416983666ef958a5a3.js"></script>

The NPE is fixed, but now a simple and elegant one-line function has a few more lines of code, a higher [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity), and error-handling is interviwened with bussiness logic. And that is mostly due to the check against `null`. That's not too bad. Nevertheless, it's a bit noisy and might become even noiser as the application grow.

How can we fix the NPE while keeping error-handling separated from the bussiness logic?

# The right Option for the right semantic

To tackle the problem, we first need to understand its root cause:

Our interface doesn't express the possibility for the **value to be absent**, which **is an expected case in our application**. By _expected_, I mean to emphasize that **it's not an exceptional situation**, hence throwing an `Exception` might not be the most idiomatic choice in Scala.

We want the **interface** to be explicit and state that it may not return a meaningful value. So that clients will be aware of it, and therefore forced to handle the disappointment properly according to its goal.

We used `null`, but as we discussed, `null` has a few problems:

  1. We can't express the possibility of absence of the value in the interface, only in the implementation.
  2. We're not forcing our clients to handle the failure, and therefore they may forget to do it.

From my perspective, the fundamental issue is that _null_ is a value that can inhabit all types, and further, this happens **implicitly**.

Therefore, clients have to look at the implementation (not always feasible or desired), or rely on the documentation. As I said, documentation has to be maintained: it isn't compiled, can get outdated, and nothing ensures that clients will read it.

> Prefer to document the intent of a piece of code in the code itself, especially in its public API.

Hence, it would be nice if we could express the "absence of value" in the interface. The Scala way to handle this is by using the `Option`
[monad](https://en.wikipedia.org/wiki/Monad_%28functional_programming%29).

`Option` is a type that models a value that may or may not be available, and that's exactly what we need! By stating in the interface
that our function returns an `Option`, we're **communicating** to our clients, we're being explicit and forcing them to handle the case, or at least acknoledge it. That leads to less surprising behaviour, which is an advantage of writing clearer and stronger interfaces, so that it's **easier to use correctly**. As well said by Scott Meyers's in Effective C++ Item 18:

> Make interfaces that are easy to use correctly and hard to use incorrectly.

In practical terms, `Option` has two cases (possibilities or "sub-classes"):

  * `Some(x)`: Value is present and wrapped in `x`.
  * `None`: Value is absent.

That's is exactly what we need! By changing the interface to return an `Option`, we're sending a clear message to our clients: be prepared to handle the case where I failed to return you a meaningful value. In this case, clients shall receive `None`. As I've said before: no more surprises!

> `Option` forms a **monad**, a pretty powerful concept, which is central to Category Theory and largely applied in Function Programming. Roughly speaking, it's a type that offers two operations: `identity` (a.k.a `unit`) and `bind` (a.k.a `flatMap`). Unfortunately, a deep discussion about monads is far beyond the scope of this text, but I may get back to it in a future opportunity.

The simplest way to consume an `Option` is by invoking the `getOrElse` method, which returns the wrapped value when invoked from a `Some`, or returns the fallback value supplied as an argument when invoked from a `None`:

<script src="https://gist.github.com/rvarago/0f486a71c115c3d7c6d03c844eef84de.js"></script>

A few properties of `Option`:

  * Works pretty well with pattern matching.
  * Has great support for composition (`map`, `flatten`, etc).

For instance:

<script src="https://gist.github.com/rvarago/9e24017cf7b8c83a28d477b868694ada.js"></script>

Furthermore, `Option` supports _for-comprehension_  (thanks for it being a monad!),which is nice a syntactic sugar that condenses a sequence of `flatMap`, `map`, `withFilter`, and `foreach`. And that's precisely how we may solve our initial problem:

<script src="https://gist.github.com/rvarago/31a64b3603a07a5219db9100d879b7be.js"></script>

The check against `None` has been pushed into the _for-comprehension_. It'll exit the loop if `findUser` returns `None`. Otherwise, it retrieves the `User` that is used in the `yield` clause by the `foreach`.

# Conclusion

Although `null` is extensively used and has its value, we have alternative approaches to express its intent: absence of a value. Particularly, Scala has the `Option` monad, which serves a similar purpose. Moreover, it more clearly expresses in the interface that the operation may not be able to produce a meaningful value, being then explicit to clients.

By using `Option`, we elegantly reduced the chance of ending up with a `NullPointerException`, or the noise associated with `null`
checks. The goal was to express what a piece of code means by employing the proper grammar made available by the programming language. Remember: Humans should be the first client of your code, then the compiler.

Other languages offer variations of `Option`, Java has `java.util.Optional`, C++ has `std::optional`, Haskell has the super powerful `Maybe`, etc.

As always in Software Engineering, there is no "one solution to solve them all", and this very well applies for `Option` too. Sometimes a different approach might fit better, for instance, Null Object Pattern, `Either`, `Try`, or even an plain and old exception. Therefore, it worthwhile to known our tools, so that we can be ready to solve problems with the best approach.

# References

[1] Martin Odersky. "Programming in Scala".

[2] Dean Wampler, Alex Payne. "Programming Scala: Scalability = Functional
Programming + Objects".

[3] Scott Meyers. "Effective C++".

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
