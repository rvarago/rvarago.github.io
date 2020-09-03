---
layout:	"post"
title:	"The beauty of Total Functions"
---

* * *

![](/assets/img/2019-05-26-the-beauty-of-total-functions_0.png)

😱 Trust me, it's not as scary as it looks.

> And the advantages of writing APIs that don't lie to us.

Okay, the title might sound fancy, I must acknowledge that. But I have to
tell, I really love total functions. It might sound super complicated, but the
truth is: it's reasonably simple. And once followed, it can simplify our lives
a lot. It's definitely not 🚀 science at all.

In this article, I'm going to use C++, but the same principles apply
regardless of the programming language you may happen to be using.
Furthermore, I'll try to keep the examples as simple as possible, therefore
people not familiar with C++ should be able to follow them.

* * *

#### First things…Come first, do they 😔

Before start talking about total functions, let's understand why they're
useful. Not by jumping straight to the definition, but by looking at an
example where we don't use total functions, the troubles that they might
cause, and then finally at what total functions give us in that case.

Challenge 🏃:

> Write a function " _head " _that receives a list of integers and returns its
first element.

In C++, we could write this function as:

    
    
    int head(std::vector<int> elements) {  
       return elements[0];  
    }

(For the sake of brevity, I have omitted _const_ , references, templates,
etc.)

Should we say that our task is done?

(Spoiler alert): Maybe!

* * *

#### Corner cases… And where to find them 😖

Think about the following case:

  * What happens if _elements_ is an empty list?

By looking at the function's API (its signature), the involved types tell us
nothing. Therefore anything can happen as far as the signature is concerned.

But we know, or at least we have a "gut feeling", that the function has to
fail somehow in this case. Now the question should be:

  * How does it communicate such a failure condition?

Once again, the API doesn't tell us anything, there's not even a clue.

Let's move one step further and stop thinking about the implementation.
Pretend that I have only given you the function signature. What does it say?

> Give me an _std::vector <int> _and I'll give you an _int_.

It basically promises that it'll **always** return an _int_ , no matter how.

-- **Option 1: Return an "invalid" value**

For instance, in theory, it could return an "invalid" _int_ , whatsoever being
the meaning of an invalid value here, like the (in)famous _-1_ for a non-
negative list. And, of course, we have to be **absolutely** sure that this
invalid value has NO other meaning in the application, and so it has to be
reserved as invalid. If it suddenly starts to represent another thing, then
…we shouldn't use it.

Moreover, how would we communicate that _-1_ has a special meaning? Usually,
via a comment. Oh no! The year is 2019, it doesn 't sound like a good idea to
rely solely on a comment anymore… There has to be a better way.

-- **Option 2: Use an out-parameter**

We could use an _out-parameter_ , by passing a reference (or a pointer) to an
_int_ that is set inside the function if _elements_ is not empty, and then we
might return a _bool_ to communicate whether the value was correctly set or
not. We could also go the other way around, and pass the _bool_ by reference
and returning the value via return type.

This approach has some drawbacks, for example:

  * it mutates an external parameter;
  * it doesn't compose well;
  * it may prevent inlining, which might hurt performance.

-- **Option 3: Return a pointer**

