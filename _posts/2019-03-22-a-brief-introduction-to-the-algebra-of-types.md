---
layout:	"post"
title:	"A brief introduction to the Algebra of Types"
---

It's been fairly common to find talks and articles about algebraic data types,
things like sum and product types are becoming popular in mainstream
programming languages nowadays.

* * *

![](/assets/img/2019-03-22-a-brief-introduction-to-the-algebra-of-types_0.png)

> It's been fairly common to find talks and articles about algebraic data
types, things like sum and product types are becoming popular in mainstream
programming languages nowadays. So let's try to understand what they are
about.

 ** _Disclaimer:_** _This post is a brief introduction to the topic from a
Software Engineering 's perspective, given that I'm a Software Engineer and
not a Mathematician. Therefore I'll avoid deepening too much in the
Mathematics behind it. However, I strongly encourage you to dig deeper into
the topic, especially in Category Theory. It's not mandatory to understand the
concepts, but IMHO it'll probably give you a better understanding._

At the beginning of Software Development, we used to write code in dialects of
Assembly. As time passed and requirements became more complex, we've moved to
high-level languages in order to simplify our task of writing flexible and
maintainable software. Each language has added some new levels of abstraction:
procedural programming, structured programming, object-oriented programming,
functional programming, etc. Some languages even support multiple styles of
programming.

Those languages have the goal to hide some irrelevant details from the daily
and average programming task.

However, there's one old and fairly well-known language that has been used to
build the most wonderful things: **Mathematics**! It was created _by_ humans
_for_ humans. It gives us a vocabulary to communicate ideas. That 's cool,
isn't it?

When we're writing code, we're using a plethora of concepts derived from
Mathematics. They aren't immediately evident to us but trust me, they're
there. So, let's catch them all! Well… At least some of them 😃.

Our goal is to develop a basic intuition about what an Algebraic Data Type is.

We're going to discuss product types and sum types. How we can use them as a
vocabulary to build more complex types to better represent our problem state
space in our code, and leverage the compiler to verify the properties of this
representation before the code gets the chance of being executed. To exemplify
things in code, some pseudo-code based on Haskell and C++ is going to be
provided.

#### Fantastic Types and where to Find Them

First and foremost, to understand Algebraic Data Types we need to understand
types. Okay, what's a type?

One possible answer would be:

> It depends… It depends on your intentions.

That's not helping at all. So let's go deeper.

One way of thinking about types is:

> A type represents the way objects are stored in memory.

And that's commonly called the _representational_ view of types.

Besides being absolutely useful for several theoretical and practical matters,
the representational perspective is too low-level for our purposes here. We're
actually looking for a more abstract view of types, so we would be able to do
some Mathematics with it.

Another view of types which is more related to our purposes would then be:

> A type represents the set of possible values that can inhabit in an
expression. And it's characterized by the **cardinality** of this set, ****
i.e. the number of elements (values of the type) in it, which may be finite or
infinite

What does it mean? Let's take a Boolean type as an example.

  *  _How many values can I put in an object of type Boolean?_

Answer: two. They are: _True_ or _False_.

Using a notation borrowed from Set Theory we can write this as:

> Boolean = {x : x ∈ {True, False}} => #Boolean = 2

 **NOTE:** _#T = N_ reads as:  "The cardinality of _T_ is equal to _N_ ".

Another example is a type _UInt8_ :

>  _UInt8_ = {x  : x ∈ {0, …, 255}} => #UInt8 = 256

  * How about the pair _(Boolean, UInt8)_?
  * How about the Haskell's Maybe or C++'s optional?

Things are getting more interesting… It's time to talk about the Algebraic
Data part of the name Algebraic Data Type.

#### Algebraic Data Types

An Algebraic Data Type (ADT) is roughly the name given to composite types
which are, as the name suggests, composed of other types following some
elementary combination patterns.

We say Algebraic because the ways to combine types have an equivalence in
terms of Algebraic Structures in Mathematics. This connection is heavily
exploited in a Mathematical domain called Category Theory.

We build an ADT by combining them in some ways, and this combination has
implications over the composite type's properties and it's reflected in its
cardinality.

Let's see two elementary ways to combine types, i.e to build ADTs: products
and sums.

#### Product Types

