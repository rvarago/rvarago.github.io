---
layout: "post"
title:  "Function Composition of STL Algorithms with Ranges"
tags:   stl fp c++
---

> STL is a fantastic tool that C++ provides us and using makes our code more expressiveness, cleaner, and correct. However, STL algorithms may sometimes involve a bit of boilerplate, especially when it comes to function composition, where we want to chain a sequence of operations. Fortunately, the Ranges library is about to make it better and simpler.

* * *

|![C++.](/assets/img/2018-12-03-composing-stl-algorithms-with-ranges_0.png)|
|:--:| 
| *C++. Source: <https://github.com/isocpp/logos>.*|

## Motivation

I've been using C++ pretty much since 2010, just after my first _Hello World_ program (a led blinking, written in Assembly for 8 bits PIC16f micro-controller, back in the days...). EVer since, C++ and its standard library have evolved, becoming a more pleasant language to learn, teach, and use. Further, it has done so without sacrificing its close to the metal performance and zero-overhead philosophy:

> You don't pay for what you don't use.

Let's briefly recap some of the additions by looking at the three last three standards as of today:

* C++11:
  * lambdas
  * auto type deduction
  * braced initialization
  * constexpr
  * move semantics
  * nullptr
  * std::begin/std::end

* C++14:
  * decltype(auto)
  * generic lambdas
  * relaxed conditions for constexpr

* C++17:
  * if constexpr
  * nested namespaces
  * selection statements with an initializer
  * structured binding
  * fold expressions
  * std::optional
  * std::variant

We're now moving to the next standard: C++20! And with it, new features that are coming, among them: Ranges. 

