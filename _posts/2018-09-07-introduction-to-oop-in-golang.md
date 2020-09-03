---
layout:	"post"
title:	"Introduction to OOP in Golang"
---

* * *

> Learn how to get started with Go for OOP.

Today, I'm going to talk about the basics of Object Oriented Programming (OOP)
constructions written in Golang or simply Go.

![](/assets/img/2018-09-07-introduction-to-oop-in-golang_0.png)

An "object oriented" Gopher. Source: <https://github.com/egonelbre/gophers>

#### User-Defined Types

Go supports the notion of user-defined types, that is, the composition of
native types (like int, string, etc.) to build more complex types reflecting
the concepts of the real world like person, car, bank account, etc. or
abstract concepts like requests, processes, etc.

The way to express user-defined types in Go is by means of the _struct_
keyword. For example, to define a type named _Employee,_ that models an
employee in a management system with attributes: _id_ , _name_ , and _salary_
, we can achieve it by:

Different from other languages like Java, Go's _struct_ can only contain state
(attributes), thus you can 't directly write methods (behavior) inside the
_struct_ , but you can rely upon a special syntax for functions that has a
special argument called a receiver argument.

For example, to add a method called _raiseSalary_ that receives a _bonus_
amount and mutates an instance of type _Employee,_ and a method _format_ that
accesses an instance of type _Employee_ , you can write:

Note that the receiver name can be any valid identifier in Go, I just chose
_self_ at my own convenience (Python feelings). Also, note that for
_raiseSalary_ , the receiver is a **pointer** to _Employee_ , that's because
it needs to mutate the receiver's attributes.

#### Encapsulation

The notion of encapsulation is expressed in Go by **packages** that are, up to
some extent, similar to Java 's packages and C++'s namespaces. But in contrast
to Java and C++, Go doesn't have access modifiers ( _private_ , _public_ ,
etc.), so it differentiates between private and public symbols by the case of
its first letter, where:

  * Uppercase: _public_
  * Lowercase: _private_

A private symbol is only accessible inside its own package, meanwhile, a
public symbol has visibility both inside and outside its package.

The package name is declared at the start of each file and must be the same of
its directory name in the file system. You can also have multiple files
composing a single package.

An example of a package:

Also, a package has a well-defined initialization ordering, that follows:

  1. Imported packages
  2. Package's level variables
  3.  _init()_ functions

#### Composition and Inheritance

Go doesn't support inheritance or the _is-a_ relationship. Instead, it
encourages us to apply composition over inheritance and by doing so,
preventing some classic problems that can arise with inheritance hierarchies
erroneously implemented.

One of the facilities provided is the possibility to embedding nameless
_structs_ (and also _interfaces_ , see below) inside other _structs_ (or
interfaces), and by this, the embedding _struct_ (or interface) has direct
access to the embedded _struct_ (or interface) by a technique called
promotion.

For example, you may like to have a type _Manager_ that is-a _Employee_ , you
can write this:

Note that you can call methods of _Employee_ with an instance of _Manager_ ,
but you can't call a function expecting an _Employee_ with an instance of
_Manager_ , so there aren't any implicit conversion from _Manager_ to
_Employee_ (even if we 're dealing with pointers), so there isn't an is-a
relationship here.

#### Interface and Polymorphism

However, with interfaces, you can obtain some level of inheritance-like
behavior.

An interface establishes a contract that every type must agree (sign) in order
to implement the interface. In Go, if a type implements all the methods
defined by the interface, this type is said to adhere to the interface
contract and can be used where an interface is expected.

For instance, an _Employee_ type has a role, so we can make it implement the
contract established by the _interface_ _Role_ :

When _printRoleName_ is called, the type _Employee_ is implicitly converted to
the _interface_ _Role_ that it satisfies. Thus, you 're achieving polymorphic
behavior because the call is dispatched on run time through the examination of
the actual type and the invocation of the type's right method.

One of the most interesting aspects of interfaces in Go is that there's no
need that a type explicitly states that it implements an interface, it just
needs to define its methods, and by doing this it's implicitly implementing
the interface. A nice consequence is that we can create the interface
afterwards the type has been created, so we can generalize the behavior once
necessary (instead of premature generalization, that can lead to code
difficult to maintain), which normally occur when we note that we have two or
more types satisfying a common contract, and it should make sense to
generalize it by extracting the specification of this behavior to a common
interface.

Moreover, in some scenarios, it can be convenient to combine two
characteristics of Go.

For instance, if you have a concrete type that has nothing more than the
methods defined in an interface, it should make sense to not export the
concrete type, instead export only the interface. By doing this, you're able
to change the internal representation of the concrete type to a more suitable
one and don't affect its users, because they don't know anything about it,
they just care about the exported interface. This is one of the most appealing
properties of encapsulation:

> The ability to change the internal structure of a module without breaking
its users.

#### Conclusion

Although different from other languages (C++, Java, C#, etc.) Go has nice
features to do OOP, like user-defined types, methods, inheritance, and
polymorphism. And the manner that Go provides other facilities can improve our
skills as developers, for example, by reasoning about other approaches to do
system modeling.

Moreover, Go has other great native constructions like slices, maps, and
channels that can improve both the correctness and expressiveness of your
code. It's also possible to use type assertions to inspect the dynamic type of
objects that implements an interface, which is specially useful for empty
interfaces.

I really encourage you to search for more information about Go and verify
whether it can help you and your company in your daily programming tasks by
providing, possibly, alternative ways to solve problems.

#### References

[1] Alan A. A. Donovan, Brian W. Kernighan. "The Go Programming Language".

[2] Effective Go. <https://golang.org/doc/effective_go.html>

[3] <https://golang.org/>