What if I told that you have been writing product types ever since C?

Yes, you've probably been doing so.

Going back to the question above:

  * How many values can I put in the pair _(Boolean, UInt8)_?

I can put:

> (True, 0); …; (True, 255); (False, 0); …; (False, 255)

Or using a more terse notation:

> (Boolean, UInt8) = {(x, y) : x ∈ Boolean, y ∈ UInt8}

> = Boolean * UInt8

This means that I can have all the combinations of _Boolean_ **AND** all the
combinations of _UInt8_ **all at the same time**. __ And * is the Cartesian
product of _Boolean_ and _UInt8_. Then the cardinality becomes:

> # _(Boolean, UInt8) = #Boolean * #UInt8 = 2 * 256 =  512_

And that's why it's called product type:

> The cardinality of a **product** type equals to the product of the
cardinality of its component types.

The general equation for the cardinality of a product type is:

> #(T1, …, TN) = #T1 * … * #TN

How about the following example that is written in pseudo-code:

 _User {active: Boolean, age: UInt8}_

  * How many values can I put in the _User_?

The type _User_ can be described as:

> User = {( _active_ , _age_ ) : active ∈ Boolean, _age_ ∈ UInt8}

So, in terms of values that I can put in it and hence its cardinality, it's
equivalent to the previous example about pair. To list all the values of
_User_ we have to list all the values of _Boolean_ AND all the values of
_UInt8_. So the cardinality of _User_ also equals to the 512.

See? Even a plain and old structure is an ADT.

#### Sum Types

This one might not be so familiar, but fortunately, they've been starting to
appear in mainstream programming languages and we should see an increase of
sum types in our code bases.

By the definition of product type, you're probably correctly guessing what a
sum type is. Let's check if you're correct in a moment.

Consider the following enumeration:

 _enum Connection {Disconnected, Connecting, Connected}_

  * How many values can I put in the _Connection_?

It can be described as:

> Connection = { _Disconnected_ , _Connecting, Connected_ }

> = Disconnected | Connecting | Connected

This means the value of _Connection_ can be _Disconnected_ **OR** _Connection_
**OR** _Connected_ , but **only one at a time**. And | means the sum of the
types. So the cardinality is:

> #Connection = 3

Hang on! Looking at this example you may say: Is _Boolean_ a sum type? Yes, it
is! Actually, here 's a possible implementation for a _Boolean_ type in valid
Haskell code:

> data Boolean = True | False

It reads as "a _Boolean_ is **either** _True_ **or** _False_ ".

In the same way, we could think of an _UInt8_ as being the composited of the
values _0, 1,  …, 255_. And that's why it's called sum type:

> The cardinality of a **sum** type equals to the sum of the cardinality of
its component types.

A sum type corresponds to the disjoint union of its values.

The general equation for its cardinality is:

> #(T1 | … | TN) = #T1 + … + #TN

