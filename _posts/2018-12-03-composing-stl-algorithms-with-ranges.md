---
layout:	"post"
title:	"Composing STL algorithms with ranges"
---

STL is a fantastic tool that improves our C++ code, making it more expressive,
clean, and less error-prone. However, STL algorithms don't compose well. But
fortunately, this is about to change.

* * *

> STL is a fantastic tool that improves our C++ code, making it more
expressive, clearer, and less error-prone. However, the function composition
of STL algorithms can be a bit verbose. Fortunately, this is about to change.
Why? Ranges are coming!

![](/assets/img/2018-12-03-composing-stl-algorithms-with-ranges_0.png)

"C++ Icon". Source: <https://github.com/isocpp/logos>

I've been using C++ since 2010, just after my first _Hello World_ program (by
the way, a led blinking written in Assembly for 8 bits PIC16f micro-
controller, it 's been a while…). Since then, C++ and its standard library
have been evolving, becoming an easier and more pleasant language to learn,
teach, and use IMO. Further, it has done so without sacrificing its close to
the metal performance and zero-overhead philosophy:

> You don't pay for what you don't use.

There are many improvements that have been added into the language to make it
easier to write better code.

Let's recap some of them by looking at the three last three standards as of
today:

C++11:

  * lambdas
  * auto type deduction
  * braced initialization
  * constexpr
  * move semantics
  * nullptr
  * std::begin/std::end

C++14:

  * decltype(auto)
  * generic lambdas
  * relaxed conditions for constexpr

C++17:

  * if constexpr
  * nested namespaces
  * selection statements with an initializer
  * structured binding
  * fold expressions
  * std::optional
  * std::variant

