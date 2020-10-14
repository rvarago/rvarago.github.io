---
layout: "post"
title:  "Managing Coupling with Dependency Injection"
tags:   c++
---

> Managing dependencies between modules is a critical part of software development. The goal is to design flexible programs that are able to change without incurring into high-costs. 

* * *

# Coupling

An application may be structured as a collection of modules, where each module
has a responsibility, and also exposes a public Application Programming
Interface (API) that other modules can consume.

Suppose that we have a module `A` that needs a functionality provided by another module `B`, e.g, `A` invokes a method of `B`.

We say that **module B is a dependency of module A** or **module A is a client of module B**. 

When this occurs, we need some way to assemble such a dependency graph, by binding a module with its dependencies.

By having such a chain of dependencies, we introduce coupling between the modules and that
complicates their evolution. This is because we need to be careful so that we don't break a
module by changing its dependencies in a backwards-incompatible way.

We want to be able to evolve a module with the guarantee of keeping the whole
system working correctly. Therefore, we need to **manage the level of coupling
between modules** to keep it as lower as possible, but not lower.

## Interface vs Implementation

There are some techniques for managing the level of coupling. One is to follow the Dependency Inversion Principle (DIP), which says:

> Avoid depending on concrete concepts, instead, modules should **depend on abstractions**.

We've stated that a module has two dimensions: its responsibility and its public API.
Basically, we're saying that a module has implementation-details and an interface so that clients can use it.

The implementation is tied to specific algorithms and data structures employed to achieve the module's goal efficiently.
They may change and probably will. For example:

1. A hash table might have properties that fit better in a problem
2. A different sorting algorithm could be chosen.
3. You need to optimize a hot-loop.
4. You simply need to refactor a piece of code to keep tech-debt under control.
  
Therefore, you usually want the freedom to evolve the implementation of your module without breaking its clients.

On the other hand, the interface is the way the clients communicate with your module. According to Scott Meyers: 

> It should be simple to use correctly and hard to use incorrectly

For instance, it should be:

  * Robust.
  * Flexible.
  * Stable.
  * Easy.
  * Testable.

The general idea is that an interface represents a contract that clients of
your module must abide by so that they can use your module.

But that is a two-way path: your clients have to obey your contract, but you also have to ensure that once they have agreed with the contract you will not change it, otherwise, you run into the risk of breaking your clients!

Once you have established an interface that your clients are depending on, you
are free to change the internal implementation. Additionally, your clients may very well swap an implementation by another as long as both implement the same interface.

## Dependency Injection

Dependency injection (DI) is a technique based on the general idea of
setting the modules to depend on abstractions stated by the DIP.

In DI, clients don't instantiate their concrete dependencies. Instead, dependencies are injected into clients by binding the abstractions that they have declared.

For example, suppose that you want to log some stuff, so you have a service `Logger`. But you may want to have multiple ways to log (console, file
etc), hence you turn `Logger` into an abstraction that has multiple implementations (`ConsoleLogger`, `FileLogger`, etc). Thus, every client that wants to log declares a dependency on `Logger`, and such dependency will be provided to the client when requested by some "piece of code" outside the client itself.

Although not mandatory, it's fairly common to bind (or wire) dependencies by employing a **container**, like Java's Spring and Guice. 

A container is basically a mapping between interfaces and implementations, a registry. It can
be made by metadata or configuration files or by coding the mapping directly in the source code. But you can benefit from DI
even if you don't use any framework at all, but rather sticking to the bare concepts. Let's go to an example!

### Example

Suppose that you have a class `Client`, which uses a service provided by the class `Computation` that represents an intensive computation, e.g., a factorial.

The first attempt to model this requirement might be:

<script src="https://gist.github.com/rvarago/b53973fff718bc1412f31ffcb05507b4.js"></script>

**Inside** the body of `Client`, we've explicitly **bound** its
**concrete** dependency on `FactorialComputation`, and therefore we've ended up with a
high-level of coupling between `Client` and `FactorialComputation`.

After **profiling** our application, we've noticed that in some contexts, the variability of `n` is quite low, so that we could benefit from memoizing the result of computations and reuse them later.

However, you may not want to use this technique in all the places of your application. This means that changing `Client` is not the best alternative, because not all clients want to use the optimization.

So we find ourselves with the following problem: we **coupled the client with the concrete implementation of its dependency**. We cannot change the latter without breaking the former.

We've figured out that we should reduce the level of coupling between our modules.

We could change `Client` so that it just declares that it needs a way to compute factorials, but it doesn't really care about how the computation is performed. That is, it doesn't need to know about the implementation details of its dependency.

By applying DI, `Client` needs only some sort of reference to an interface `FactorialComputation`. The concrete dependency shall be provided by some external piece of code that we are not aware of yet. It could very well be done inside the `main` function or some other place.

Here is the code that achieves the same behaviour, but with a lower level of coupling, therefore with more flexibility to accommodate future changes:

<script src="https://gist.github.com/rvarago/684fde5fb197c04e9e22cb0e5f94739c.js"></script>

Now, we've separated the interface from its implementation.

The abstract class `FactorialComputation` defines an interface with a pure
virtual member function with [default
behaviour]({{ site.baseurl }}{% link _posts/2018-05-02-default-implementation-for-pure-virtual-functions-in-cpp.md %}), and a virtual destructor.

The interface has two implementations: `SimpleFactorialComputation` which computes the factorial each time it's requested, and the second is
`CachedFactorialComputation` which caches the result of previous computations.

The dependency on the factorial computation inside `Client` is now a reference, implemented by a smart pointer, to the interface that must be externally injected with the concrete implementation by the constructor, here it's done by calling
`std::make_unique <CachedFactorialComputation>` inside the `main` function

With that solution, you can also inject a tailored implementation and as long as it follows the contract established by `FactorialComputation`, the `Client` should still work.

On top of that, should we want to test a given feature of `Client` in isolation of its
dependencies, we could easily mock `FactorialComputation`.

# Conclusion

We've discussed the basics of software coupling and DI as an approach to decrease it. Then, we walked through a simple example of how
to do DI in C++.

By applying DI, we've seen that we're free to change implementations as long
as we don't change the interface.

# References

[1] [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html).

[2] Effective C++. Scott Meyers.

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
