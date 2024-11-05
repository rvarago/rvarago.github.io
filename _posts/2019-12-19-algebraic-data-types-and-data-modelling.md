---
layout: "post"
title:  "Algebraic Data Types and Data Modelling"
tags:   type-system adt fp c++ haskell
---

> Programming is about composition. We usually tackle a problem by breaking it up into smaller and more manageable tasks, which are then composed together into the final solution. Types can play an important role in assisting us.
> 
> Algebraic Data Types bring us yet another interesting way to express concepts in code. Let's see how they can help us.

* * *

|![](/assets/img/2019-12-19-algebraic-data-types-and-data-modelling_0.png)|
|:--:| 
| *That's the question! How about the composition of both? 😉*|

Programming is about composition.

We usually tackle a problem by breaking it up into smaller and more manageable tasks (functions, objects, modules, etc), which are then composed together into the final solution.

Along the way, interesting challenges arise when we are introducing new types to express a concept in our code, be it concrete or abstract.

That's precisely the topic that we are going to discuss a little bit. But before we start, please notice that there are quite a few options available and I am not going to argue which one is better than the others. Rather, I would like to demonstrate how to use [Algebraic Data Types]({{ site.baseurl }}{% link _posts/2019-03-22-a-brief-introduction-to-the-algebra-of-types.md %}) (ADTs) in a statically-typed programming language to accomplish one specific task. Then encourage you to read more about it, try it out, know its pros and cons, and based on that make your own decisions.

I shall be using C++ and Haskell, however, the ideas can be applied to other programming languages as well.

## Types are Cool, Let's Introduce Some Types

Consider the following C++ type, meant to express the outcome of a read operation over a communication channel:

```cpp
template <typename T>  
struct ReadResult {  
  enum class State {  
    Success, Failure  
  };  
  State state;  
  T payload;  
  std::string error_code;  
};
```

The questions that pop up are:

1. What is the meaning of a `payload` when `state` equals to `Failure`?
2. Equivalently, what is the meaning of an `error_code` when `state` equals to `Success`? Maybe there should not even exist a default constructor for `T`.

I would say, that those should be invalid scenarios. However, as far as our types are concerned, we are allowing those scenarios, they are perfectly valid (i.e. the code compiles just fine).

Unless we build an encapsulation layer on top of this type to enforce its validity, one could easily access the `payload` on a `Failure` state. This would break internals and violate invariants, as the type itself does not guarantee to always be in a valid state.

## Make it Optional, or Shouldn't I?

One way to act upon the issue is by re-writing the type and lift its members that are _per-state-nullable_ into pointers and use `nullptr` to express emptiness.

Alternatively, and normally more robustly, we could lift such members into [optional types]({{ site.baseurl }}{% link _posts/2019-03-22-a-brief-introduction-to-the-algebra-of-types.md %}):

```cpp
template <typename T>  
  struct ReadResult {  
  enum class State {  
    Success, Failure  
  };  
  State state;  
  std::optional<T> payload;  
  std::optional<std::string> error_code;  
};
```

Now, we represent "empty" types with null options, i.e. `std::nullopt`. The code compiles just as fine even for a type `T` where a default constructor is not available.

However, I would argue that it simply patches the issue up (maybe even makes things worse), rather than properly fixing it. We are still able to represent invalid scenarios, i.e. we can access `payload` when `state` equals to `Failure`.

I firmly believe we can do much better.

We might want to trigger a compilation error when the user attempts to trigger such invalid operations as opposed to an error that only shows up during runtime when it might be too late (or worse, missed).

## It's Either This or That

If we carefully inspect `ReadResult<T>`, we shall see that `state` is not as tightly bound to `payload` and `error_code` as it could be. The possible states and their respective values should be strongly tied together, such that each state only exposes the members that make sense to it.

