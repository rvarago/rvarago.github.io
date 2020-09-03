---
layout:	"post"
title:	"Introduction to Google Guice for DI"
---

Guice is a Java framework for automation of DI, so it can leverage your code
flexibility to evolution by minimizing coupling between modules. In this
article, you'll learn how to get started with it.

* * *

Hello,

Today, I'm going to talk about how to get started with Google Guice for
dependency injection in Java. Firstly, I'll introduce some concepts about the
framework, then we'll write a simple application to exemplify.

#### Dependency Injection

As we've talked on [this post](https://medium.com/@varago.rafael/managing-
coupling-with-dependency-injection-46157eb1dc4d), dependency injection (DI) is
a technique of providing dependencies for clients instead of having the latter
explicitly obtaining them. DI is basically a way to achieve a more general
goal named dependency inversion principle (DIP), which states:

> Instead of depending on implementations, prefer to depend on abstractions.

When applying DI, we need a way to inject (wire, bind) the dependencies for
the clients that are requesting them, we could do it manually when
instantiating the client class or we could rely on a framework to automate
this tasks and also add some interesting functionalities, like life-cycle
management.

Regarding Java, there are a variety of frameworks with possibly pros and cons,
for example, [Weld](http://weld.cdi-spec.org/), [Spring](https://spring.io/),
[Guice](https://github.com/google/guice) etc.

In this post, we'll be using Guice and in a future post, I intend to talk
about Spring, probably in the context of designing RESTful APIs with Spring
Boot and Spring REST.

#### Google Guice

Google Guice is a framework to automate dependency injection by providing a
container to where we can map abstractions and implementations. After the
mapping, the dependencies will be automatic injected in clients when
requested.

The mapping in Guice is achieved by the implementation of
_com.google.inject.Module_ which is normally done by inheriting from the
abstract base class _com.google.inject.AbstractModule_.

After, we need to override the _configure_ method and rely on a fluent API by
calling _bind_ and _to_ methods to define the mapping between the abstraction
(parameter of _bind_ ) and implementation (parameter of _to_ ).

Then, we can inject the dependencies by annotating your dependencies with
_com.google.inject.Inject._

Finally, we need to obtain a _com.google.inject.Injector_ from our previously
defined module, so we are now able to retrieve the client with the
_getInstance_ method and its dependencies will be automatically injected.

#### Example

This example consists of a part of a Java system that sends log information
about its operation. To simplify the inclusion of Guice in our project, we're
going to use Maven as a build tool.

In the _pom.xml_ , add the following artifact for Guice at version 4.0 in the
dependencies section:

> <dependency>  
>  <groupId>com.google.inject</groupId>  
>  <artifactId>guice</artifactId>  
>  <version>4.0</version>  
> </dependency>

Let's create the interface _LogSender_ to represent the behavior:  "sends the
log to some medium":

This service will be used by _Exchanger_ class that has a reference to
_LogSender_ and the injection will be done by its constructor annotated with
@Inject:

The implementation _StdoutLogSender_ will send the log to the standard output
stream, in this case the console:

Now, we need to tell Guice how to map _LogSender_ to _StdoutLogSender_ , and
we do it by the _LoggingModule_ class:

Finally, in the main class _Application_ , we can create an _Injector_ and
pass our _LoggingModule_ to its constructor. Then, we are able to get an
instance of _Exchanger_ which will have its dependency on _LogSender_ bound:

#### Conclusion

In this article, we've discussed the basics regarding how to get started with
Google Guice for automation of dependency injection tasks in Java application.
We wrote a simple application to exemplify how to define the mapping between
abstractions and implementations and how to inject them in our clients.

I hope that with this basic overview of Guice you can continue your studies
about dependency injection and how to develop code with a lower level of
coupling and high level of cohesion.

#### References

[1] <https://medium.com/@varago.rafael/managing-coupling-with-dependency-
injection-46157eb1dc4d>

[2] <https://www.martinfowler.com/articles/injection.html>

[3] <https://github.com/google/guice>