We could return not an _int_ , but a pointer to an _int_ , which brings a
blurred concept of _nullability._ In this case, a pointer to an _int_ can have
all values that an _int_ can have plus one: the absence of any value =
_nullptr_ (please, keep this in mind, it 'll be useful soon).

In terms of [Algebraic Data Types](https://code.egym.de/a-brief-introduction-
to-the-algebra-of-types-df92f0820e5) that means: _#Pointer_to_int = #Int + 1_.

However, the meaning might not be clear, since a pointer also has many other
uses, i.e it has overloaded meanings, like to express that some dynamic memory
allocation has happened, etc.

-- **Option 4: Throw an exception**

Another option, which is usually thought as the obvious one, would then be to
simply throw an exception in case of an empty list having been provided.

This is generally more robust than the previous ones. However, exceptions are
usually better when applied to truly exceptional situations, something really
unexpected and, preferably, unrecoverable. For instance, a failed memory
allocation that seriously compromises the application, maybe preventing it
from starting correctly.

Needless to say that I'm not talking about performance, but more importantly,
about semantic: Would you say that passing an empty list is really an
exceptional situation? If the answer is **no** , but you still use an
exception, then maybe you're using exceptions for flow-control, which might
not be a good idea…

Another drawback of choosing an exception is that maybe we're being too eager
to already trigger an exception at this point, we may not have all the
information and context to decide it straight away. For instance, _head_ might
be a utility function deep inside a helper library far away from the main
application where the user has more information to make better decisions.

However, if we still decide to throw an exception and the user forgets to
handle it by wrapping the call in some sort of _try/catch_ block, then the
stack unwinding will happen to the point where it 's handled or crashing the
application if it's not handled.

Thus, instead of making the decision by our own, we could push it to the
client, which could then push to its client and so on up onto the call stack
to the border of the application, usually the UI, where we might have more
information to decide what we should do.

-- **Enough of options for now**

Each of them may have their pros and cons, but they all share a common problem
that was already briefly mentioned:

> The fact that the function might fail is **not** part of its API. Thus, we
have to rely on some external information: comments, access to the
implementation, etc, to understand what happens in case of failure.

> Also, even if we do know what happens in case of failure, the type-system
cannot enforce the client to handle it properly. Or, at least, to acknowledge
that such failure may happen.

Actually, the options attempted to encode the failure in the API. But it's not
as clear as it could and should be:

  * to use an out-parameter combined with a flag;
  * to return a pointer.

* * *

#### Back to our example… Where we should be ☺️

I'd like to stress out what I meant in regard to the type-system:

Consider that we've changed the signature to return a pointer and we used a
_nullptr_ to express the absence of a value. Once again, for the sake of
brevity, I 'll use a plain and simple raw pointer, rather than a
[smart](https://en.cppreference.com/book/intro/smart_pointers) one.

In the meantime, in the caller code, the user accidentally forgot to check
against _nullptr_ before accessing the returned value:

    
    
    int* head_of_list = head(std::vector{});  
     use(*head_of_list); // Oops! I forgot to check. Bad, bad user! 

It failed! 💥

We de-referenced a _nullptr._ In C++, we bought our ticket to the wonderful
land of [undefined behavior](https://en.cppreference.com/w/cpp/language/ub) 🎢,
where **anything** can happen. If you 're lucky, it'd crash your application,
but there's no guarantee at all. Hence, do not rely on it. Expect anything; in
particular, the worst.

Okay, the type-system sort of tried to alert us:

  * We had to de-reference the variable to use it.

In C++, it's nothing more than a blurred hint. But it could also be the case
where we returned a pointer because we needed dynamic polymorphism, etc. We're
not being precise over our intention:

"I need to express that I may fail to return an _int_ to you. "

The problem with pointers/references is even more troublesome in programming
languages in which the only way to access objects is through
pointer/references. That's even more critical if the null value can inhabit
all the types as it happens for non-primitive types in Java.

This means that we don't need to explicitly de-reference the variable to make
use of its content and almost everything has the potential of containing
_null_.

Therefore, our humble hint to check for emptiness goes away 🚢.

* * *

#### What happens in the original example… Oh, I almost forgot 😄

In [C++,](https://en.cppreference.com/w/cpp/container/vector/operator_at)
anything can happen. It's simply undefined behaviour to access an index _i_ of
an _std::vector vec_ which is out-of-bounds _,_ i.e. _i_ ≥ _vec.size()_.

One way to make it more robust is to use
[_at(i)_](https://en.cppreference.com/w/cpp/container/vector/at) __ that
performs bound-checking and throws an exception in case the size-constraint is
violated. But it's still an exception. And such a case might not be a truly
exceptional situation and the type-system can't enforce users to handle it
adequately.

What would we happen if we had a fifth option? It turns out, we have 👏.

* * *

#### Total Functions… Finally 😉

Let's consider:

  1. The function may fail;
  2. Such failure is not exceptional;
  3. We want to be explicit, so the type-system can enforce guarantees via its type-checker.

Therefore we want to communicate the chance of failure in the API and let the
type-checker in charge of doing what it does best: enforcing guarantees. So
callers have to handle the failure. Or, at least, be aware of it, and if they
ignore the failure case, they will do it consciously.

No accidents! Not on my shift! 💂

How can we do this? As the title suggests: total functions everywhere.

A total function can be thought as:

> A function where, for every possible value of its parameters, it always
returns a value of a given return type. Therefore, it's defined for each and
every single possible input.

Mathematically speaking 📜:

Remember that function _f_ is a mapping from one set _A_ (called domain), to
another set _B_ (called co-domain). In symbols: _f  _: _A_ ↦ _B_.

Such function is said to be **total** if:

>  _∀ a  ∈ A _⇒ ∃ _f(a) = b ∈ B_

 _i._ e for each possible input, _f_ is completely defined and returns an
element from the type of possible values of its return type.

If _f_ is not total, in this case, we say it 's **partial** , and _ ∃ x ∈ A _⇒
_ ∄ f(x) ∈ B. _This means that the function is not defined, i.e. fails, for
one or more values of the given input type. It's also possible to turn a
partial function into a total function by reducing its domain.

 _NOTE: in programming languages, we usually don 't talk directly in terms of
sets, but in terms of types, which are "proxies" for sets._

In our example, **an empty _std::vector <int>{}_ is a value that belongs to
the _std::vector <int>_ type**. Therefore, you can provide an empty list to
the function, it's a valid value like _std::vector <int>{1}_, _std::vector
<int>{1, 2}_, etc. However, for the empty case, the function is not defined,
it fails.

The function fails for one particular value and we have to communicate it.

It turns out that we can convert our partial function into a total function by
using a return type that accepts all possible values for an _int_ plus
"nothing", the absence of any meaningful value. It's similar to returning a
pointer, but we're looking for a type whose only purpose is to express the
_nullability_.

In C++, the type we're looking for is _std::optional <int>. _It's also known
as Maybe, [Option](https://code.egym.de/from-null-to-option-in-scala-
3436cfeef7b0), etc, depending on the programming language you're using, but
regardless of it, the semantics are essentially the same:

> A value of type std::optional<T> either contains a value of type T, or it's
empty.

By lifting the return type into an _std::optional <int>_ we can change the
signature of _head_ to:

    
    
    std::optional<int> head(std::vector<int> elements)

It precisely expresses the semantics we were looking for.

 _std::optional <int>_ represents a value that may (function succeeded) or may
not be there (function failed).

Now, the API says:

"Hey, this function may fail, in which case it returns an empty optional. Or
if it succeeds, then it returns the value wrapped inside the optional."

Wonderful! The signature clearly expresses what happened.

A possible implementation could then be:

    
    
    std::optional<int> head(std::vector<int> elements) {  
        if (elements.empty()) {  
            return std::nullopt;  
        }  
        return std::optional{elements[0]};  
    }

Here, if _elements_ is not empty, we simply return an empty optional (
_std::nullopt_ ). Otherwise, it accesses the first element (at this point
we're sure it's not empty because we checked it before) and returns it wrapped
in the _std::optional_.

Meanwhile in the caller code…If the user tries to use the value directly as:

    
    
    use(first_element);

It does **not** compile!  🆒

Wonderful, the type-checker enforces that the user has to acknowledge that the
function might have failed. If one forgets to acknowledge, an accident, well,
the type-checker won't forgive and will happily trigger a compilation error as
it was designed to do.

That's way more robust than an error we may only find in run-time.

One way to handle the condition in the caller code could be:

    
    
    std::optional<int> first_element = head(std::vector<int>{});  
    if (first_element) {  
        first_element.value();  
    }  
    else {   
       /* whatever we have to do in case of an empty list */  
    }

Now the user has to be explicit by invoking  _value()_. One has to think about
what should be done to handle the failure condition.

It's also possible to provide a default value such as:

    
    
    first_element.value_or(0) // if optional is empty, value_or returns 0 

_NOTE:_ Of course, unfortunately, it 's possible to directly access the
wrapped value without even checking it. But if you do, you must be explicit,
the advice was given and you were aware of it.

So don't 🚨, just don't recklessly access an optional without checking it
before.

#### Is this the best solution? Not necessarily 😮

We have turned _head_ into a total function by changing its return type, thus
augmenting the possible values that it can return. We basically have worked on
the function **co-domain** , making it bigger to accommodate an empty object
of the  **return**  type.

Another way to go would be to change the parameter type in such a way that it
reduces the possible values that it can receive. Thus, we'd work on the
function **domain** , making it smaller to not allow the empty object of the
**parameter** type.

So, instead of accepting a "raw" _std::vector <int>_, we could use a strong
type that can't be empty by definition. One example is
[_NonEmptyVector_](https://typelevel.org/cats/api/cats/data/NonEmptyVector.html)
provided to Scala via a library named _cats_ 😸 _._ This type always has one
element plus an optional "tail" of elements.

This approach is even better since the illegal state can't be represented in
the first place.

Therefore, we're once again leveraging the capabilities of the type-system to
prevent us from committing mistakes at compile time. But now, by prohibiting
us from passing an invalid argument.

Besides being slightly more complicated, this a great approach to solving
problems. And hopefully will be covered in a future article 🔜 .

* * *

#### How about composition? Bonus 😍

One thing that can sometimes be frustrating when using _std::optional_ in C++
is the fact that, so far, it doesn 't compose quite nicely as it does in other
programming languages, such as in Haskell's Maybe, Scala's Option, etc.

Consider an API that makes use of _std::optional_.

We might then have the following functions:

    
    
    std::optional<person> find_person() const;  
    std::optional<address> find_address(person const&) const;  
    zip zip_code(address const&) const;

In this case, a fairly common pattern in C++ would then be:

    
    
    auto const maybe_person = find_person();  
    if (!maybe_person) return;  
      
    auto const maybe_address = find_address(*maybe_person);  
    if (!maybe_address) return;  
      
    auto const zip_code = zip_code(*maybe_address);

  1. We found the person;
  2. Then we checked if it's not empty;
  3. Then we found the address given the previously found person;
  4. Then we checked if it's not empty;
  5. Then we found the ZIP code given the previously found address.

The checking for emptiness (steps 2 and 4) clutters our logic and hurts
readability. Ideally, we would check only once, at the end of the whole chain.
And at this point, we would decide what we should do to handle a failure.
Also, each check adds more complexity to the code, being susceptible to an
accident where we forget to cover one of them. 😐

It'd be nice to compose the operations and only check if something has failed
at the end.

Fortunately, C++ [is getting a nice monadic interface soon](http://www.open-
std.org/jtc1/sc22/wg21/docs/papers/2019/p0798r3.html)! 👌

Besides its scary name, a monadic interface **roughly** means that we 'd be
able to compose multiple _std::optional_ as we do with the types wrapped by
them.

In the meantime, there are some alternatives. One of them is to use
[_absent_](https://github.com/rvarago/absent).

 _absent_ is a tiny open-source header-only library inspired in a declarative
style encouraged by programming languages such as Haskell and Scala.

I wrote _absent_ with the main goal of lifting _std::optional_ ; actually any
optional-like type as long as it conforms to the expected API, into a monad
that can be nicely composed.

Using _absent_ , the same example may be re-written using an infix notation
as:

    
    
    auto const maybe_zip_code = find_person() >> find_address | zip_code;
    
    
    if (maybe_zip_code) {  
        /* Use the value */  
    }  
    else {  
        /* Handle failure */  
    }

 _NOTE: It 's also possible to use named function and prefix notation._

Now the check only happens once, at the end as we wanted it. And the
composition fails fast, which means that if any of the intermediate steps
returns an empty optional, then an empty optional will be returned as the
result of the whole chain of functions.

Therefore you don't need to check for every single step, _absent_ handles it
for you. That simplifies the code, and reduce the surface on which a bug can
be hidden.

 _absent_ offers even more features, for instance:

  * Different combinators for common scenarios.
  * The possibility of adapting your custom optional-like type to work with _absent_ without needing to change your type.

 _absent_ is a fairly new project that, of course, lacks various features and
improvements but it may be helpful in some circumstances.

Needless to say: since it's an open-source project, you're more than welcome
to submit PRs with suggestions and improvements 👐 .

* * *

#### Conclusion

In this article, we saw the importance of writing total functions, their
benefits and what could go wrong if we forget to think about failure
conditions.

Of course, there's no silver bullet, and sometimes you may need to resort to
other alternatives. For instance, if some serious condition from which your
application can't or shouldn't recover, then an exception may be a better
call.

Summing up, here are some advice:

  * strive to write APIs that convey the necessary information for users to understand what happens not only in that "happy path" but also when things go wrong;
  * prefer to fail at compile-time rather than at run-time;
  * remember to check for errors and prefer to define a strategy to handle them beforehand;
  * be pragmatic. Know all your tools and pick the best one for each scenario;
  * [ _absent_](https://github.com/rvarago/absent) may be an interesting project to help you compose optional-like types.

If your programming languages offer you a static type-system, then strive to
make the best possible use of it by picking up the right type to express your
intentions. Thus, you can rely on the type-checker to enforce guarantees to
your API via types that precisely express your intentions 😉.

#### References

[1] A Fistful of Monads. <http://learnyouahaskell.com/a-fistful-of-monads>.

[2] A brief introduction to the Algebra of Types. <https://code.egym.de/a
-brief-introduction-to-the-algebra-of-types-df92f0820e5>.

[3] absent: A simple library to compose nullable types in a declarative style
for the modern C++ programmer. <https://github.com/rvarago/absent>.

