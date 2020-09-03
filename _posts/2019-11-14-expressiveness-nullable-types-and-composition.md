---
layout:	"post"
title:	"Expressiveness, Nullable Types, and Composition"
---

absent is a small library meant to simplify the composition of nullable types
in a generic, type-safe, and declarative style for some C++ type constructors.

* * *

> An introduction to the C++ library
[absent](https://github.com/rvarago/absent).

I have created a small C++ library called absent:
<https://github.com/rvarago/absent> inspired by functional programming
languages, e.g. Haskell and Scala, whose purpose is to simplify the functional
composition of nullable types, such as, but not limited to,
[std::optional](https://en.cppreference.com/w/cpp/utility/optional).

It offers some useful combinators, for instance, _bind_ , _fmap_ , _eval_ ,
alongside an infix notation based on operator overloading, that aim to reduce
boilerplate from our daily C++ code while increasing its type-safety and
expressiveness.

One of its main purposes is to refactor this kind of code snippet:

    
    
    auto const maybe_person = find_person();  
    if (!maybe_person) return;  
      
    auto const maybe_address =1 find_address(*maybe_person);  
    if (!maybe_address) return;  
      
    auto const zip_code = zip_code(*maybe_address);

To a declarative pipeline of composed functions that pushes the checking
against emptiness to the very end of the chain, avoiding repetitive error
handling, and therefore reducing the chances of accidental invalid accesses,
like this:

    
    
    auto const maybe_zip_code = find_person()  
                                 >> find_address | zip_code;  
    if (!maybe_zip_code) return

The following links lead to the series of two guest posts that I wrote for
Jonathan Boccara's __[_Fluent C++_](https://www.fluentcpp.com/) __ that
describes the motivation behind absent, related projects, and how to make use
of it.

  * [Expressiveness, Nullable Types, and Composition (Part 1)](https://www.fluentcpp.com/2019/07/16/expressiveness-nullable-types-and-composition-part-1/)
  * [Expressiveness, Nullable Types, and Composition (Part 2)](https://www.fluentcpp.com/2019/07/19/expressiveness-nullable-types-and-composition-part-2/)


***
*Originally published at https://medium.com/@rvarago*
