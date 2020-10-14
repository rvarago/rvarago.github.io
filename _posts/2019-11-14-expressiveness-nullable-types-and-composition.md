---
layout: "post"
title:  "Expressiveness, Nullable Types, and Composition"
tags:   adt absent fp c++
---

> _absent_ is a tiny open-source C++ library meant to simplify the composition of nullable types in a generic, type-safe, and declarative style.

* * *

I have started a tiny open-source C++ library called [_absent_](https://github.com/rvarago/absent) inspired by functional programming
languages, e.g. Haskell and Scala, whose purpose is to simplify the functional composition of nullable types, such as, but not limited to,
[std::optional<T>](https://en.cppreference.com/w/cpp/utility/optional).

_absent_ offers some useful combinators to make the composition of nullable types more expressive, e.g. `and_then`, `transform`, `eval`. Furthermore, it also supports infix notations based on operator overloading that aim to reduce boilerplate when chaining operations on nullable-types while increasing type-safety and expressiveness.

As an example, consider the following snippet:

```cpp
auto const person_opt = find_person();  
if (!person_opt) return;  
  
auto const address_opt = find_address(*person_opt);  
if (!address_opt) return;  
  
auto const zip_code = get_zip_code(*address_opt);
```

We can use _absent_ to refactor this piece of code into a declarative pipeline of functions and push the checks against emptiness
to the very end of the chain, avoiding repetitive and entangled error handling:

```cpp    
auto const zip_code_opt = find_person()  
                            >> find_address
                            | get_zip_code;  
if (!zip_code_opt) return
```

I briefly mentioned _absent_ before in the context of [total functions]({{ site.baseurl }}{% link _posts/2019-05-26-the-beauty-of-total-functions.md %}).

Additionally, the following links lead to the series of two guest posts that I wrote at Jonathan Boccara's [Fluent C++](https://www.fluentcpp.com/), where I described the motivation behind _absent_ and how we may profit from it:

  * [Expressiveness, Nullable Types, and Composition (Part 1)](https://www.fluentcpp.com/2019/07/16/expressiveness-nullable-types-and-composition-part-1/).
  * [Expressiveness, Nullable Types, and Composition (Part 2)](https://www.fluentcpp.com/2019/07/19/expressiveness-nullable-types-and-composition-part-2/).

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
