---
layout:	"post"
title:	"Haunting bugs with phantom types"
---

* * *

|![Spooky type](/assets/img/2020-05-18-haunting-bugs-with-phantom-types_0.png)|
|:--:| 
| *For once, let's try to haunt bugs rather than be haunted by them 🐛.*|

> Phantom types are useful to encode information about how and where values
are supposed to be used, which can catch a class of bugs early.

In programming, we frequently implement protocols that our building blocks are
supposed to follow, e.g. _if x, then y, otherwise z_. We are sequencing a
series of operations and each operation may depend on the application 's
current state.

Sometimes the protocol is implicit in our head, and we don't make assumptions
as explicit and formal in the code as they are in reality.

A protocol might also vary in complexity, ranging from a straightforward _read
a character from the keyboard, then write it to the terminal_ , to a fully-
fledged [TCP](https://tools.ietf.org/html/rfc793).

Regardless of its complexity, when implementing a protocol we normally need to
restrict how and/or where and/or when values are supposed to be used. We can
formalize such constraints in different forms, and one is by the meticulous
use of types.

I'll be using C++ from now on, but the same ideas should be valid in other
languages that afford some basic support for parametrizing types with types
(templates, generics, etc).

Let's say that we have a small application that implements the protocol: 1)
reads an SQL query from some external source, e.g. keyboard, 2) and then runs
it in a database:

    
```cpp    
string read_query();  
void run_query(string query);


run_query(read_query());
```

This should work fine. However, as far as our protocol is concerned, nothing
prevents a malicious query from running, which may lead to bad consequences,
e.g. SQL injection.

To cope with that, we could make our protocol slightly more complex, albeit
safer, by introducing an intermediate step that should be responsible for
sanitizing the query **after** reading it from the external source and
**before** running it in the database:

    
    
    string read_query();  
     string sanitize_query(string raw_query);  
    // NOTE: Must be called after sanitize_query has been called.  
    void run_query(string query);
    
    
      
    run_query(sanitize_query(read_query()));

That looks safer. The function _sanitize_query_ is making sure that no
malicious query gets run, perhaps by throwing an exception when detecting it.
Notice the comment on _run_query_ , which states that it must be called with a
properly sanitized query, i.e. after _sanitize_query_ has been called.
Otherwise, the protocol shall be considered violated.

Nevertheless, strings are too low-level and make no distinction between raw
and sanitized queries. Actually, a string doesn't even hold enough information
in its type to clearly state that it represents a query at all. Hence, one
might accidentally skip the comment and then break the assumption made by the
protocol by passing in a raw query that hasn't been sanitized:

    
    
    run_query(read_query()); # Oops! Forgot to sanitize.

It would be nice to strengthen the implementation in such a way that this kind
of violation wouldn't compile at all.

There are several means to do that, and one of them is by using phantom types,
where the type-checker can statically enforce guarantees based on extra-
information added to the types. If one fails to adhere to the protocol, then a
compilation error is triggered.

#### Phantom types

Phantom types encode information at type-level about where, when, and how
values should be used. It's a fairly popular idiom in Haskell and occasionally
shows up in other languages.

> Roughly speaking, we say that a parameterized type _X <T>, _where _T_ is a
type-parameter, is a phantom type if its type-parameter _T_ doesn 't appear in
_X <T>_'s definition, i.e. implementation or body.

This may sound scary (no pun intended), but as a matter of fact, the
implementation is relatively simpler than it sounds:

    
    
    template <typename T>  
    struct X {  
    };

 _X <T>_'s body doesn't refer to _T._ For instance, there are no member-
variables of type _T_. That 's basically the rationale for calling them
phantom types as the type-parameter is not "concretely" used in the type
itself, having no specific meaning in it at the value-level, but rather as an
abstract element that only exists at the type-level. At first sight, we could
simply strip _T_ off and nothing would change.

Instead of declaring member-variables of type _T_ , the real purpose of having
_T_ is to encode extra-information at the type-level into _X <T>_. Such
information gives enough power to the type-checker, so it can enforce
guarantees that a protocol expects.

