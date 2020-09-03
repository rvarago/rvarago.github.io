---
layout:	"post"
title:	"Groovy - Some Cool Features"
---

* * *

Hello,

In this article, I'm going to talk about some interesting and useful features
of the Groovy programming language and provide examples of its usage.

### Groovy

Groovy [1] is a dynamic language that runs inside the Java Virtual Machine
(JVM). Groovy is very similar to Java, but has a lot of particularities which
give to the developer awesome tools to solve daily programming problems in a
simple and elegant way.

### Environment

First at all, you need to install Groovy. You can download it at <http://www
.groovy-lang.org/> or use your package manager. On my machine, I'm running
Groovy at version 2.4.5.

After installed and configured, open your terminal and type:

    
    
    groovyConsole

This will open the Groovy Console, where you can write commands and see the
results one after other. Now, you are ready to start explore some Groovy
features.

### Description

This example declares a _List_ of _Strings_ , iterates over it and prints a
greeting text in the console. And it introduces new syntaxes of Groovy.

Firstly, Groovy is an optionally typed language, which means that we aren't
forced to declare the type for our variables. Rather, we can simply declare
variables with the _def_ keyword and the type will be automaticaly resolved at
runtime based on the value assumed by the variable.

Secondly, Groovy has an embedded syntax to declare lists, _listOfNames_ is a
_List_ , not an array, of _Strings_. Another interesting point of this example
is the enhanced for loop, in which we don 't declare any type to our loop
variable.

Thirdly, Groovy introduces a very simple way to print data, the _println_
function is a shortcut to print in the system standard output and put a new
line at the end (in Java, it'll be _System.out.println_ ).

Fourthly, the text printed has a special meaning too, it uses the new
_GString_ , which is an "evolved" _String_ , that gives us the ability to
insert formatted values inside the _String_. In the example,  " _Hello ${name}
"_, $ _{n_ ame _}_ solves __ for the loop variable _name_.

### Range, closure and each

This example computes the sum of numbers between 1 and 5, including the end
points.

The above code introduces new structures of the Groovy programming language
which allows us to write code inspired in the Functional Programming (FP)
paradigm.

Firstly, the range notation where _(a..b)_ returns a _List_ of numbers between
_a_ and _b_. We iterate over this _List_ in a different way, using a concept
of FP named closure. In simple words, a closure is a block of code which has
access to the surrounding scope. Here, _each_ is a method that accepts a
closure with one argument which is the loop variable obtained from the range,
when we don 't declare the variable name, Groovy injects a default named _it_.

I have mentioned that, inside a closure, we have access to the surrounding
scope. In the example, we used this resource by modifying the _sum_ variable
declared outside the _each_ block scope.

### Reduce and inject

In FP, we have a very important operation called reduce, which is defined as a
function applied on a collection of values returning only one value. The sum
that we have done above is an example of reduction.

We'd started with a _List_ , operated on it and returned just one
representative value, which is the sum of the _List_ elements. Groovy has its
own syntax to perform reduction, that uses inject, which follows the syntax

### Filter and findAll

This example introduces one my favourites resources of Groovy: filter.

As the name suggests, a filter is a function applied in collections, that
removes all entries in the collection which doesn't satisfy the predicate
passed to it.

In Groovy, the filter operation is represented by the _findAll_ method, which
accepts a predicate that can be built by a closure. In the example, we 've
built a closure which tests whether the element is an even number, returning
true is this case.

### Class, named parameter, map, collect and dot operator

Groovy supports Object Oriented Programming (OOP). To do so, it's classes as
structural elements for representing models. Nonetheless, Groovy has a
relevant difference, it introduces default accessors and mutators (getters and
setters) for every attribute, which implies that attributes are, in some way,
public by default.

Another difference compared with Java, is that Groovy has named parameters,
which enables us to change the order that we access function arguments,
passing the argument name with the value, we've used it in constructor syntax,
indicating what attributes we're initializing.

As well as for _Lists_ , Groovy has the embedded syntax _def myMap = [key:
value]_ for _Map_ declaration and initialization.

Together with reduce, the other important operation of the functional
programming is map. Map is simply an association. Given a collection of
values, a map operation returns another collection of values, whose each
returned value is a function of the original value.

In Groovy, we do map with the _collect_ method, which accepts a closure for
each entry and returns another value based on the corresponding entry. Here,
we have used it to build a _List_ of ages from a _Map_ of _Person_.

Although its simplicity, the above example can be improved using the dot
operator (*.), replacing the _collect_ by _println mapOfPeople*.value.age_

Where for every entry in _mapOfPeople_ , we access the _value_ element, then
we get the age attribute of it. It 's like we're accessing each element
individually, but the same operation is "spread over" the entire _Map_.

### Final Example

The final example summarizes the learned content.

It resembles a common task in daily programming. Where we have some data, we
filter it, access a determined property of the data and apply a function on
theses properties.

Here, we are summing the amount of sales of houses, where this data was
retrieved from a _Map_ of _Sales_. You may be wondering why we used == instead
of _equals_? It 's because Groovy overrides operators, where for _Strings_ ,
== is done by the _equals_ method under the hoods.

### Conclusion

In this article, we've learned about the Groovy programming language, we
mentioned few features of it, but there are many others and you're encouraged
to follow the reference at [1] to learn more.

### Bibliography

[1] Groovy. <http://www.groovy-lang.org/>


***
*Originally published at https://medium.com/@rvarago*
