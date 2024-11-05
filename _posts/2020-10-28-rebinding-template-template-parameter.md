---
layout: "post"
title:  "Rebinding Template Template Parameters"
tags:   c++ meta-programming
---

> Rebinding a template template parameter in C++ might be useful at times, or at least fun to do.

* * *

|![Types](/assets/img/rebinding-template-template-parameters.png)|
|:--:|
| *Rebinding a `T<A>` into a `T<B>`. [sketchpad.io](https://sketch.io/sketchpad)*|

While upsetting my compiler and having some fun with template metaprogramming in C++, I stumbled upon the need of rebinding the inner type `A` of a template template parameter `T<A>` into a different type `B`, resulting in `T<B>`.

That is not particularly complicated and probably not very interesting. Still, it is a fun exercise and might be useful once in a while when writing generic code. Thus I have decided to code up a small type trait and share it.

**Challenge:**

> Given a template parameter `T` matching some type such as `T<A>` I want a `T<B>`.

Let's start off writing compile-time tests that shall demonstrate the API that we want to end up with and later check its implementation:

```cpp
static_assert(std::is_same_v<rebind<std::vector<int>>::value_type, int>);
static_assert(std::is_same_v<rebind<std::vector<int>>::to<float>, std::vector<float>>);
```

We have given the name `rebind` to our type trait.

Given a template template parameter `T<A>`, we have that `rebind<T<A>>` should expose two member aliases: `value_type` and `to`.

We access the inner type `A` as `value_type` and rebind the template template parameter to another inner type `B` like `T<B>` as `to<B>`.

# Implementation

With an idea on what the API of `rebind` will look like, we are ready to implement it:

```cpp
template <typename T>
struct rebind;

template <template<typename> typename T, typename A>
struct rebind<T<A>> {
    using value_type = A;

    template <typename B>
    using to = T<B>;
};
```

As usual, we pattern match on templates.

We have a primary template of `rebind` which we left empty, and a partial specialization for the case where we have a type matching `T<A>`.

In the partial specialization, we destructure the parameters and store them in the member aliases `value_type` and `to`. Since `to` rebinds the inner type `A` into another type `B`, it needs to accept `B` as a template parameter as well.

If we run our tests, they should pass.

As a convenience we can add an alias `rebind_to` that should ease usage, mainly when dealing with dependent names:

```cpp
template <typename T, typename B>
using rebind_to = typename rebind<T>::to<B>;
```

Then, we can write `rebind_to<std::vector<int>, float>>` as opposed to `rebind<std::vector<int>>::to<float>`, which is a bit shorter.

# Improving Error Messages

Our implementation relies on the "catch-all" primary template, which we should not instantiate.

However, when we attempt to use `rebind` with an invalid type, the compiler will then try to instantiate the primary template, which we had not defined; therefore we will likely get a rather unclear compilation-error message:

```bash
error: incomplete type 'rebind<int>' used in nested name specifier
    static_assert(std::is_same_v<rebind<int>::value_type, int>);
```

We may improve it by statically asserting that our primary template is never used, and if it does, we then display a more descriptive message:

```cpp
template <typename T>
struct rebind {
    static_assert(deny<T>, "T must match T<A>");
};
```

Where `deny` is a variable template whose sole purpose is to delay the evaluation of the `static_assert` to the point when we *do* attempt to, mistakenly, instantiate the primary template:

```cpp
template <typename...>
inline constexpr auto deny = false;
```

Now, we should see the following error message whenever we pass an invalid parameter:

```bash
error: static assertion failed: T must match T<A>
    static_assert(deny<T>, "T must match T<A>")
```

# Final Code

```cpp
template <typename...>
inline constexpr auto deny = false;

template <typename T>
struct rebind {
    static_assert(deny<T>, "T must match T<A>");
};

template <template<typename> typename T, typename A>
struct rebind<T<A>> {
    using value_type = A;

    template <typename B>
    using to = T<B>;
};

template <typename T, typename B>
using rebind_to = typename rebind<T>::to<B>;
```

# Implementing `transform` for `std::optional<A>`-like types with `rebind`

An example where we *may* want to use `rebind` is to implement a generic `transform` for `std::optional<A>`-like types.

`transform` allows us to map over a type such as `std::optional<A>` with a function `A → B` to produce an `std::optional<B>`, or return an empty `std::optional<B>` if the input `std::optional<A>` is empty.

However, we want to extend it to support other types that are "similar" to `std::optional<A>`, i.e. all types that model the same optional-like concept.

> **Disclaimer:** A C++20 concept for optional-like/nullable would probably fit the bill **far** better.

We may implement a simplified (omitting forwarding references, etc) version of `transform` as:

```cpp
template <typename OptionalA, typename UnaryFunction,
    typename OptionalB = rebind_to<OptionalA, decltype(std::invoke(std::declval<UnaryFunction>(), *std::declval<OptionalA>()))>>
[[nodiscard]] constexpr auto transform(OptionalA opt, UnaryFunction fn) -> OptionalB {
    if (!opt) {
        return OptionalB{}; // empty optional in, then empty optional out.
    }
    else {
        return OptionalB{std::invoke(fn, *opt)}; // apply `fn` to the value inside `opt` and wrap it in a new optional.
    }
}
```

The signature might look scary, especially the last template parameter `OptionalB`, which is arguably an abuse of default parameters. That is only meant to have the name `OptionalB` available in both return type and body, and thus avoid repeating the same expression twice.

Fundamentally, `OptionalA` has the type `T<A>` and we de-reference it with `*` to access the inner `A` which we feed into `UnaryFunction` of type `A -> B` to obtain a `B` that we finally lift into the expected return type `OptionalB` of type `T<B>`.

We might use `transform` as:

```cpp
int main(int, char*[]) {
    std::optional<double> const in_opt{1.5};
    std::optional<int> const out_opt = transform(in_opt, [](double const x) {return static_cast<int>(x) + 2;}); // std::optional<int>{3}
    return out_opt.value();
}
```

The assembly instructions generated from the [Compiler Explorer](https://godbolt.org/) when compiling with *x86-64 gcc 10.2* and *-std=c++17 -O1 -Wall -Wextra -Werror*:

```assembly
main:
        mov     eax, 3
        ret
_GLOBAL__sub_I_main:
        sub     rsp, 8
        mov     edi, OFFSET FLAT:_ZStL8__ioinit
        call    std::ios_base::Init::Init() [complete object constructor]
        mov     edx, OFFSET FLAT:__dso_handle
        mov     esi, OFFSET FLAT:_ZStL8__ioinit
        mov     edi, OFFSET FLAT:_ZNSt8ios_base4InitD1Ev
        call    __cxa_atexit
        add     rsp, 8
        ret
```

That means that the compiler managed to evaluate the whole expression at compile-time, yielding the value *3*.

> From my perspective, a [member function](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0798r3.html) in `std::optional<T>` or, perhaps preferably, language support for something like [extension methods](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4474.pdf) would lead to a much nicer syntax (`in_opt.transform([](auto const x) {return std::to_string(x + 1);})`) and cleaner chaining (`in_opt.transform(to_this).transform(to_that)`).

## Conclusion

In this purposefully short post, we have seen how to write a type trait `rebind` to rebind a template template parameter `T<A>` to different inner type `B` resulting in a new type `T<B>`, which might be useful when writing generic code.

For simplicity, we have limited ourselves to types with single parameters (e.g. `T<A>`). However, we could extend `rebind` to work with variadic templates (e.g. `T<A, As...>` and store `As...` in an `std::tuple<As...>`) without much hassle.

## References

[1] [C++ reference: Template parameters and template arguments](https://en.cppreference.com/w/cpp/language/template_parameters).

[2] [Expressiveness, Nullable Types, and Composition]({{ site.baseurl }}{% link _posts/2019-11-14-expressiveness-nullable-types-and-composition.md %}).