From an ADT's perspective, the fundamental issue is that the type `ReadResult<T>` can be both a `T` (`payload`) **and** `std::string` (`error_code`) **at the same time** . That is, `ReadResult<T>` is a [**product** type.]({{ site.baseurl }}{% link _posts/2019-03-22-a-brief-introduction-to-the-algebra-of-types.md %}). Therefore the set of possible values that can inhabit in a `ReadResult<T>` is the cartesian product of the set of possible values of its members:

> #ReadResult<T> = #T * #std::string * 2

The trailing _2_ is due to the two possible values of `State` (either `Success` **or** `Failure`), hence _#State = 2_.

To express the idea of "letting each state having only the members that make sense to it", we can use a **sum** type:

```cpp    
template <typename T>  
  struct Success {  
  T payload;  
};

struct Failure {  
  std::string error_code;  
};

template <typename T>  
using ReadResult = std::variant<Failure, Success<T>>;
```

By using an `std::variant<Failure, Success<T>>`, `ReadResult<T>` has become a type alias.

We can interpret `ReadResult<T>` as "either a `Failure` **or** a `Success<T>`, but **not both at the same time**". `Failure` and `Success<T>` form a **closed set** **of** **alternatives** for the variant `ReadResult<T>` type.

Now, each value of `state` encapsulates only the types it cares about. We have built a much stronger association between possible states and their attributes.

Our `ReadResult<T>` is a sum type with the following set of possible values:

> #ReadResult<T> = #Success<T> + #Failure = #T + #std::string

Moreover, the fact that `T` may not have a default constructor only matters when we are in the `Success` case, but not when in the `Failure` case. Nevertheless, we can still lift the `payload` into an `std::optional<T>` should we want to.

As a side-note, here's the equivalent in Haskell, which has native support for writing ADTs and I found super useful to explore and prototype on:

```haskell
data ReadResult a = Failure String | Success a
```

It reads as follows "`ReadResult` is **either** a `Failure` of `String` **or** a `Success` of `a`", where the bar `|` means _or_ and `a` is a type-parameter, i.e. a place-holder for a type, playing a similar role as the template type parameter `T` played in the C++ version.

This is a fairly common pattern in Haskell and the standard library provides us with a convenient type:

```haskell
data Either a b = Left a | Right b
```

Instead of `Failure` and `Success`, `Either a b` refers to its constructors as `Left a` and `Right b`, respectively.

Thus we can re-write `ReadResult` as the following type alias:

```haskell    
type ReadResult a = Either String a
```

## Conclusion

The idea of composing product and sum types is fairly powerful and can unlock new design possibilities that are interesting to have in our toolbox.

ADTs are particularly suitable when designing state machines, e.g. game engines, communication protocols, etc. That is the case when states are allowed to share some attributes (product), but some attributes only make sense in specific states (sum).

Furthermore, there are other, albeit more exotic, kinds of ADTs built on top of products and sums, for instance, PI types, where types can depend on values, which can help us to define powerful invariants that are verified at compile-time.

ADTs are useful tools to keep in mind, as they can be used to solve real-world problems and help us to write more expressive and correct code by binding possible states and their values together. By allowing the type-system to work on our behalf, we can banish illegal states even before our code gets executed.

Lastly, it's up to you, my fellow developer, to decide which tools fit better into each requirement that you happen to be working on. Know your alternatives and choose them wisely.

## References

[1] [Categories for the Working Hacker.](https://www.google.com/url?sa=t&source=web&rct=j&url=https://m.youtube.com/watch%3Fv%3Dgui_SE8rJUM&ved=2ahUKEwi71OHTrrPmAhUjxqYKHYTaDkAQwqsBMAB6BAgKEAQ&usg=AOvVaw0PbvucHrQmaDmT6v0wGD-z)

[2] [A Brief Introduction to the Algebra of Types.]({{ site.baseurl }}{% link _posts/2019-03-22-a-brief-introduction-to-the-algebra-of-types.md %})

[3] [Expressiveness, Nullable Types, and
Composition.]({{ site.baseurl }}{% link _posts/2019-11-14-expressiveness-nullable-types-and-composition.md %})

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
