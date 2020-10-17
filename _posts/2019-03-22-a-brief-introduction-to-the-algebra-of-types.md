---
layout: "post"
title:  "A Brief Introduction to the Algebra of Types"
tags:   math adt fp c++ haskell
---

> It's been common to find talks and articles describing what an Algebraic Data Types (ADT) is. Particularly, we've been testifying sum types finally making their way into mainstream programming languages. We shall briefly discuss ADTs and understand how we can profit from this cool trend.

* * *

|![](/assets/img/2019-03-22-a-brief-introduction-to-the-algebra-of-types_0.png)|

**Disclaimer:**

> This post is a brief introduction to the topic from a Software Engineering's perspective, given that I'm a Software Engineer and not a Mathematician. Therefore I will not go too far into Mathematics behind the concepts. Further, I will probably abuse the notation and not be precise nor rigorous enough. However, I strongly encourage you to dig deeper into the topic, especially in Category Theory. It's not mandatory to understand the concepts, but IMHO it'll probably give you a better understanding.

* * *

At the beginning of Software Development, we were used to writing code in dialects of Assembly. As time went by and requirements became more complex, we've moved to high-level programmings languages. Those are meant to simplify our task of building flexible and maintainable software. High-level languages brought us to new levels of abstraction: procedural programming, structured programming, object-oriented programming, functional programming, generic programming, etc. Quite a few languages even support multiple styles of programming simultaneously.

Those programming languages allow us to encapsulate details and leverage code sharing.

However, there's a well-known programming language that has been around for quite some time and lots of wonderful things were built with it: **Mathematics**! It was created _by_ humans and _for_ humans. Mathematics grants us a vocabulary to communicate ideas. That's cool, isn't that?

When writing code, we're applying a plethora of concepts derived from Mathematics, even so, they might not be _that_ apparent, unless we look closely. By looking at things through the lens of Mathematics we may find new and useful properties that are ready to be explored and profit from.

That's what we shall be doing: catching all of the mathematical objects out there! Well, not close, just a tiny bit 😃.

The goal is to develop a basic intuition about what an Algebraic Data Type is. Algebraic data types are popular in functional programming languages, such as Haskell, but are gaining lots of traction in other languages (C++, Rust, Java, Swift, etc).

We will be discussing product and sum types, how we can use them as a vocabulary to build more complex types that better represent our problem state space in code. By doing so, we shall leverage the compiler to verify properties of the representation before the code gets the chance of being executed. To make things concrete, some pseudo-code written in Haskell and C++ will be provided.

## Fantastic Types and Where to Find Them

Understanding Algebraic Data Types implies that we first need to understand types.

What's a type?

A frustrating answer could be:

> Well, that depends… That depends on which perspective we looking at.

That's quite disappointing. Let's go deeper.

One possible way of thinking about types is:

> A type represents the way objects are stored in memory.

That's commonly referred to as the _representational_ view of types.

Although extremely useful for theoretical and practical aspects, the representational view is too low-level to fit out goal. We're
looking for a more abstract view of types, so that we would be able to do Mathematics with it.

This being said, another view of that comes closer to our needs is:

> A type is the set of possible values that an expression can inhabit. That's characterized by the **cardinality** of the given set, i.e. the number of elements (values of the type) in the set, which may be finite or infinite

What does that mean? Perhaps we should take the Boolean type as an example.

> How many values can I put in a variable of Boolean type?

Answer: two. They are `True` and `False`.

Picking a notation from Set Theory, we can write this statement as:

> Boolean = {x : x ∈ {True, False}} ⇒ #Boolean = 2

`#T = N` means "The cardinality of `T` equals to `N`".

Another example is the type `UInt8`:

>  UInt8 = {x : x ∈ {0, …, 255}} ⇒ #UInt8 = 256

  * How about the pair `(Boolean, UInt8)`?
  * How about Haskell's `Maybe a` or C++'s `optional<T>`?

