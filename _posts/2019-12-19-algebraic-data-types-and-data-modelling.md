---
layout:	"post"
title:	"Algebraic Data Types and Data Modelling"
---

Programming is about composition, we usually tackle a problem by breaking it
up into smaller and more easily comprehensible tasks that are then composed
together, and types play an important role.

* * *

![](/assets/img/2019-12-19-algebraic-data-types-and-data-modelling_0.png)

That's the question! How about a composition of both? 😉

> Algebraic Data Types bring us yet another interesting way to express
concepts in code, let's see how they could help.

Programming is about composition. We usually tackle a hard problem by breaking
it up into smaller and more easily comprehensible tasks, that are then
composed together, be it functions, objects, etc.

Along the way, interesting challenges arise, when we are introducing new types
to express a concept in our code, be it concrete or abstract. And that's
precisely the topic that we are going to discuss a little bit. But before we
start, please notice that there are quite a few options available and I am not
going to argue which one is better than the other. Instead, I would like to
demonstrate how to use [Algebraic Data Types](https://code.egym.de/a-brief-
introduction-to-the-algebra-of-types-df92f0820e5) (ADTs) in a statically-typed
programming language to accomplish one specific task, and then encourage you
to read more about it, try it out, know its pros and cons, and based on that
make your own decisions. I shall be using C++ and Haskell, however, the ideas
can be applied to other programming languages as well.

### Types are cool, let's introduce some

Consider the following C++ type, meant to express the outcome of a read
operation over a communication channel:

    
    
    template <typename T>  
    struct ReadResult {  
      enum class State {  
        SUCCESS, FAILURE  
      };  
      State state;  
      T payload;  
      std::string error_code;  
    };

The questions that pop up are: What is the meaning of a _payload_ when the
_state_ equals _Failure_? Or equivalently, what is the meaning of an
_error_code_ when _state_ equals to _Success_? Maybe there cannot even exist a
default constructor for _T._

I would say, that those should be invalid scenarios. However, as far as our
types are concerned, we are allowing those scenarios.

Unless we build an encapsulation layer on top of this type to enforce its
validity, one could easily access the _payload_ on a _FAILURE_ state. This
would break internals, as the type itself does not guarantee to always be in a
valid state.

### Make it optional, or shouldn't I?

One way to act upon the issue is by re-writing the type to lift members that
are _per-state-nullable_ into pointers and use _nullptr_ to express emptiness,
or alternatively lift members into [optional types](https://code.egym.de
/object-instantiation-immutability-and-expression-oriented-style-
e847fead14ed):

    
    
    template <typename T>  
     struct ReadResult {  
      enum class State {  
        SUCCESS, FAILURE  
      };  
      State state;  
      std::optional<T> payload;  
      std::optional<std::string> error_code;  
    };

Now we can represent "empty " types by using null options, i.e.
_std::nullopt_. The code compiles fine even for a type _T_ , that does not
provide a default constructor.

However, I would argue that it simply patches the issue up, rather than fixing
it, because we are still able to represent invalid scenarios, i.e. accessing
_payload_ when _state_ equals _FAILURE_.

I believe we can do better. We might want to trigger a compilation error when
the user attempts to do such invalid operations as opposed to an error that
only shows up during runtime, when it might be too late.

### It's either this or that

If we carefully inspect _ReadResult <T>_, we will see that _state_ is not as
tightly coupled with _payload_ and _error_code_ as it could be, the possible
states and their respective values should be strongly tied together in such a
way that each possible state only exposes the members that make sense to it.

From an ADT's perspective. The fundamental problem is that the type
_ReadResult <T>_ can be both _T_ ( _payload_ ) **and** _std::string_ (
_error_code_ ) **at the same time** , in other words, it's a [**product**
type.](https://code.egym.de/a-brief-introduction-to-the-algebra-of-types-
df92f0820e5) Therefore the set of possible values that can inhabit a
_ReadResult <T>_ is the cartesian product of the set of values of its members:

