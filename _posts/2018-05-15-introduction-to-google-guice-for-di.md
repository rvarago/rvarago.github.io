---
layout: "post"
title:  "Introduction to Google Guice for DI"
tags:   java
---

> Guice is a Java framework that aims to simplify the application of the Dependency Injection pattern to minimize coupling between modules.

* * *

# Dependency Injection

As we discussed in a previous [post]({{ site.baseurl }}{% link _posts/2018-05-13-managing-coupling-with-dependency-injection.md %}), Dependency Injection (DI) is a way of providing dependencies to clients, rather than having clients
explicitly bootstrapping their dependencies by themselves.

That's one way to achieve Dependency Inversion Principle (DIP), which states:

> Instead of depending on implementations, prefer to depend on abstractions.

When applying DI, we need a way to inject (a.k.a. wire, or bind) dependencies into clients that need them. It sure is possible to do it manually when instantiating the client class, or we could pull a framework to help us with the task while also bringing additional benefits, like life-cycle management.

Regarding Java, there is a variety of frameworks, each of those comes with pros and cons,
for example, [Weld](http://weld.cdi-spec.org/), [Spring](https://spring.io/), [Guice](https://github.com/google/guice), etc.

In this post, we'll be using Guice.

## Google Guice

Google Guice is a framework meant to reduce the boilerplate that comes with DI, it does so by providing a
container to where we can map abstractions into implementations. After having established this
mapping, the dependencies will be automatically injected into proper clients when requested.

The mapping in Guice is achieved by the implementation of `com.google.inject.Module`, which is normally done by inheriting from the
abstract base class `com.google.inject.AbstractModule`.

Afterwards, we need to override the `configure` method and rely on a fluent API by calling `bind` and `to` methods, which define the mapping between the abstraction (parameter of `bind`) and implementation (parameter of `to`).

Then, we can inject the dependencies by annotating your dependencies with `com.google.inject.Inject`.

Finally, we need to obtain a `com.google.inject.Injector` from our previously defined module, so we are now able to retrieve the client with the
`getInstance` method and its dependencies shall then be injected.

### Example

The example is part of a Java application that sends log information about its operation. To simplify the inclusion of Guice in our project, we're
going to use Maven as a build tool.

In the `pom.xml`, add the following artifact to get Guice at version 4.0 in the dependencies section:

```xml
<dependency>  
  <groupId>com.google.inject</groupId>  
  <artifactId>guice</artifactId>  
  <version>4.0</version>  
</dependency>
```

Let's then create the interface `LogSender` to represent the behaviour: "send the log to some medium":

<script src="https://gist.github.com/rvarago/711e9bf4aa280e6913b8a2eb30eba4a6.js"></script>

This service will be used by `Exchanger` class, which has a reference to `LogSender` and the injection will be done by its constructor annotated with `@Inject`:

<script src="https://gist.github.com/rvarago/c88882010dc3f46cedc7bf18d7986296.js"></script>

The implementation `StdoutLogSender` will simply send the log to the console:

<script src="https://gist.github.com/rvarago/93f3b976e9b7115221b807e82532039a.js"></script>

Now, we need to tell Guice how to bind `LogSender` to `StdoutLogSender`, and we will do it in the `LoggingModule`:

<script src="https://gist.github.com/rvarago/d5b9b7a8f72c69e58def4a3d3f5f43d7.js"></script>

Finally, in the main class `Application`, we will create an `Injector` and pass our `LoggingModule` to its constructor. Therefore, we will able to get an instance of `Exchanger`, which will have its dependency on `LogSender` bound:

<script src="https://gist.github.com/rvarago/6f857e79d894a74723039eb9e32e4c67.js"></script>

# Conclusion

We've discussed the basics of how to get started with Google Guice to automate DI tasks in Java applications. We wrote a simple application to exemplify how to define the mapping between abstractions and implementations and how to inject them in our clients.

# References

[1] [https://medium.com/@varago.rafael/managing-coupling-with-dependency-
injection-46157eb1dc4d](https://medium.com/@varago.rafael/managing-coupling-with-dependency-
injection-46157eb1dc4d).

[2] [https://www.martinfowler.com/articles/injection.html](https://www.martinfowler.com/articles/injection.html)

[3] [https://github.com/google/guice](https://github.com/google/guice)

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