Things get more interesting. It's about time we talk about the "Algebraic Data" part of Algebraic Data Types.

## Algebraic Data Types

Roughly speaking, an Algebraic Data Type (ADT) is a composite type (a type made of other types) created by "algebraic operations" abiding by some laws. We say Algebraic because the ways to combine types correspond to Algebraic Structures in Mathematics. This connection between types and mathematical structures is heavily exploited in Category Theory and Homotopy Type Theory.

We build ADTs by combining them in specific ways, such that the combination has implications on the properties of the composite types, which are reflected their cardinalities.

Two fundamental ways of building ADTs are products and sums, which we shall cover shortly.

### Product Types

> What if I told that you have been writing product types ever since C?

Going back to the question we raised before:

  * How many values can I put in the pair `(Boolean, UInt8)`?

We can (sort-of) enumerate the combinations:

> (True, 0); …; (True, 255); (False, 0); …; (False, 255)

Or, employing a terser notation, we end up with:

> (Boolean, UInt8) = {(x, y) : x ∈ Boolean ^ y ∈ UInt8}
>
> = Boolean * UInt8

This means that I get the combination of all values of type `Boolean` **AND** all values of type `UInt8` **at the same time**. The `*` represents the Cartesian product of `Boolean` and `UInt8`. The cardinality is:

> #(Boolean, UInt8) = #Boolean * #UInt8 = 2 * 256 = 512

That's the reason why it's called a product type:

> The cardinality of a **product** type equals to the **product** of the cardinality of its component types.

The general equation for the cardinality of a product type is:

> #(T1, …, TN) = #T1 * … * #TN

How about the following example that is written in pseudo-code:

```
User {active: Boolean, age: UInt8}
```

  * How many values can I put in a variable of type `User`?

The type `User` can be described as:

> User = {(active, age) : active ∈ Boolean, age ∈ UInt8}

In terms of values that I can put in a `User` and hence its cardinality, it's equivalent to the pair `(Boolean, UInt8)` that we previously saw. The list all possible values of `User` is the list of all possible values of `Boolean` AND all possible values of `UInt8`. Therefore the cardinality of `User` equals to 512 as well.

Fundamentally, those two types (pair and `User`) are _isomorphic_, or, equivalently, they are the same _up to the isomorphism_. That says we can go from one representation to the other and vice-versa, without losing information in the process.

Even plain structures correspond to ADTs, a bit of it.

### Sum Types

This one might not be so familiar. Fortunately, they've been starting to show up in mainstream programming languages and we should see more examples of sum types in our codebases.

By the definition of product type, we're probably guessing what a sum type is. Let's check if we've got it presently.

Consider the following enumeration:

```
enum Connection {Disconnected, Connecting, Connected}
```

  * How many values can I put in a variable of type `Connection`?

`Connection` can be described as:

> Connection = { Disconnected, Connecting, Connected}
>
> = Disconnected  \| Connecting  \| Connected

This means that the possible values of `Connection` are `Disconnected` **OR** `Connection` **OR** `Connected`, but **only one at a time**. This translates to the sum of the types. Thus the cardinality is:

> #Connection = #Disconnected + #Connecting + #Connected = 3

Hang on! Looking at this example we may say: Is `Boolean` a sum type?

Yes, it turns out that a `Boolean` is indeed a sum type!
Here's a possible implementation of a `Boolean` type in Haskell:

```haskell
data Boolean = True | False
```
It reads as "a value of type `Boolean` is **either** `True` **or** `False`".

In the same way, we could think of a `UInt8` as the composition of the values `0`, `1`, …, `255`.

That's why they are called sum types:

> The cardinality of a **sum** type equals to the sum of the cardinality of its component types.

A sum type corresponds to the disjoint union of its possible values.

The general equation for the cardinality of a sum type is:

> #(T1  \| …  \| TN) = #T1 + … + #TN