> #ReadResult<T> = #T * #std::string * 2

The trailing 2 is due to the possible values of _State_ , which is either
_SUCCESS_ **or** _FAILURE_ , and hence # _State_ = 2.

To express the idea of "letting each state have only the members that make
sense to it", we can use a **sum** type:

    
    
    template <typename T>  
     struct Success {  
      T payload;  
    };
    
    
    struct Failure {  
      std::string error_code;  
    };
    
    
    template <typename T>  
    using ReadResult = std::variant<Failure, Success<T>>;

By using _std:: variant <Failure, Success<T>>_, _ReadResult <T> _has become a
type alias.

You can interpret _ReadResult <T>_ as "either _a Failure_ **or** _a Success
<T>_, but **not both at the same time** ". _Failure_ and _Success <T>_ are now
a **closed set** **of** **alternatives** for the variant _ReadResult <T>
_type.

Now each value of _state_ encapsulates only the types it cares about and we
have achieved a much stronger association between possible states and their
attributes.

Our _ReadResult <T>_ is a sum type with the following set of possible values:

> #ReadResult<T> = #Success<T> \+ #Failure = #T + #std::string

Moreover, the fact that _T_ may not have a default constructor only matters
when we are on the _Success_ alternative, but not on the _Failure_ case.
Nevertheless, we can still lift the _payload_ into an _std::optional <T>_ if
we have to.

As a side-note, here's the equivalent in Haskell, which has native support for
writing ADTs:

    
    
    data ReadResult a = Failure String | Success a

It reads as follows " _ReadResult_ is **either** a _Failure_ of _String_
**or** a _Success_ of _a_ ", where the bar | means _or_ and _a_ is a type-
parameter, i.e. a place-holder for a type, playing the same role as the
template type parameter _T_ has done in the C++ version.

This is a fairly common pattern in Haskell and the standard library provides
us with a convenient type:ata Either a b = Left a | Right b

Instead of _Failure_ and _Success_ , _Either_ refers to its value constructors
as _Left_ and _Right_ , respectively.

Thus we can re-write _ReadResult_ as the following type alias:

    
    
    type ReadResult a = Either String a

* * *

### Conclusion

The idea of _combining_ product and sum types is fairly powerful and can
unlock new design possibilities that are interesting to have in our toolbox.
They are particularly suitable when designing state machines, for instance,
game engines. This is the case when states are allowed to share some
attributes (product), but some attributes only make sense for specific states
(sum).

Furthermore, there are other, albeit maybe more exotic, kinds of ADTs built on
top of products and sums; for instance, PI types. Types that depend on values,
can help us to define powerful invariants, evaluated at compile-time. I would
recommend to check them out.

ADTs are useful tools to keep in mind, and quite often can be used to solve
real-world problems and help us write more expressive and correct code by
binding possible states and their values together. By allowing the system to
work on our behalf, we can prevent our code from representing invalid states,
even before it gets executed.

And now it's up to you, my fellow developer, to decide what tool fits better
each requirement that you happen to be working on. Know your alternatives (no
pun intended) and choose wisely 😺.

### References

[[1] [Categories for the Working
Hacker.](https://www.google.com/url?sa=t&source=web&rct=j&url=https://m.youtube.com/watch%3Fv%3Dgui_SE8rJUM&ved=2ahUKEwi71OHTrrPmAhUjxqYKHYTaDkAQwqsBMAB6BAgKEAQ&usg=AOvVaw0PbvucHrQmaDmT6v0wGD-z)

[[2] A brief introduction to the Algebra of Types](https://code.egym.de/a
-brief-introduction-to-the-algebra-of-types-df92f0820e5)

[[3] Expressiveness, Nullable Types, and
Composition](https://medium.com/@varago.rafael/expressiveness-nullable-types-
and-composition-949285bd6d1)


***
*Originally published at https://medium.com/@rvarago*
