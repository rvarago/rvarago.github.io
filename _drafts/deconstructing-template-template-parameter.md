---
layout: "post"
title:  "Deconstructing Template Template Parameters"
tags:   c++ meta-programming
---

> Deconstructing a template template parameter in C++ might be useful at times, or at least fun to do.

* * *

|![Types](/assets/img/deconstructing-template-template-parameters.png)|
|:--:|
| *Deconstructing a Template Template Parameter into its Components. [sketchpad.io](https://sketch.io/sketchpad)*|

While upsetting my compiler and having some fun with template metaprogramming in C++, I stumbled upon the need of deconstructing a template template parameter `T<A>` into its components and rebinding its inner template parameter `A` into a different type `B`, resulting in `T<B>`.

That is not particularly interesting and probably not very fruitful. Still, it is a fun exercise and might be useful once in a while when writing generic code. Thus I have decided to code up a small type trait and share it.

**Challenge:**

> Given a template parameter `T` matching some type such as `Container<A>` I want a `Container<B>`.

Let's start off writing compile-time tests that shall demonstrate the API that we want to end up with and later check its implementation:

```cpp
static_assert(std::is_same_v<deconstruct<std::vector<int>>::value_type, int>);
static_assert(std::is_same_v<deconstruct<std::vector<int>>::rebind_to<float>, std::vector<float>>);
```

We have given the name `deconstruct` to our type trait.

Given a template template parameter `T<A>`, we have that `deconstruct<T<A>>` should expose two member-aliases: `value_type` and `rebind_to`.

We access the inner type `A` as `value_type` and rebind the template template parameter to another inner type `B` like `T<B>` as `rebind_to<B>`.

# Implementation

With an idea on what the API of `deconstruct` will look like, we are ready to implement it:

```cpp
template <typename T>
struct deconstruct;

template <template<typename> typename Container, typename A>
struct deconstruct<Container<A>> {
    using value_type = A;

    template <typename B>
    using rebind_to = Container<B>;
};
```

As usual, we pattern match on templates.

We have a primary template of `deconstruct` which we left empty, and a partial specialization for the case where we have a type matching `T<A>`.

In the partial specialization, we deconstruct the parameters and store them in the member-aliases. Since `rebind_to` rebinds the inner type `A` into another type `B`, it needs to accept `B` as a template parameter as well.

If we run our tests, they should pass.

As a convenience we can add an alias for `rebind_to` that should ease usage, mainly when dealing with dependent names:

```cpp
template <typename T, typename B>
using rebind_to = typename deconstruct<T>::rebind_to<B>;
```

Then, we can write `rebind_to<std::vector<int>, float>>` as opposed to `deconstruct<std::vector<int>>::rebind_to<float>`.

## Improving the Error Message

Our implementation relies on the "catch-all" primary template, which we should not instantiate.

However, when we attempt to use `deconstruct` with an invalid type, then the compiler will try to instantiate the primary template, which we had not defined and therefore the user will likely be present with a rather obscure compile-error:

```bash
error: incomplete type 'deconstruct<int>' used in nested name specifier
    static_assert(std::is_same_v<deconstruct<int>::value_type, int>);
```

We may improve it by statically asserting the primary template and displaying a more descriptive message:

```cpp
template <typename T>
struct deconstruct {
    static_assert(reject<T>, "T must be of form T<A>");
};
```

Where `reject` is a variable template whose sole purpose is to delay the evaluation of the `static_assert` to the point when we *do* attempt to, mistakenly, instantiate the primary template:

```cpp
template <typename...>
inline constexpr auto reject = false;
```

Now, we should see the following error message whenever we pass an invalid parameter:

```bash
error: static assertion failed: T must be of form T<A>
    static_assert(reject<T>, "T must be of form T<A>")
```

# Final Code

```cpp
template <typename...>
inline constexpr auto reject = false;

template <typename T>
struct deconstruct {
    static_assert(reject<T>, "T must be a template template parameter");
};

template <template<typename> typename Container, typename A>
struct deconstruct<Container<A>> {
    using value_type = A;

    template <typename B>
    using rebind_to = Container<B>;
};
```

# Implementing `transform` with `deconstruct`

An example where we *may* want to use `deconstruct` is to implement a generic `transform`.

`transform` allows us to map over a type such as `std::optional<A>` with a function `A → B` to produce an `std::optional<B>`. However, we want to extend it to support other types that are "similar" to `std::optional<A>`, i.e. all types that model the same concept.

> **Disclaimer:** A C++20 concept for optional-like/nullable would probably fit the bill **far** better.

We may implement `transform` as:

```cpp
template <typename OptionalA, typename UnaryFunction,
    typename OptionalB = typename deconstruct<OptionalA>::rebind_to<decltype(std::invoke(std::declval<UnaryFunction>(), *std::declval<OptionalA>()))>>
[[nodiscard]] constexpr auto transform(OptionalA opt, UnaryFunction fn) -> OptionalB {
    if (!opt) {
        return OptionalB{};
    } else {
        return OptionalB{std::invoke(std::move(fn), *std::move(opt))};
    }
}
```

The signature looks scary, especially the last template parameter `OptionalB`, which is possibly an abuse of default parameters. That is only meant to have the type `OptionalB` available at both return type and body and therefore avoid repeating the same expression twice.

Fundamentally, `OptionalA` has the type `T<A>` and we can de-reference it with `*` to access the inner `A` which we feed into `UnaryFunction` of type `A -> B` to obtain a `B` that we finally lift into the expected return type `OptionalB` of type `T<B>`.

We might use `transform` as:

```cpp
std::optional<int> const in_opt{1};
std::optional<std::string> const out_opt = transform(in_opt, [](auto const x) {return std::to_string(x + 1);}); // std::optional<std::string>{"2"}
```

> IMO, a member-function (e.g. `transform`, `map`) in `std::optional<T>` or, perhaps preferably, language support for extension methods would lead to a much nicer syntax (`in_opt.transform([](auto const x) {return std::to_string(x + 1);})`), and easy of chaining (`in_opt.transform(to_this).transform(to_that)`).

## Conclusion

In this purposefully short post, we have seen how to rebind a template template parameter `T<A>` to different type `T<B>`, which might be useful when writing generic code.

For simplicity, we have limited ourselves to types with single parameters (e.g. `Container<A>`). However, we could extend `deconstruct` to work with variadic templates (e.g. `Container<A, As...>` and store `As...` in an `std::tuple<As...>`) without much hassle.

## References

[1] [cppreference: Template parameters and template arguments](https://en.cppreference.com/w/cpp/language/template_parameters).

[2] [Expressiveness, Nullable Types, and Composition]({{ site.baseurl }}{% link _posts/2019-11-14-expressiveness-nullable-types-and-composition.md %}).