A famous sum type is Haskell's [`Maybe`](http://hackage.haskell.org/package/base-4.12.0.0/docs/Data-Maybe.html), which has similar semantics as C++'s [`optional`](https://en.cppreference.com/w/cpp/utility/optional), Rust's [`Option`](https://doc.rust-lang.org/std/option/), Swift's [`Optional`](https://developer.apple.com/documentation/swift/optional), Scala's [`Option`](https://www.scala-lang.org/api/current/scala/Option.html), etc.

`Maybe` may be defined in Haskell as:

```haskell
data Maybe a = Just a | Nothing
```

This reads as "a value of type `Maybe a` can be `Just a` OR `Nothing`".

This means that `Maybe a` is parametrized over a generic type `a`. Think of `a` as a placeholder to a concrete type, similar to C++'s templates, or Java's generics. `Maybe a` has two constructors (variants, choices, etc), it can either be a `Just a` (contains a value of type `a`) OR `Nothing`.

  * How many values can I put in a `Maybe a`?

That'd be the values that I can put in each constructor of `Maybe a`: `Just a` and `Nothing`. Thus:

> #Maybe a = #Just a + #Nothing

The cardinality of `Just a` depends on the type parameter `a`. `Just a` can contain precisely all possible values that `a` can contain, hence:

> #Just a = #a

`Nothing` can contain only one value: `Nothing` itself, which models the "absence of a value", hence:

> #Nothing = 1

Thus, the cardinality of `Maybe a`:

> #Maybe a = #a + 1

## Composing Product and Sum Types

The true power of ADTs arises from the combinations that we can create by treating product and sum types as principled building blocks.

ADTs offer a mechanism to design expressive abstractions on top of elementary components. These abstractions become part of the type system and therefore are verified at compile-time, which prevents some classes of bug from emerging later on, often in production.

A particularly interesting component that gets mixed in the composition of ADTs is the _recursive_ definition of data structures.

A recursive data type is a type whose definition is part of itself. A linked list is a common example, which can be expressed in Haskell as:

```haskell
data [] a = [] | a : [a]
```

Meaning that a generic linked list `[]` parametrized over `a` is either empty (`[]`), or (`|`) it contains an element of type `a` (head), concatenated by `:` with a sub-list `[a]` (tail). For instance, a list with elements `1` and `2` is represented as `[1,2]`, which is sugar for `1 : 2 : []`.

The definition shows that a linked list can be expressed a sum type. And the expression for its cardinality looks like the following recurrence relation:

> #List a = 1 + a * (#List a)
>
> #List a = 1 + a * (1 + a * (#List a))
>
> #List a = 1 + a * (1 + a * (1 + a * (#List a)))
>
> ...
>
> ⇨ #List a = 1 + a + a^2 + a^3 + ... + a^n 

That is:

> A list of elements of type `a` is either empty (`1`), or has one element (`a`), or two elements (`a^2`), or three elements (`a^3`), ..., or *n* elements (`a^n`). 

Further, a binary tree can also be expressed as a recursive data type in Haskell:
    
```haskell
data BinaryTree a = Leaf a | Node (BinaryTree a) a (BinaryTree a)
```

Meaning that a generic binary tree parameterized over `a` is either a `Leaf a` (with elements of type `a`), or a `Node` (with a left sub-binary-tree), an element of type `a`, and a right sub-binary-tree (with an element of type `a`).

This definition of a binary-tree shows that we can combine sum and product types in many different ways.

> Challenge: Write down the expression for the cardinality of `BinaryTree a`.

## Expressing Business Logic in the Type System

Error handling is not trivial.

It's easy to forget to cover error paths with checks and even trickier to handle errors properly when we catch them. It'd be nice if we could handle errors by simply avoiding them in the first place.

Of course, it's impossible to avoid all sources of errors. Yet, can avoid _some_ errors.

The idea is:

> Catch errors as early as possible. Ideally at compile-time, before the code gets the chance of being executed and thereby show erroneous behaviour.

Rationality:

> What can be done at compile-time should (probably?) be done at compile-time.

We can achieve this by making assumptions explicit in the type system. Thus, at compile-time, the type-checker ensures that **illegal states are not expressible**.

How feasible this is? Ultimately, that depends on the actual scenario and on how expressive a type system offered by the language is.

Consider the following, possibly stretch, C++ snippet:

<script src="https://gist.github.com/rvarago/f8e89ae923a7adfe20213598b22e69d1.js"></script>

Let's assume for the sake of an argument that `Name`, `Age`, `WorldMap`, `Weapon`, and `Pillow` are classes declared somewhere else and their details are not relevant to us.

We have a class `Hero`, which can be in one of three disjoint states coordinated by the enumeration `State`. In terms of ADTs we saw above, we can express `Hero` as a combination of products and sums types:

> Hero = State * Name * Age * WorldMap * Weapon * Pillow
>
> = (EXPLORING \| FIGHTING \| SLEEPING) * Name * Age * WorldMap * Weapon * Pillow

This expression says that `Hero` is `Name` AND `Age` AND `WorldMap` AND `Weapon` AND `Pillow` AND one of `EXPLORING` OR `FIGHTING` OR `SLEEPING`.

However, `Hero` has attributes that only make sense when in a particular state and not in the others. For example: Why should a hero have a pillow when he's fighting (pillow wars?)? Or why should a hero have a world map when he's sleeping (The grant map of dreams?)?

That raises the questions:

  * How can we make sure that a hero will not access an attribute that doesn't make sense for his current state?
  * What should happen with the attributes that aren't valid in a state when the hero is in this exact state?

We could use pointers that should be set to `nullptr` to represent "absence of value" if they aren't at the expected state.
However, this approach has the potential of dragging us to Undefined Behaviour if we try to access a pointer when we shouldn't. Sprinkling `if` checks probably solves the problem, but those are only performed at run time and, as I said, we may miss some cases.

A more explicit way to express the absence of value would be to use `std::optional<T>` (or equivalent) as we discussed [here.]({{ site.baseurl }}{% link _posts/2018-10-29-from-null-to-option-in-scala.md %}). However, that would simply patch the issue up, rather than properly fix it. We would enlarge the set of possible values, whereas we should be shrinking it.

Essentially, the question is:

  * Should we _really_ have a product of `WorldMap`, `Weapon`, and `Pillow`?

I'd say no. A `Hero` should be **one of those, but only one at a time**.

This example reveals a mismatch between reality (problem state-space) and code (model state-space). This happens due to the enumeration, which describes the current state, not being strongly tied to the attributes of each state. The type system is not aware that such association between states and attributes even exists, without this precious information it can't help us out.

We need to bind them together in such a way that they're always synced (we don't want to change the state without updating the attributes). Moreover, this association has to be expressed in the type system, so the type checker shall enforce this requirement and therefore relieve us from some checks at run time.

