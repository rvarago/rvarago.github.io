---
layout: "post"
title:  "Haunting Bugs with Phantom Types"
tags:   type-system adt fp c++
---

> Phantom types are useful to encode information about how, when, and where values are supposed to be used, which can catch a class of bugs early.

* * *

|![Spooky type](/assets/img/2020-05-18-haunting-bugs-with-phantom-types_0.png)|
|:--:| 
| *For once, let's try to haunt bugs rather than be haunted by them 🐛.*|

In programming, we frequently implement protocols that our building blocks are supposed to abide by, e.g. _if x, then y, otherwise z_. We are sequencing a series of operations and each operation may depend on the application's current state.

Often, protocols are implicit in our minds, and we don't make assumptions as explicit and formal in the code as they truly are in reality.

Protocols might also vary in complexity, ranging from a straightforward _read a character from the keyboard, then write it to the terminal_, to a fully-fledged [TCP](https://tools.ietf.org/html/rfc793).

Regardless of its complexity, when implementing a protocol we normally need to restrict how, where, and when values are supposed to be used. We can formalize such constraints in different forms, one is by the meticulous use of types.

I'll be using C++ to make exemplify the underlying principles. However, the same ideas should be more-or-less valid in other languages, as long as they afford parametrizing types with types (templates, generics, etc).

Let's say that we have a small application that implements the protocol:

1. Reads an SQL query from some external source, e.g. keyboard.
2. Then runs such a query in a database.

Perhaps we could with end up with the following set of functions:

```cpp    
string read_query();  
void run_query(string query);

// Use it like so:
run_query(read_query());
```

This should fit the bill just fine.

However, as far as our protocol is concerned, nothing prevents a malicious query from running, which may lead to bad consequences, e.g. SQL injection.

To cope with that scenario, we could make our protocol slightly more complex, albeit safer, by introducing an intermediate step that should be responsible for sanitizing the query **after** reading it from the external source, but **before** running it in the database:

```cpp    
string read_query();  
string sanitize_query(string raw_query);  
// NOTE: Must be called after sanitize_query has been called.  
void run_query(string query);

// Use it like so:
run_query(sanitize_query(read_query()));
```

This looks safer. The new function `sanitize_query` makes sure that no malicious query gets run, perhaps by throwing an exception when detecting it. Please notice the comment at `run_query`, which states that it must be called with a properly sanitized query, i.e. after `sanitize_query` has been called, otherwise, the protocol shall be considered violated.

That's still not great and hence we may land into troubles.

Essentially, strings are far too low-level and make no distinction between raw and sanitized queries. Even worse, a string generally doesn't even hold enough information in its type to clearly state that it represents a query at all. Hence, one might accidentally skip the comment and then break the assumption made by the protocol by calling into `run_query` with a raw query that hasn't been sanitized:

```cpp    
run_query(read_query()); // Oops! We forgot to sanitize.
```

It would be nice if we could harden the implementation, such that this kind of violation would be readily rejected by the compiler.

Luckily, there are several ways to do that. One is by employing phantom types, where the type-checker can statically enforce guarantees based on extra bits of information we've added into the types. If one fails to adhere to the protocol, then a compilation error is triggered.

## Phantom Types

Phantom types encode information at type-level about where, when, and how values are supposed to be used. It's a popular idiom in Haskell and occasionally shows up in other languages.

> Roughly speaking, we say that a parameterized type `X<T>`, where `T` is a type-parameter is a phantom type if its type-parameter `T` doesn't appear in `X<T>`'s definition, i.e. implementation or body.

That may sound scary (no pun intended), but truth is that usage is reasonably simple:

```cpp
template <typename T>  
struct X {  
};
```

`X<T>`'s body doesn't refer to `T`. For instance, there are no member-variables of type `T`.

That's pretty much the reason why we call them phantom types, their type-parameters don't manifest in the type itself with no meaning at the "value-level", they are rather used only at the "type-level". At first sight, we could simply strip `T` off and nothing would change.

> The sole purpose of phantom types is to help the type-checker during static analysis.

Instead of declaring member-variables of type `T`, the real purpose of `T` is to encode extra-information into `X<T>` that will only be used at the type-level during type-checking as part of the larger compilation process. Such information gives enough power to the type-checker, so it can enforce what the protocol expects.

Armed with phantom types, we can start re-writing our SQL query example:

```cpp
struct Raw{};  
struct Sanitized{};  
    
template <typename SanitizationState>  
struct Query {  
    string value;  
};
```

We've introduced two empty data structures (type-tags): `Raw` and `Sanitized`, whose sole purpose is to encode the set of valid states that a query can be in:

  * `Raw` -- The query has been read from an external source and it's ready to be sanitized.
  *  `Sanitized` -- The query has been sanitized and it's ready to be run in a database.

`Query<SanitizationState>` is the actual phantom type, holding the query string in its member-variable `value`. Notice that `SanitizationState` is not used anywhere inside `Query<SanitizationState>`, it simply goes away once compilation is finished.

We expect `SanitizationState` to either be `Raw` or `Sanitized`, this means that types such as `Query<double>` or `Query<Foo>` wouldn't make any sense and shouldn't be permitted. We can go further and refine our design by pulling some meta-functions in to statically enforce this new requirement and fail the compilation should `SanitizationState` be anything other than `Raw` or `Sanitized`:

```cpp
template <typename SanitizationState>  
struct Query {  
    static_assert(std::disjunction_v<std::is_same<SanitizationState, Raw>, std::is_same<SanitizationState, Sanitized>>, "invalid sanitization state");  
    
    string value;  
};
```

Lastly, instead of having functions that accept and return plain strings, they will operate on the embellished types `Query<Raw>` and `Query<Sanitized>`:

```cpp
Query<Raw> read_query() {  
    string raw_query = read_raw_query_from_source(); // Reads from the external source.   
    return Query<Raw>{raw_query};  
}  
      
Query<Sanitized> sanitize_query(Query<Raw> const& raw_query) {  
    string sanitized_query = sanitize_query_impl(raw_query.value); // Applies whatever algorithm for sanitization.   
    return Query<Sanitized>{sanitized_query};  
}
      
void run_query(Query<Sanitized> const& sanitized_query);
        
run_query(sanitize_query(read_query()));
```

We've established an explicit order for the operations at the type-level. Now, if we attempt to violate the protocol, perhaps by forgetting to sanitize a query before running it:

```cpp
run_query(read_query());
```

We shall get a compilation error as the types don't match. `read_query` returns `Query<Raw>`, whereas `run_query` accepts `Query<Sanitized>`, and `sanitize_query` is the function encharged to convert the former into the latter.

The error message might look like:
    
    error: no matching function for call to 'run_query'

That happens because the type-checker is more aware of the protocol and thus can help us to enforce it.

Even with phantom types, one can still by-pass the rules and instantiate a `Query <Sanitized>` directly without any sanitization at all (we have public access to its constructor). However, this might be less likely to occur accidentally. The types now advertise that there's a protocol underneath, and also suggest how such a protocol should be used.

On top of that, we could do even better and limit the functions that are allowed to instantiate the types on any given state, for example, `sanitize_query` would forcefully be the single place where `Query<Sanitized>` can be instantiated. The [pass-key idiom](https://arne-mertz.de/2016/10/passkey-idiom/) could be helpful to achieve such a design goal. Notwithstanding that this may imply in more boilerplate that would then lead to an even more complex implementation and therefore trade-offs might need to be taken into account.

## Conclusion

We saw that phantom types help us to encode how values we expect values to be used in a protocol and thus establish a chain of type-safe operations on them.

In the example, we had only two possible states, but nothing stops us from having more if we need it.

There are also alternatives to achieve the same result, for instance, by defining types like `RawQuery` and `SanitizedQuery`. Moreover, some idioms are essentially different realizations of the same underlying ideas. Although, given some optimization metric, not necessarily a single solution solves all the problems equally well. Consequently, it's up to our requirements and judgment.

Furthermore, instead of free-functions, we could very well have used member-functions with `static_assert`s and/or SFINAE, which sometimes may lead to more pleasant APIs. Also, it's possible to strengthen the protocol even more by controlling access to the constructors, making the whole implementation much more robust. The design space is rich and offers many paths to be explored. 

Lastly, phantom types introduce boilerplate that increases the overall implementation complexity. Thus, it may not worthwhile in all circumstances. Perhaps a comment or an assertion in addition to a reasonable test-coverage might be good enough in some cases.

Nonetheless, I regard phantom types as a useful tool to keep in our belts and pull it in whenever the time comes.

## References

[1] [Phantom type - Haskell Wiki.](https://wiki.haskell.org/Phantom_type)

[2] [Phantom types - kean.github.io.](https://kean.github.io/post/phantom-types)

[3] [Phantom Sub-typing.](https://www.cs.rit.edu/~mtf/research/phantom-subtyping/jfp06/jfp06.pdf)

[4] [Algebraic Data Types and Data Modelling.]({{ site.baseurl }}{% link _posts/2019-12-19-algebraic-data-types-and-data-modelling.md %})

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