With phantom types, we can start re-writing our SQL query example:

    
    
    struct Raw{};  
    struct Sanitized{};  
      
    template <typename SanitizationState>  
    struct Query {  
        string value;  
    };

We've introduced two empty data structures (type-tags) _Raw_ and _Sanitized_ ,
whose single purpose is to encode the set of valid states that a query can be
on:

  *  _Raw  -- _query has been read from an external source and it's ready to be sanitized.
  *  _Sanitized  -- _query has been sanitized and it's ready to be run in a database.

 _Query <SanitizationState>_ is the actual phantom type and it holds the query
string in its member-variable _value_. Notice that _SanitizationState_ is not
used anywhere inside _Query <SanitizationState>_.

We expect _SanitizationState_ to be either _Raw_ or _Sanitized_ , i.e. types
like _Query <double>_, _Query <Foo>_, etc, don't make sense. It's possible to
go further and use meta-functions to statically enforce this specification and
fail the compilation if _SanitizationState_ is anything other than _Raw_ or
_Sanitized_ :

    
    
    template <typename SanitizationState>  
    struct Query {  
        static_assert(disjunction_v<is_same<SanitizationState, Raw>, is_same<SanitizationState, Sanitized>>, "invalid sanitization state");  
      
        string value;  
    };

Lastly, instead of having functions that accept and return plain strings, they
now operate on embellished types _Query <Raw>_ and _Query <Sanitized>_:

    
    
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

We have established an order for the operations at the type-level. Now, if we
attempt to violate the protocol, perhaps by forgetting to sanitize a query
before running it:

    
    
    run_query(read_query());

We shall get a compilation error as the types don't match, given that
_read_query_ returns _Query <Raw>_, whereas _run_query_ accepts _Query
<Sanitized> _and  _sanitize_query_  is precisely the responsible from
converting the former into the latter.

The error message might look like:

    
    
    error: no matching function for call to 'run_query'

This happens because the type-checker is more aware of the protocol and can
help us to enforce it.

* * *

Even with phantom types, one can still by-pass the rules and instantiate a
_Query <Sanitized>_ directly without any sanitization at all. However, this
might be less likely to occur accidentally, as the types advertise that
there's a protocol underneath and also suggest how such protocol should be
used.

On top of that, we could even limit the functions that are allowed to
instantiate the types on any given state, for example, _sanitize_query_ would
forcefully be the single place where _Query <Sanitized>_ can be instantiated.
The [pass-key idiom](https://arne-mertz.de/2016/10/passkey-idiom/) could be
helpful to achieve it. Notwithstanding that it may imply in more boilerplate
that would then lead to an even more complex implementation and therefore
trade-offs might need to be taken into account.

#### Conclusion

We saw that phantom types can be used to encode how values are expected to be
used in a protocol and thus establish a chain of type-safe operations on them.
In the example, we had only two possible states, but nothing stops us from
having more if we need.

There are also alternatives to achieve the same result, for instance by
defining types like _RawQuery_ and _SanitizedQuery_. Moreover, some idioms are
essentially different realizations of the same underlying ideas. Although,
given some optimization metric, not necessarily a single solution solves all
the problems equally well. Consequently, it 's up to our requirements and
judgment.

Furthermore, phantom types introduce boilerplate that increases the overall
implementation complexity. Therefore, it might not worthwhile in all projects.
Perhaps a comment, an assertion, and/or a reasonable test-coverage, etc might
be more suitable to handle protocol violations in some situations.

Nonetheless, I regard phantom types as a useful tool to keep in our belts and
pull it in whenever the time comes.

#### References

<https://wiki.haskell.org/Phantom_type>

<https://kean.github.io/post/phantom-types>

<https://www.cs.rit.edu/~mtf/research/phantom-subtyping/jfp06/jfp06.pdf>

[https://code.egym.de/algebraic-data-types-and-data-modelling-
7d05a77a510?gi=9b0a44f3cfd7](https://code.egym.de/the-beauty-of-total-
functions-e8c35fee2d87)


***
*Originally published at https://medium.com/@rvarago*