To illustrate, I will be using C++17 with [Eric Niebler's range-v3](https://github.com/ericniebler/range-v3) library, from which ranges are based on.

## Function Composition and STL

Before talking about ranges, we shall first look at a small example of what we can achieve with ranges.

> Given a sequence of `user`s wrapping `id` and `age`:

```cpp
struct user {
    long id;
    long age;
};
```

> Our goal is to obtain a sequence of `id`s for all `user`s whose `age` are greater than 18.

# First Attempt: Range-based for Loop

<script src="https://gist.github.com/rvarago/992e5f56df36053f4d1fe61f6df35086.js"></script>

It works as expected. However, we needed to write an imperative loop that dictates how the machine should do a series of computations. Albeit simple, we have to mentally parse the loop's body to understand what it's going on, what the pre-conditions and post-conditions are, etc. Moreover, it arguably violates the Single Responsibility Principle (SRP) as the same code is responsible for:

  1. Filtering the sequence of users.
  2. Transform such a filtered sequence of users into a sequence of IDs.

We could extract those two responsibilities into two separate functions and chain them.

The STL already provides algorithms to accomplish that, which is neat!

# Second Attempt: STL Algorithms

<script src="https://gist.github.com/rvarago/07323edd03ec01200b9679ef31b881ef.js"></script>

We've extracted the filtering (`std::copy_if`) and transformation (`std::transform`) steps into two functions to be applied sequentially.

Let's hold on that thought and review what we achieved from the perspective of Functional Programming (FP).

In the world of FP, there are two central higher-order functions (functions that receive and/or return other functions):

>  **Filter:** Given a sequence of values, returns a new sequence whose elements meet a certain criterion specified by a predicate.

>  **Map:** Given a sequence of values, returns a new sequence whose elements are the result of applying a mapping function to each element of the original sequence.

An interesting observation of the definitions above is that *filter* and *map* return a new sequence, rather than mutating the input in-place.

Another key element of FP, and programming in general, is the notion of **function composition**, whose mathematical definition looks similar to:

> Given a variable x: A_, and two functions _f: A -> B_ and _g: B -> C_, we compose _f and then g_, i.e., _(g ∘ f)(x): A -> C_ as _g[f(x)]_.

Composition gives us the ability to chain a series of functions, where the target of  _f_  must match the source of _g_.

Looking at our example, say that _f_ is the filter and _g_ is the mapping, then to map a filtered sequence:

> map(filter(sequence))

Comparing that result with our second attempt, we may spot:

  1.  `std::copy_if` acts as a filter.
  2.  `std::transform` acts as a map.

But we can also detect some differences:

  1. It's mixed with iterators (`std::begin` and `std::end`) that are powerful, yet technical constructions, which reside in a lower level of abstraction.
  2. The functions don't return the new sequences. Instead, they mutate the sequence passed as parameters through the output iterator returned by `std::back_inserter`.

Therefore, composing the expressions became a wee bit verbose, since the algorithms don't return containers, but rather use actions to mutate them in-place through iterators.

> I am **NOT** saying that composing STL algorithms is impossible, of course, it is, see  *stable_sort* for an example. Moreover, iterators are an amazing tool, which glues algorithms and containers in a flexible, consistent, and efficient way. Iterators are one of the major components that make the STL the powerful software library that we know and love. The algorithms library is a remarkably well-designed set of abstractions.

> I just want to point it out to the narrow issue where composing STL algorithms as a chain of expressions may be slightly inconvenient, and sometimes rather verbose.

What we'd like in this case is an algorithm with the following semantic: `transform_if`.

OK, let's roll our own implementation.

# Third Attempt: Rolling Our Own `transform_if`

<script src="https://gist.github.com/rvarago/d6229859f1ae20c563dd7c0c982cf60f.js"></script>

We made it! We composed the calls and got rid of iterators inside the API, constraining them inside the implementation. So, no more IN/OUT
parameters and in-place mutations. We now have:

  *  **Input** : _only_ through parameters.
  *  **Output** : _only_ through the return statement.

We didn't mutate the input container by writing to its iterator. We returned new containers that could then be sent to other functions within the chain.

Although we've achieved pretty much what we wanted, the solution comes with its drawbacks, namely:

  1. It requires some boilerplate, which might distract us from the task at hand.
  2. It doesn't scale very well. It's easy to come up with new use cases, where we need to compose other algorithms. Thus, leading to the repetition of the same kind of pattern.

# Ranges

Sometimes we want to traverse the whole container (from the first element to the last element), as opposed to part of it. Thus, instead of receiving iterators, which specify exactly where to start and stop the traversal, we can receive the whole container. and then call `begin()` and
`end()` from inside the implementation. That's less powerful than receiving iterators, yet convenient at times. Our implementation of `transform_if` relied on this special case. 

Therefore, we lost some power, but raised the level of abstraction of our client code, erasing the iterators from the API and passing containers directly:

> algorithm(begin(c), end(c)) => algorithm(c)

That is roughly the essence of a range: something that can be traversed. Or, in a loosely spoken C++, a concept that offers the `begin()` and `end()`:
      
    Range {  
      Iterator begin()  
      Iterator end()  
     }

Therefore, STL containers are themselves ranges.

So far, it simply moved the iterators from the API into the implementation, but it goes far beyond.

By working directly with ranges, as opposed to through iterators, we move closer to the chaining of operations that we wanted.

The range-v3 library already provides overloads to the STL algorithms with ranges for us, delegating to the proper iterator-based versions internally.

# Iterator Adapters

The second, and perhaps more interesting, aspect of working with ranges is the possibility of enriching them with some extra behaviour. We can such "iterator with behaviour" as **iterator adapter**.

A refresher on iterators:

> Roughly speaking, an iterator is a mechanism that let us move along a container and get access to its elements, without knowing details of how the container was implemented.

Essentially, it elegantly bridges the gap between containers and algorithms, without neither needing to know implementation details of the other. That's information hiding at our service.

On top of this, an iterator adapter is built on top of an iterator (no pun intended). An adapter customizes the behaviour provided by the iterator, and it does so by combining the iterator with a function.

For instance, a `transform_iterator` is an iterator enriched with a unary function `transformer`, such that, for each element of the container referenced by the iterator, it adapter applies `transformer` to the de-referenced element before actually returning it.

How about combining iterator adapters with ranges? That brings us to **range adapters**.

# Range Adapters

A range adapter combines a range with an iterator adapter to produce yet another range.

And by associating ranges with adapters, the result is itself a range, which can be further adapted again, and so on and so forth. 

The coolest part is that the adaption happens lazily. We fundamentally build a tree of expressions (filtering, transformation, etc) that are only evaluated when demanded (e.g. when creating a new vector from an adaptor), which allows single-pass traversals with fusion and some other interesting optimizations.

Putting that all together, we gain what is arguably a more ergonomic API without sacrificing efficiency. 

> At its core, the lazyness of range adaptors makes for a rather concise and functional composition of STL algorithms, which can be done quite elegantly by chaining range adapters with the "pipe operator": `|`.

# View Adapters

View adapters correspond to a subset of range adapters, and as the name suggests, they create a lightweight view over the underlying range. Notwithstanding, view adapters do not mutate the referred range in-place.

View adapters have the following properties:

  1. Lazy adaptation of the underlying range.
  2. Don't mutate the underlying range.
  3. Cheap to create and copy.
  4. Non-owning reference semantics.

There are plenty of view adapters. But, for our example, we are particularly interested in the following ones:

  *  `view::filter`: Given a range and a unary predicate, returns a new range based on the original range whose elements satisfy the predicate.
  *  `view::transform`: Given a range and a unary function, returns a new range resulting from applying the function to each element of the original range.

That should solve our original problem: How we can compose filter and map into `transform_if`. We shall solve it with range-v3's range adapters.

# Fourth and Final Attempt: Using Range Adapters

<script src="https://gist.github.com/rvarago/2b13bb46c2fe54d89994f76b323a52ba.js"></script>

Concise and reasonably clean (although lambdas have their share of verbosity, a topic for another day).

Furthermore, it's easy to extend the solution and create even more complex adapters by composing primitive and orthogonal building blocks.

That's not all that range-v3 has to offer. The library provides many more functionalities, such as overloads for the STL algorithms, actions, ability to write user-defined ranges, etc.

# Bonus: Actions and Transformations

If we step back and check the amazing [Elements of Programming](http://elementsofprogramming.com/) we might recall the duality between actions and transformations (generally, to only `std::transform`).

Actions communicate their result back to the caller by changing the state of the objects that were passed in:

```cpp
f(x) // f presumably mutates x.
```

Where transformations communicate by retuning new values:

```cpp
y = f(x) // f presumably returns a new y and keeps x unchanged.
```

Essentially, STL's `std::copy_if` and `std::transform` are actions. Their intended behaviour is achieved by manipulating iterators that are passed in and therefore the underlying containers.

Contrastingly, ranges favour transformations, where brand new containers are returned based on the other containers that are passed in.

No approach is strictly superior to the other, they are simply different. Both have pros and cons (efficiency, simplicity, etc), which vary depending on the exact context.

Moreover, there's an equivalence between actions and transformation that allows us to implement one in terms of the other:

```cpp
// action in terms of transformation.
void action(T& x) { x = transformation(x); }

// transformation in terms of action.
T transformation(T& x) { action(x); return x; }
```

I've found this result fascinating and we can exploit it to our best interest.

If we restrict ourselves to `std::vector`, then we can write thin-wrappers around `std::copy_if` and `std::transform` to turn these actions into the transformations `select` and `map`, respectively:

<script src="https://gist.github.com/rvarago/377fb86ca69df266cc160ae38490ca7c.js"></script>

## Conclusion

C++ and its standard library (STL in particular) are fantastic programming tools. Furthermore, as the years go by, the C++ community has been working pretty hard to make them even better.

That spirit stays strong as we get closer to C++20. Among its new features, one that might help us in our daily programming tasks is the introduction of ranges, which extends the STL algorithms making it easier to combine algorithms and function composition.

Therefore, we won't miss non-existing composite algorithms such as `transform_if` anymore. With ranges, we can straightforwardly compose filters, transformations, and many other operations.

Furthermore, sometimes we can just resort to the equivalence between actions and transformations to come up with the best solution to our problem.  

## References

[1] [Elements of Programming](http://elementsofprogramming.com/).

[2] [range-v3](https://ericniebler.github.io/range-v3/).

[3] [Ranges library (C++20)](https://en.cppreference.com/w/cpp/ranges).

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
