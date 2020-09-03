---
layout:	"post"
title:	"Managing Coupling with Dependency Injection"
---

Today, I'm going to talk about the importance of managing the level of
coupling between modules and how to achieve it with dependency injection.
We'll review some basic concepts regarding the…

* * *

![](/assets/img/2018-05-13-managing-coupling-with-dependency-injection.jpeg)

Hello,

Today, I'm going to talk about the importance of managing the level of
coupling between modules and how to achieve it with dependency injection. The
goal is to design more flexible code that is able to change with minimum side
effects.

#### Coupling

An application can be structured by a collection of modules where each module
has a responsibility and also exposes a public Application Programming
Interface (API) for interaction with other modules.

Suppose that we have a module A that needs some functionality from module B,
for example, A invokes a method of B to complete some task.

We say that **module B is a dependency of module A** , or in other words
**module A is a client of module B**. Thus, we need a way to bind a module
with its dependencies.

When we have such dependency structure, we are coupling the modules and it
complicates their evolutions because we'll need to be careful to don't break a
module by changing one or more of its dependencies.

We want to be able to evolve a module with the guarantee of keeping the whole
system working correctly. Therefore, we need to **manage the level of coupling
between modules** to keep it as lower as possible.

#### Interface vs Implementation

There are some available techniques for managing the level of coupling, but
one of the basic ideas is to follow the Dependency Inversion Principle (DIP)
that says:

> Avoid depending on concrete concepts, instead, make your modules **depend on
abstractions**.

We've stated that a module has two dimensions: a (single) responsibility and a
public API. Basically, we're saying that a module has an implementation of its
purpose and a well-defined and structured interface (public API) to able
clients to communicate with its features.

The implementation is directly related to the specific algorithms and data
structures employed to achieve a defined goal. They can change and probably
will, for example, you can realize that a hash table is more suitable than a
tree to save and retrieve a particular kind of data, or a sorting algorithm is
more efficient than another, or you may want to apply some optimization
technique like proxy objects or early computation etc.

Therefore, you want to be as free as possible to evolve the implementation of
your module without break its clients.

On the other hand, the interface is the way the clients communicate with your
module. By Scott Meyers " _it should be simple to use correctly and hard to
use incorrectly "_, for instance, it should be:

  * Robust
  * Flexible
  * Stable
  * Easy
  * Testable

The general idea is that the interface represents a contract that clients of
your module must agree to be able to use your module. But this is a two-way
street, your clients have to agree to obey your contract, but you **must**
ensure that once they agree with the contract you 'll change neither its
**syntax** nor its **semantics**. If you do, you 'll be breaking the contract
and thus, your clients!

Once you have a contract (interface), that your clients are depending on, you
are free to change the internal implementation and also, you are freeing your
clients to change one implementation for another if both implementations agree
with the interface.

#### Dependency Injection

Dependency injection (DI) is a technique heavily based on the general idea of
setting the modules to depend on abstractions stated by DIP.

In DI, you don't instantiate the concrete dependencies inside the client
module, instead, you get them injected by binding the abstractions that you
have declared with the correct implementations.

For example, suppose that you want to log some data, so you can have a service
named _Logger_ , but you may want to have multiple ways to log (console, file
etc), so you make _Logger_ an abstraction and it has multiples implementations
( _ConsoleLogger_ , _FileLogger_ etc). Thus, every client that want to log
data will declare a dependency for _Logger_ , and the dependency will be
provided to the client when requested by some "piece of code" outside the
client.

Normally, the binding or wiring of dependencies is achieved by the employment
of a **container** (examples for Java: Spring, Guice etc). A container is
basically a mapping between interface and implementations. Basically, it can
be made by metadata or configuration files (Spring 's way) or by coding the
mapping directly in the source code (Guice's way). But you can benefit from DI
even if you don't use any framework, just with the bare concepts. Let's go to
an example!

Suppose that you have the class _Client_ that uses a service of the class
_Computation_ that represents an intensive computation, for example, a
factorial. The first attempt to model this requirement can be:

 **Inside** the code of class _Client_ , we've explicitly **bound** its
**concrete** dependency for _FactorialComputation_. Therefore, we 've made a
high coupling between _Client_ and _FactorialComputation._

Now, suppose that, **by profiling** our **** application, we've noted that for
some contexts, the variability of _n_ is very low, so we want to apply the
optimization technique of caching the computations outcomes to use them
lately. But you may want to use this technique only in some instances of your
running application and the need to change _Client_ is not the best option,
because not all the clients may want to use this optimization.

The problem here is: we **coupled the client with the concrete
implementation** , so we're unable to change the latter without break the
former.

The best solution is to remove the high coupling dependency between the
modules. So, we make the _Client_ just say that it needs a service for
computing the factorial, but it really doesn 't care about how the computation
may be done, **it doesn 't need to know about implementation details of its
dependency**.

Applying DI, _Client_ just need a reference for an interface
_FactorialComputation_ and the binding between it and its concrete
implementation will be provided by an external code, in this case by the
_main_ function, but it could be also a container.

Here is the code that achieves the same behaviour but with a lower level of
coupling, therefore with more flexibility to accommodate changes:

Now, we've separated the interface from implementation.

The abstract class _FactorialComputation_ defines an interface with a pure
virtual member function with [default
behaviour](https://medium.com/@varago.rafael/default-implementation-for-pure-
virtual-functions-in-c-3c525cc820af) and a virtual destructor. This interface
has two implementations, the first is _SimpleFactorialComputation_ and it
computes the factorial each time requested and the second is
_CachedFactorialComputation_ and it uses a hash map to cache the computed
results.

The dependency for the factorial computation inside _Client_ is now just a
smart pointer for the interface that must be externally injected with the
concrete implementation by the constructor, in this case, is by the
_std::make_unique <CachedFactorialComputation>_ inside the _main_ function

With this solution, you can also inject your own tailored implementation and
if you follow the contract established by _FactorialComputation_ , the
_Client_ will still work with your version and won 't be even required to be
recompiled.

Also, if we want to test some feature of _Client_ in isolation of its
dependencies (unit test) we can apply mocks to fake the behavior of
_FactorialComputation_ to supply a fake version of it.

#### Conclusion

In this article, we've discussed the basics of module coupling and DI as a
manner to decrease the level of coupling. Then, we discussed an example of how
to achieve DI in C++.

By applying DI you've seen that you're free to change implementations as long
as you don't change the interface without break clients.

I hope that with this article you can be able to advance in your studies about
how to design systems that are correct, robust and flexible.

In a future post, I intend to talk more about DI and how to achieve it with
the assistance of frameworks for container management.

#### References

[1] <https://martinfowler.com/articles/injection.html>

[2] Effective C++. Scott Meyers.


***
*Originally published at https://medium.com/@rvarago*
