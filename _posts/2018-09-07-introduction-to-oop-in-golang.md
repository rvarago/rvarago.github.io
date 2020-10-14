---
layout: "post"
title:  "Introduction to OOP in Golang"
tags:   oop golang
---

> Getting started with Object-Oriented Programming (OOP) in Golang (Go).

* * *

|![An object-oriented Gopher.](/assets/img/2018-09-07-introduction-to-oop-in-golang_0.png)|
|:--:| 
| *An object-oriented Gopher. Source: <https://github.com/egonelbre/gophers>.*|

# User-Defined Types

Go supports the notion of user-defined types, that is, the composition of native types (like int, string, etc.) to build more complex types reflecting the concepts of the real world like person, car, bank account, etc. or
abstract concepts like requests, processes, etc.

In Go, we declare user-defined types with the `struct` keyword. For example, to define a type `Employee` with attributes: `id`, `name`, and `salary` we might do:

<script src="https://gist.github.com/rvarago/82a07c08693a0000cbc3f483398d5eb2.js"></script>

Differently from languages like Java, Go's `struct` only contains state (attributes), thus we can't directly write methods (behavior) inside the
`struct`. Instead we rely upon a special syntax for functions with a special argument, the receiver.

For example, to add a method called `raiseSalary`, which receives a `bonus` argument and mutates an instance of type `Employee`; and another method `format`, which only reads from an instance of type `Employee`:

<script src="https://gist.github.com/rvarago/71a306c93c99d28b8a3f5269c9f2b467.js"></script>

Notice that the name given to the receiver can be any valid identifier, I just chose `self` to illusrate, but we could very well have chosen a better and/or shorter name. Additionally, notice that in `raiseSalary`, the receiver is a **pointer** to an `Employee`, that is needed as we want to mutate `self.salary`.

# Encapsulation

The notion of encapsulation is expressed in Go by **packages** that are, up to some extent, similar to Java's packages and C++'s namespaces. However, in contrast to Java and C++, Go doesn't provide keywords to express access modifiers (_private_, _public_, etc.), instead it distiguinshes between private and public by the case of the symbol's first letter. Where:

  * Uppercase: _public_.
  * Lowercase: _private_.

A private symbol is only accessible inside the package where it is defined, whereas a public symbol is also visible outside its package.

The package name is declared at the start of each file and must be the same of its directory name in the file system. You can also have multiple files inside a single package. A package `employee` might look similar to:

<script src="https://gist.github.com/rvarago/1cb4a942762fb5ac3af054120798a268.js"></script>

Moreover, a package has a well-defined order of initialization:

  1. Imported packages.
  2. Variables local to the package.
  3. `init()` functions.

# Composition and Inheritance

Go doesn't support inheritance, or _is-a_ relationships. It rather favours composition over inheritance, and so attempts to prevent some classic problems that may arise from the abuse of inheritance that occurs with bad hierarchies.

Go allows us to embedded nameless `structs` (or `interfaces`, see below) inside other `structs` (or `interfaces`). The embedding _struct_ (or interface) has direct access to the members of the embedded `struct` (or `interface`) via promotion. For example, we may like to model a type `Manager` that _is-sort-of-a_ `Employee` (oh, boy):

<script src="https://gist.github.com/rvarago/376fed717be529e926dc90d9059aad77.js"></script>

Then we can call methods defined for `Employee` with an instance of `Manager`. Although we can't call a function expecting an `Employee` with an instance of `Manager`. That avoids implicit conversions and hence surprising behaviours, so there isn't a strict _is-a_ relationship.

# Interface and Polymorphism

Interfaces provide much of the benefits that we would otherwise have with inheritance. That is with more restrictions and therefore fewer surprises or edge-cases.

An interface establishes a contract that every type must abide by if they mean to implement such an interface. In Go, if a type implements all the methods defined by the interface, this type is said to adhere to the interface contract and can be used where an interface is expected.

Let's say that an `Employee` type has a role, so we can make it implement the contract established by the `interface` `Role`:

<script src="https://gist.github.com/rvarago/4c965b0b0bd66eda441ae22f757e059f.js"></script>

When `printRoleName` is called, the type `Employee` is implicitly converted into the`_interface` `Role` that it satisfies. Thus, we're achieving run-time polymorphism with late binding.

Go is fairly different from many languages in this regard, where in other languages would commonly favour nominal-typing:

> A type `T` implements the interface `I` if it explicity spells `I`'s name (e.g `T : I`, `T implements I`, etc); and `T` implements the methods defined by `I`.

In contrast, Go prefers structural-typing:

> A type `T` implements the interface `I` if `T` implements the methods defined by `I`.

Therefore we don't need to explicitly state that a type implements an interface, it only needs to define its methods and that's all, it's implicitly implemented the interface.

A neat consequence is that we can create the interface after having introduced type, so that we can generalize the behavior only when the need for such arises, instead of prematurely generalizing, which can lead to code that is harder to maintain. That normally occur when we notice that we have two or more types satisfying a common contract, and it should make sense to formalize it, by factoring the specification out of this behavior to a common interface.

Furthermore, if we have a concrete type that has nothing more than methods defined by an interface, it might make sense to not export the
concrete type itself, but rather the interface. By doing this, we're able to change the internal representation of the concrete type without breaking external clients, as they don't know anything about the type, except that it happens to implement the exported interface. That's one of the main selling points for encapsulation:

> The ability to change the internal structure of a module without breaking its clients.

# Conclusion

Although different from other languages (C++, Java, C#, etc.), Go has its own set of features and opnions on how to do OOP, like user-defined types, methods, inheritance, and run-time polymorphism. And how Go provides such tools may improve our skills as developers, as we have to learn other approaches to write software, and incorporte these in our toolboxes.

Moreover, Go offers interesting constructions like slices, maps, channels, etc, which might improve both the correctness and expressiveness of our
code.

# References

[1] Alan A. A. Donovan, Brian W. Kernighan. "The Go Programming Language".

[2] Effective Go. <https://golang.org/doc/effective_go.html>

[3] <https://golang.org/>

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
