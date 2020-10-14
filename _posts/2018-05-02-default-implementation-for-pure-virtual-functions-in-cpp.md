---
layout: "post"
title:  "Default Implementation for Pure Virtual Functions in C++"
tags:   c++
---

> C++ allows default implementation for pure virtual member-functions.

* * *


# Interface Inheritance

Programming languages like Java and C# have explicit support for the keyword _interface_ to express interfaces that
must be implemented by concrete types that want to be part of the interface.

By doing so, one gains some flexibility provided by runtime polymorphism with dynamic dispatching, _vtables_, and the like.

In C++, you can achieve pretty much the same effect by using pure virtual member-functions (kind of abstract functions), which are member-functions that must be implemented by every concrete derived class that implements the "interface".

It specifies a set of behaviours and where we don't care about representation.

The syntax to declare an abstract member-function is to append the suffix `= 0_` to the virtual member-function declaration, like so:

```cpp
struct shape {
    virtual double area() const = 0; // Shapes and areas... :-)
}
```

Declares the abstract member-function `area` of the "interface" `shape`.

# Default Implementation

An abstract member-function says that every concrete class that implements the
the interface must provide an implementation for every abstract member
function.

It doesn't say though that the interface can't provide a default
implementation. Regardless of having a default implementation in the interface, the derived class still needs to provide
an implementation.

For example, the default implementation offers part of the behaviour that must be completed by the derived class.

# Example

Suppose that you're modelling a game system for an epic adventure, and your game has a variety of weapons (swords, arrows, etc),
which the hero makes use to save the world from its doom.

You've decided to create an interface `Weapon` as an abstract concept in your game. This interface offers
the abstract member-function `attack` that needs to be implemented by every
concrete weapon in the game. `Weapon` itself doesn't enough information to be concrete, but it may be reasonable to
offer a default behaviour for the `attack` that the concrete classes could use.

## Helper member-function

The first option is to have a second member-function, say `defaultAttack` (naming is hard, isn't it?)
that can be called by the concrete `Weapon`s, but given that this member-function has the sole purpose of being used by derived
classes, `defaultAttack` only needs `protected`:

<script src="https://gist.github.com/rvarago/102275897b79f56bf101e4bfc9bd2c88.js"></script>

Here, the `Sword` must provide an implementation for `Weapon::attack`. And `Sword::attack` is implemented in terms of
`Weapon::defaultAttack`.

## Pure Virtual member-function Implementation

As I've said, **a pure virtual member-function must be implemented by
the concrete derived class**. That is, it doesn't say that an abstract base class can't provide a default implementation for its pure virtual member-functions, though:

<script src="https://gist.github.com/rvarago/49e7ffc74203540827c2925c29450f06.js"></script>

In this approach, we can't define the abstract member-function inside the class declaration, it has to be defined out of it.

# Conclusion

We've discussed a bit about a possible meaning of pure virtual member-functions and the option of providing default implementations so that derived classes can use.

In my experience, this is not a well-known feature and it might be confusing at times. Thus, one might need to consider whether it makes sense to use, perhaps providing a member-function with a different name in the base class is clearer and therefore preferred sometimes.

As it usually happens in programming, the fact that you can doesn't necessarily mean that you should.

An important fact about pure virtual member-functions is that for a pure
virtual destructor, you're **required** to provide a default implementation, and you do it by following the syntax of the example, where we defined a pure virtual member-function outside of the class declaration.

# Acknowledgements

I would like to thank **Simon St James** for kindly reviewing this text.

# References

[1] Meyers, Scott. Effective C++.

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*