One very famous sum type is Haskell's
[_Maybe_](http://hackage.haskell.org/package/base-4.12.0.0/docs/Data-
Maybe.html), which has similar semantics as C++'s
[_std::optional_](https://en.cppreference.com/w/cpp/utility/optional), Swift's
[_Optional_](https://developer.apple.com/documentation/swift/optional),
Scala's [Option](https://www.scala-lang.org/api/current/scala/Option.html),
etc.

 _Maybe_ can be defined as:

> data Maybe a = Just a | Nothing

This reads as: " _Maybe_ can be _Just a_ OR _Nothing_ "

This means that _Maybe_ has a parametric type called _a;_ think of it as a
placeholder to a concrete type, similar to C++'s templates, or Java's
generics. And it can be either _Just_ in this case it contains a value of type
_a_ OR _Nothing_.

  * How many values can I put in the _Maybe_?

It'd be the same of the values that I can put in each variant of _Maybe_ :
_Just a_ and _Nothing_. Thus:

> #Maybe a = #Just a + #Nothing

The cardinality of _Just a_ depends on the type of the parameter _a_. It can
contain all the values that _a_ can contain, so _#Just a_ equals to _#a_.

And _Nothing_ can contain only one value: _Nothing_ , which represents the
absence of any value, so #Nothing = 1.

Hence, the cardinality of _Maybe a_ is:

> #Maybe a = #a + 1

#### Composing Product and Sum Types

The true power of ADTs comes from the plethora of combinations that we can
create by grouping product and sum types as basic building blocks.

ADTs are a mechanism to create expressive abstractions by composing simple
elements, and these abstractions become part of the type system; therefore,
they're verified at compile time which avoids the late discovery of some
classes of errors.

Another component that is mixed in the composition of ADT is the **_recursive_
definition of data structures**. Those are data structures can be roughly
thought as a type whose definition is part of itself.

One common example is a linked list, that can be expressed in Haskell as:  
`data [] a = [] | a : [a]`

Which means that a generic list _[]_ parametrized by _a_ is either empty, or
it contains an element of type _a_ called head, concatenated by _:_ with
sublist _[a]_ called tail. This definition shows that a list is a sum type.

A more elaborate example would be the definition of a binary tree, which is
another recursive type, since a tree may contain subtrees, composed of basic
ADTs. We can represent a binary tree in Haskell as:

    
    
    data BinaryTree a = Leaf a | Node (BinaryTree a) a (BinaryTree a)

Which means that a generic binary tree parameterized by the parameter _a_ is
either a _Leaf_ with an element of type _a_ , __ or a _Node_ containing a left
_BinaryTree_ , an element of type _a_ , and a right _BinaryTree_. This
definition shows that a binary tree is a combination of the sum and product
types.

#### Expressing the business state space in the type system

It's hard to get error handling correctly. It's pretty easy to forget to check
for errors, and not so easy to handle when we catch them.

It'd be nice if we could handle errors by avoiding errors in the first place.

Of course, it's impossible to avoid all kinds of errors, but we can avoid some
of them.

The idea is to catch some errors as early as possible, ideally at compile
time, before the code gets the chance of being executed and thereby show the
erroneous behaviour. Bear in mind:

> What can be done at compile time should, in most cases, be done at compile
time.

One method to achieve this is by lifting assumptions in the type system in
such a way that it makes sure that **illegal states aren 't expressible** in
the type system.

That's even easier to achieve if the programming language offers is equipped
with an expressive type system.

Consider the following snippet:

 _NOTE:_ _Name_ , _Age_ , _WorldMap_ , _Weapon_ , and _Pillow_ are classes
declared somewhere else and their details aren 't relevant for us here.

We have a class _Hero_ that can be in one of three different states depending
on the enumeration _State_. We can express _Hero_ in terms of Set Theory as a
combination of products and sums types.

> Hero = State * Name * Age * WorldMap * Weapon * Pillow

> = (EXPLORING | FIGHTING | SLEEPING)

> * Name * Age * WorldMap * Weapon * Pillow

It says that _Hero_ is _Name_ AND _Age_ AND _WorldMap_ AND _Weapon_ AND
_Pillow_ AND one of these _EXPLORING_ OR _FIGHTING_ OR _SLEEPING_.

However, _Hero_ has attributes that only make sense for one state and not for
others. For example: Why should a hero have a pillow while he 's fighting? Or
why should a hero have a world map while he's sleeping?

It immediately raises the questions:

  * How can we make sure that a hero will not access an attribute that doesn't make sense for his current state?
  * What should happen with attributes that aren't valid in one state when a hero is in this given state?

We could use pointers that should be manually set to _nullptr_ to represent
the absence of value if they aren 't valid at one state, but this can yield
exceptions if we try to access it when we shouldn't. Adding checks might solve
the problem, but they're only performed at run time and it's easy to forget
some cases. A more explicit way to express the absence of value would be to
use _std::optional_ as we discussed [here](https://code.egym.de/from-null-to-
option-in-scala-3436cfeef7b0). However, the fundamental problem lies in the
following question:

  * Should we have a product of _WorldMap_ , _Weapon_ , and _Pillow_?

I'd say no. It should be **one of those, but _only one at a time_**.

This example shows a classical mismatch between the reality (problem state
space) and our code (model state space). And it happens because the
enumeration that controls the state is not strictly associated with the
attributes that belong to each state, the type system is not aware that such
association even exists, so it can't help us.

We need to somehow tie them together in such a way that they're always synced.
Moreover, this association has to be expressed in the type system, so it can
enforce this requirement and therefore relieve us from needless checks at run
time.

In C++ we can do this with
[_std::variant_](https://en.cppreference.com/w/cpp/utility/variant) which is
available since C++17. A variant is a sum type defined as a tagged union that
encapsulates the pattern "union + tag".

The union part means that a variant can hold many types, but only one at a
time. And the tag is used to distinguish which one is actively being used. The
_std::variant_ exposes a type-safe API to query and use the type being held in
it.

Being a [variadic-
template](https://en.cppreference.com/w/cpp/language/parameter_pack), in
theory, an _std::variant_ can hold any number of types, let 's see an example
using three: _A_ , _B_ , and _C_.

    
    
    auto v = std::variant<A, B, C>{}

This means that:

> v = A | B | C => #v = #A + #B + #C

So _v_ can be _A_ OR _B_ OR _C_.

It's worth to mention that a lot of languages offer sum types that behave more
or less likes _std::variant_ , possibly under different names (e.g. _choice_
).

Let's revisit our previous example equipped with _std::variant_ and close the
gap between reality and code, by tying the state and its attributes together:

We've introduced three inner structs and each one consolidates the state and
the attribute that only makes to the referred state. Example, _WorldMap_ is
now only part of the _Exploring_ state, it isn 't shared and can't be accessed
by the _Fighting_ or _Sleeping_ state.

The most important aspect of this new design is that:

> The impossibility of accessing attributes that we shouldn't in a given state
is now **encoded in the type system**.

Thus, it's impossible to violate the constraint. And if we attempt to do so,
the compiler will be the one to stop us, which is usually better than an
exception only possibly caught at run time.

Our _Hero_ now becomes:

> Hero = Name * Age * State

> = Name * Age * (Exploring | Fighting | Sleeping)

> = Name * Age * (WorldMap | Weapon | Pillow)

It reads as: "A _Hero_ has _Name_ AND _Age_ AND **one of these** : _WorldMap_
OR _Weapon_ OR _Pillow_ ". So, it groups the state and its attributes
together. Therefore, a hero always has precisely what he's meant to have at
one time, no more and no less.

It's important to emphasize that this constraint is not a comment in the code,
a phrase in the design document, or an idea in our head, it's engraved in the
type system. It shall be validated by the compiler each time we compile the
code, and if we try to violate this constraint the compiler, and not the end
user, is going to immediately point us the mistake.

By the way, _std::variant_ might be an interesting alternative to consider
when implementing the [State
Pattern](https://en.wikipedia.org/wiki/State_pattern).

 **Bonus:**

If we want our _Hero_ to perform an action depending on his state, one way to
do this using _std::variant_ is by the [Visitor
Pattern](https://en.wikipedia.org/wiki/Visitor_pattern) implemented as
[_std::visit_](https://en.cppreference.com/w/cpp/utility/variant/visit):

And the right overload is selected based on the _Hero_ 's state.

#### Conclusion

This article touched the surface of the vast topic of ADT. We've used some
notation from Set Theory and focused more on the intuition of ADTs instead of
the Mathematics behind it.

Besides product and sum types, there are other ADTs like exponential,
derivative, etc, but I'd rather omit those at first since the ones we talked
about today give us a good framework to build useful abstractions on a daily
basis for programming.

The applications for ADTs are many, they allow you to model the requirements
in the type system, instead of checks in run time. In the end, the goal is to
use the type system in our favour, so the more expressive it is, the greater
is the chance that it'll help us to commit mistakes in compile-time.

Once again it's important to repeat what I've said before: the compiler can't
prevent us from committing all kind of mistakes, but it can catch some of
them, which is far better than nothing.

I'd encourage you to go through the references to acquire a deeper
understanding of the subject. Spoiler alert: there's a very interesting
duality relationship between products and sums.

I hope that you can take value from this article and that it helps you to
write abstractions that simplify your coding tasks.

Of course, we're also going to briefly discuss their importance. However, I'll
let most of this part of the topic to upcoming posts, things like phantom
types, pattern matching, etc. So stay tuned.

#### References

[1] Basic Category Theory. Tom Leinster.

[2] Category Theory for Programmers. Bartosz Milewski.
<https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-
preface/>

[3] Algebraic data type. <https://en.wikipedia.org/wiki/Algebraic_data_type>

