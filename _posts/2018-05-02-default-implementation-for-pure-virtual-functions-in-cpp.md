---
layout:	"post"
title:	"Default Implementation for Pure Virtual Functions in C++"
---

* * *

Hello,

Today, I'm going to talk about an interesting and underused feature of C++ for
Object Oriented Programming (OOP), the possibility to provide a default
behavior for abstract member functions.

### Interface Inheritance

Some languages, like Java and C#, explicitly support the keyword _interface_
to express an user-defined type where all methods don 't have implementations,
just signatures. Every concrete, that is non-leaf, class that implements the
interface must agree with the established contract and provides an
implementation for all the inherited methods.

In C++, you can achieve the interface inheritance concept by the use of pure
virtual member functions, or abstract member functions, which are member
functions that must be implemented by every concrete class that inherits from
the "interface class".

The syntax to declare an abstract member function is to append the suffix: _=
0_ to the virtual member function declaration, for example:

 _virtual std::string speak() const = 0_

Declares a constant abstract member function named _speak_ that takes no
arguments and returns _std::string_.

### Default Implementation

An abstract member function says that every concrete class that implements the
interface must provide an implementation for every inherited abstract member
function in order to compile.

But, it doesn't say that the interface class can't provide a default
implementation, it can and sometimes it makes some sense. But independently of
the default implementation, all concrete subclass must provide an
implementation.

For example, the default implementation can offer part of the behavior and
must be completed by the inherited class to have a full meaning.

### Example

Suppose that you're modelling a game system for an epic adventure, and your
game has a variety of weapons (swords, arrows etc) that the hero uses to save
the world from the evil.

You've decided to create an interface _Weapon_ that models the abstract
concept that must be the base for your weapons system. This interface offers
the abstract member function _attack_ and it needs to be completed by every
concrete weapon in the game. The _Weapon_ concept doesn 't have a concrete
meaning, but perhaps it's reasonable to have a default behavior for the
_attack_ that the concrete classes may use.

### Helper Member Function

The first manner to achieve the goal is to have a second member function, for
example, _defaultAttack_ that can be called by the concrete classes, but given
that this member function has the sole purpose of being used by derived
classes, _defaultAttack_ should be made protected.

Thus, you can have:

Here, the _Sword_ class must provide an implementation for _Weapon::attack_ in
order to be concrete and _Sword::attack_ calls _Weapon::defaultAttack_ to use
the default behavior provided by it. The drawback of this approach is that you
need to have a member function that only exists to support another member
function semantics and you may want to express the default behavior of
_attack_ in the _attack_ itself.

### Pure Virtual Member Function Implementation

As I've said, **a pure virtual member function must always be implemented by
every derived class intended to be concrete**. But, it doesn 't mean that you
can't provide a default implementation for it in the abstract base class.

You can and sometimes you may wish do it and this is how:

In this approach you can't define the member function inside the class
declaration, otherwise you will get a compilation error. So, you must define
your implementation outside the class declaration.

The implementation of the derived concrete class continues to be mandatory,
but it can call the base class implementation.

### Conclusion

In this article, we've discussed the meaning of pure virtual member function
(abstract member function) and the approaches to providing a default
implementation for this kind of member function that derived class can use
inside its proper implementation.

An important fact about pure virtual member functions is that for a pure
virtual destructor, you're **required** to provide a default implementation,
and you do it by following the syntax of the the example, where we've defined
our pure virtual member function outside the class declaration.

### Acknowledgements

I would like to thank **Simon St James** for kindly revised this text.

### References

[1] Meyers, Scott. Effective C++.


***
*Originally published at https://medium.com/@rvarago*