Now, we're moving to the next standard: C++20! And with it, new features that,
potentially, will make our programmer lives even better might show up. Among
them: ranges that will [likely make it, at least partially, into
C++20](https://herbsutter.com/category/c/).

For the illustration that follows, I will be using C++17 and [Eric Niebler's
range-v3](https://github.com/ericniebler/range-v3) library, from which ranges
are based on.

### Function composition and STL

Before talking about ranges, let's look at one of the issues that it solves by
the means of an example.

Given a sequence of users with fields _id_ and _age_ , we'd like to obtain the
sequence of IDs for all users which have more than 18 years.

 **First attempt: Using a range-based for loop**

It works. However, we needed to write a loop that imperatively (do this and
then do that, etc) mandates what the machine should do. Besides being simple,
we need to mentally parse the loop to understand what it's going on. Moreover,
it arguably violates the Single Responsibility Principle (SRP) as the same
code is responsible for:

  1. Filtering the sequence of users
  2. Map the filtered sequence of users to a sequence of IDs

We could extract those two responsibilities into two functions and call them
in a proper order.

Actually, STL already provides those functions for us, and that's neat!

 **Second attempt: Using STL algorithms**

We extracted the filtering and mapping operations into two function.

Let's hold on and review what we achieved so far from a Functional Programming
(FP) perspective.

In FP, there are two important higher-order functions (functions that receive
and/or returns other functions):

>  **Filter:** given a sequence of values, returns a new sequence whose
elements meet a certain criterion specified by a predicate.

>  **Map:** given a sequence of values, returns a new sequence whose elements
are the result of applying a mapping function to each element.

An interesting observation of the definitions above is that the filter and map
operations return a new sequence, rather than mutating the input.

Another key element of FP, and programming in general, is notion of **function
composition** :

> Given a variable x: A, and two functions f: A -> B and g: B -> C, we compose
f and g, i.e., (g∘ f)(x): A -> C as g[f(x)].

This cool property of composition gives us the possibility to chain functions,
where the target of  _f_  matches the source of  _g_.

Going back to our example, say that _f_ is the filter and _g_ is the mapping,
to map a filtered sequence, we need:

> map(filter(sequence))

Comparing that result with our second attempt, we can spot that:

  1.  _copy_if_ acts like our filter
  2.  _transform_ acts like our map

But we can also detect some discrepancies:

  1. It's cluttered with iterators ( _begin_ / _end_ ) that are too technical for the code's level of abstraction
  2. The functions don't return the new sequences. Instead, they mutate the sequence passed as parameters through the output iterator returned by _back_inserter_

So it isn't possible to easily compose the expressions as they are, because
the STL algorithms used don't return the container, which would allow us to
chain the operations.

> I am not saying that composing STL algorithms is impossible, of course it
is, see  _stable_sort_ for an  example. Moreover, iterators are an amazing
tool, which glues algorithms and containers, and it's one the main components
that make the STL such a powerful software library. The algorithms library is
a remarkably well-designed set of abstractions.

> I just want to point it out that composing STL algorithms as function-
expressions is a bit inconvenient and rather verbose.

What we'd like in this case is an algorithm with the following semantic:
_transform_if_.

OK, let's write our own possibly implementation.

 **Third attempt: Writing our transform_if**

It works! Now, we can compose calls, and we got rid of iterators inside the
API, constraining them inside the implementation. So, no more IN/OUT
parameters or in-place mutations. Now we have:

  *  **Input** : _only_ through parameters
  *  **Output** : _only_ through return

We didn't mutate the input container by writing to its iterator. Instead, we
return a new container that can be chained with functions.

>  **Note about performance:**

> Yes, we're returning a new container by-value. But it might be totally fine
in C++17 due to modern C++ features: move-semantics and copy-elision, which
hopefully eliminate the potentially expensive copy. In case of doubt, it's
advisable to measure.

But it has some drawbacs as well:

  1. A significant amount of glue code that can distract us from the problem to be solved
  2. It doesn't scale well. It's easy to come up with new use cases, where we need to compose other algorithms (filter, map, reduce, etc). Thus leading to the repetition of this kind of code

### Ranges to the rescue

Commonly, we want to traverse the whole container, that is, from its first
element to its last element, rather than part of it. This was assumed by
_transform_if_.

Thus, instead of receiving iterators that specify exactly where to start and
stop the traversal, we received a container and then, buried the _begin()_ and
_end()_ calls inside the implementation.

Therefore, we raised the level of abstraction of our client code, erasing the
iterators from the API and passing containers directly:

> algorithm(begin(c), end(c)) => algorithm(c)

That is roughly the essence of a range: something that can be traversed. Or,
in a loosely spoken C++, a concept that offers the _begin_ and _end_
functions:

    
    
    Range {  
      Iterator begin()  
      Iterator end()  
     }

So, yes, you're right!

> STL containers are themselves ranges.

So far, it simply moved the iterators from the API into the implemention, but
it goes far beyond.

By working directly with ranges, instead of iterators, we enabled the chaining
of operations that we wanted. The range-v3 library already provides overloads
to the STL algorithms with ranges for us, delegating to the proper iterator-
based versions internally.

### Iterator adapters

The second aspect of working with ranges is the possibility to enriching an
iterator by mixing a behavior, turning it into an **iterator adapter**.

Firstly, let's recap:

> An iterator is, roughly speaking, a mechanism that let us move along a
container and access its elements, without knowing how the container was
implemented.

Essentially, it's an elegant bridge that plugs containers to algorithms,
without neither needing to know implementation details of the other. That's
information hiding in its essence.

Based on this, an iterator adapter is built on top of an iterator. The adapter
customizes the behaviour provided by the iterator by combining it with a
function.

For instance, a _transform_iterator_ is an iterator combined with an unary
function _transformer_ , such that, for each element of the container indexed
by the iterator, it applies _transformer_ to the de-referenced value before
returning it.

Can you guess where we're going?

How about combining iterator adapters with ranges? That brings us to **range
adapters**.

### Range adapters

> A range adapter is basically an object that combines a range with an
iterator adapter to produce another range.

Moreover, by associating ranges with adapters, the result is itself a range
that can be adapted again. Therefore, composability comes easily by elegantly
chaining range adapters with the operator pipe _|_.

 **View adapters** are a subset of the range adapters, and as their name
suggests, they create a lightweight view over the underlying range, and they
do not mutate the referred range.

View adapters have the following properties:

  1. Lazy adaptation of the underlying range
  2. Don't mutate the underlying range
  3. Cheap to create and copy
  4. Non-owning reference semantics

There are plenty of view adapters. And for our example we are particularly
interested in the following:

  *  _view::filter_ : Given a range and a unary predicate, returns a new range based on the original range whose elements satisfy the predicate
  *  _view::transform_ : Given a range and a unary function, returns a new range resulting from appliying the function to each element of the original range

That should solve our original problem: How to compose the filter and map
operations to achieve the _transform_if_ semantics. Let 's solve it with
range-v3's range adapters.

 **Fourth and final attempt: Using range adapters**

Concise and clean! Furthermore, it's easier to extend and create more complex
adapters by composing simple and orthogonal building blocks.

That isn't all. The range-v3 library provides overloads for the STL
algorithms, actions, possibility to extend it with our own ranges, etc. It
definitively worth to check it out.

### Conclusion

C++ and its standard library (particularly the STL) are fantastic programming
tools. Furthermore, as the years go by, the C++ community has been working
hard to make them even better.

That spirit should stay strong in C++20. And one of the features that might
help us in our daily programming tasks is the introduction of ranges, which
extends the STL algorithms making them easier to write code based on function
composition.

Therefore, we won't miss non-existing algorithms, such as _transform_if_
anymore, as we can straightforwardly compose the filter and mapping
operations. And that among many other operations

### References

[1] Eric Niebler. Range-v3. <https://ericniebler.github.io/range-v3/>

[2]Ranges library (C++20). <https://en.cppreference.com/w/cpp/ranges>


***
*Originally published at https://medium.com/@rvarago*