In C++, we can do the trick with [`std::variant<Ts...>`](https://en.cppreference.com/w/cpp/utility/variant), which is available since C++17. A variant is a sum type implemented as a tagged union that encapsulates the pattern "union + tag (discriminator)".

The union part means that a variant can hold many types, but only one at a time. The tag distinguishes which type is active at a time. `std::variant<Ts...>` exposes a type-safe, albeit verbose, API to query and use the type held in it.

As a [variadic-template](https://en.cppreference.com/w/cpp/language/parameter_pack), an `std::variant<Ts...>` can hold one of any number of types, e.g. for types: `A`, `B`, and `C`:
    
```cpp
auto v = std::variant<A, B, C>{}
```

Meaning that:

> v = A \| B \| C => #v = #A + #B + #C

So, `v` can be `A` OR `B` OR `C` at a time.

Revisiting our previous `Hero`example equipped with `std::variant<Ts...>` we can finally bridge the gap between reality <-> code and pack state and its attributes:

<script src="https://gist.github.com/rvarago/4ad2bd6b04dd81c902465613eb3fae9d.js"></script>

We now have three inner structures, each consolidating state and its corresponding attributes. Example, `WorldMap` is only available in the `Exploring` state, it not shared and therefore cannot be accessed by the `Fighting` or `Sleeping` state.

The most important aspect of this new design is:

> The impossibility of accessing attributes that we shouldn't in a given state is now **witnessed in the type system**.

By design, it becomes impossible to violate the constraint. If we attempt to break the assumption, then the compiler will yell and stop us.

Now, our brave `Hero` is:

> Hero = Name * Age * State
>
> = Name * Age * (Exploring \| Fighting \| Sleeping)
>
> = Name * Age * (WorldMap \| Weapon \| Pillow)

It reads as: "`Hero` is `Name` AND `Age` AND **one of these**: `WorldMap` OR `Weapon` OR `Pillow`". We've bound state and attributes closer to each other. Therefore, a hero always has access to exactly what he should have had at one time, nothing more and nothing less.

The constraint is not just a (hopefully up-to-date) comment in the code, a statement in the design document, or an idea that only lives in our mind, etc. The constraint is rather "carved in the stone of our type system". It shall be check for validity by the compiler on every single time we attempt to compile the code, and if we attempt to violate it then the compiler (in contrast to the end-user), will happily yell and point us to the mistake.

Oh, by the way, `std::variant<Ts...>` might be an interesting alternative design to consider when implementing the [State Pattern](https://en.wikipedia.org/wiki/State_pattern).

### Bonus: Visitor Pattern

If we want our `Hero` to act depending on the state, we could then combine `std::variant<Ts...>` with the [Visitor Pattern](https://en.wikipedia.org/wiki/Visitor_pattern) implemented as [`std::visit`](https://en.cppreference.com/w/cpp/utility/variant/visit):

<script src="https://gist.github.com/rvarago/c016e53a2a0512015bcd84822a45e4c8.js"></script>

And the right overload is selected based on the `Hero`'s state.

 The Poor's Man pattern matching, we may say 😉.

## Conclusion

We've merely scratched the surface of ADTs. We've introduced some of the notation from Set Theory and focused on the intuition behind ADTs instead of the Mathematics that back them up.

Apart from product and sum types, there are other ADTs (e.g. exponential and derivative), but I'd rather omit those in this introductory (and already lengthy) text, given that products and sums should provide a good framework to get us started on the matter.

> There's even a far deeper and beautiful connection between ADTs and Logical Connectives, namely "Propositions as Types". See [Programming Language Foundations in Agda](https://plfa.github.io/Connectives) for reference.

Applications for ADTs are many, they allow us to model requirements in the type system, rather than relying solely on checks that are carried in run time.

In the end, the goal is to take the type system into our best advantage. Thus the more expressive a type system is, the higher is the chance that it'll help us to prevent some (but surely not all) mistakes.

Compilers can't catch all kinds of mistakes, but they can catch some of them, which is far better than nothing.

I'd encourage you to go through the references to acquire a deeper understanding of the subject. Spoiler alert: there's a quite interesting
duality relationship between products and sums.

I hope that this brief discussion will be of some help in the science/engineering/art of designing solid, lasting, and clean abstractions.

Eventually, I expect to go through topics in futures posts. Perhaps covering things like phantom types, pattern matching, etc.

Stay tuned.

## References

[1] [Algebraic data type - Haskell Wiki.](https://wiki.haskell.org/Algebraic_data_type)

[2] [Algebraic data type - Wikipedia.](https://en.wikipedia.org/wiki/Algebraic_data_type)

[3] [Category Theory for Programmers. Bartosz Milewski.](https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/)

[4] [Basic Category Theory. Tom Leinster.](https://www.cambridge.org/core/books/basic-category-theory/A72533879BBC7BD956CC415777B7DA99)

[5] [Programming Language Foundations in Agda.](https://plfa.github.io/Connectives)

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
