---
layout:	"post"
title:	"From null to Option in Scala"
---

null has been used to represent the absence of values, but Scala offers us are
better alternatives to model this scenario: Option

* * *

> NULL, nullptr, null, etc. have been used to represent the absence of values
for some many years, but there are better alternatives to model this scenario.
In Scala, the answer is: Option.

![](/assets/img/2018-10-29-from-null-to-option-in-scala_0.png)

"Scala Icon". Source: <https://github.com/OlegIlyenko/scala-
icon/blob/master/README.md>

Normally, when we call a piece of code (function, method, etc) we expect that
it'll return a **meaningful** value that will be further processed by us
afterwards.

For instance, the bread and butter task for a user management system:

  1. Fetch a user from some storage by name
  2. Access the fetched user's role
  3. Print the name of the retrieved roles

Sounds really straightforward, right?

We could come up with the following solution in Scala as our first attempt:

What do you think? Is it right? The answer is: it depends! What?? :S

  * Is it allowed to call _findUser_ with a name that doesn 't have a corresponding user?
  * Is it allowed to have a user without any role?

If you answered "yes" for one or both of the above questions, then this
solution is wrong! Why?

For the normal scenario, _findUser_ will return a user, then _getRoles_ will
return its roles that will be printed by _printRolesFromUser_. Everything went
fine, it 's the "happy path".

However, if we didn't find a user, what will _findUser_ return? And if the
user doesn 't have any role, what will _getRoles_ return? We need to model
these scenarios in our code, but how? The common and not so good solution is
to express this absence of value by using _null_ , which is the default value
for references, we can think of it as a reference to nowhere or an empty box.
As we, unfortunately, use it in cases where we can't return what the user
expects, instead we need to represent the missing value somehow to make the
code compile.

But, think about the _printRolesFromUser_ 's perspective, by inspecting the
used API, that is the called functions' signatures, it can't know what they
return to represent the absence of a value.

  1. Maybe _findUser_ returns an "empty" _User_ and _getRoles_ returns an empty collection of _Role_ s, which is the best case.
  2. On the other hand, one or both can return _null_. __ And accessing _null_ will lead us to the tragic _NullPointerException_ (NPE).

Our interface is violating the [Principle of Least
Astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment)
because it doesn't make this kind of situation explicit, so in theory,
"anything could happen".

Suppose that both can return _null_ , so what should we do? You can say: we
should handle the possibilities of _null_!

Then, we have our second attempt:

The NPE is fixed, but now a simple and elegant one-line function has 7 lines
and a higher [cyclomatic
complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity), mostly
because of the checks against _null_ values, a lot of noise!

How can we fix the NPE without adding unnecessary complexity to the code,
keeping it simple and focused solely on the business requirements?

#### The right Option for the right semantic

To tackle the problem, first we need to understand its root cause:

Our interface doesn't express the possibility for the **absence of a value** ,
which **is a perfectly valid scenario in our application**. No more and no
less. I mentioned,  "valid scenario" to emphasize that **it 's not an
exceptional situation**, hence throwing an _Exception_ is completely ruled
out.

Simply put, we need to make the **interface** states that it may not return a
meaningful value. So the client can be aware of this fact and forced to handle
it properly according to its goal.

In the old days, we've used _null_ , but as we discussed, _null_ has the
basically the following flaws:

  1. We can't express the possibility for the absence of value in the interface, just in the implementation
  2. We're not forcing our clients to handle the possibility for _null_ , so anything can happen when it appears given that we hadn't warned them

Therefore, the clients can only guess, look at the implementation (not always
possible and/or desirable) or rely on documentation stating what the functions
are supposed to do in those cases. But documentation is inferior to code, it
isn't compiled, tested, can be out of date, nothing can guarantee that the
user will read it, etc. Remember:

> Whenever possible, document the intent of the code in the code itself,
mainly in the API.

Hence, it'll be nice if we could express the "absence of value" semantic in
the interface; spoiler: we can! The Scala way to handle this is by using the
_Option_
[monad](https://en.wikipedia.org/wiki/Monad_%28functional_programming%29).

 _Option_ is basically a type that models the possibility that a value may or
may not exist, and it 's exactly what we need! By stating in the interface
that our function returns an _Option_ , we're making it explicit and so we're
forcing the clients to handle it. No more surprises, which is a positive
effect of writing clearer and stronger interfaces, so **easier to use
correctly**. Quoting Scott Meyers 's Effective C++ Item 18:

> Make interfaces easy to use correctly and hard to use incorrectly.

In practical terms, _Option_ has two sub-classes or possibilities:

  *  _Some_ => indicates that the value has been found and wraps it inside
  *  _None_ = > indices that the value is absent

This is exactly what we need! By changing the interface to return _Option_ ,
we're sending a clear message to our clients: be prepared to handle the case
when I can't return a value to you. In this case, the client will receive
_None_ and it should handle it properly because he was warned about this
possible outcome. As I've said before: no more surprises here!

> To be strict, _Option_ is a **monad** , which is a very important concept
from Category Theory and vastly applied in Function Programming. Practically
speaking, it a type __ that supports two operations: _identity_ ( _unit_ ) and
_bind_ ( _flatMap_ ). Unfortunately, a deep discussion about monads is far
beyond the scope of this article, so it'll be postponed for a future
opportunity.

The most basic way of using the returned _Option_ is by calling its
_getOrElse_ method, which returns the wrapped value when invoked from a _Some_
or returns the default value supplied as an argument when invoked from a
_None_. Like this:

Some of the nicest properties of _Option_ :

  * Works pretty well when combined with pattern matching
  * Has suitable algorithms to handle it (like _map_ , _flatten_ , etc)

Let me show you those properties working:

Another awesome _Option_ 's feature (thanks for it being a monad!):

  * Supports _for-comprehension_

And this is exactly what we'll use for our initial example:

So much better! No checks, just the logic to handle our requirements, the
treatment for _None_ is done by the _for-comprehension_ , which is a syntactic
sugar for a combination of _flatMap_ , _map_ , _withFilter_ and _foreach_.
Here, it 'll exit the loop if _findUser_ or _getRoles_ returns _None_.
Otherwise, it retrieves the collection of roles that are used in the _yield_
clause by the _foreach_.

#### Conclusion

Although _null_ is extensively used, nowadays we have better alternatives to
express its common intent: the absence of a value. Particularly, Scala has
native support for the _Option_ monad which serves for this exact purpose, a
value that may or may not exist. Moreover, it clearly expresses in the
interface that the operation might not be able to return a meaningful value,
hence forcing the client to handle the situation properly.

By using _Option_ , we elegantly reduce the chance for the (in)famous
_NullPointerException_ , together with the noise associated with _null_
checks, etc. As always, the goal is to clearly model the semantics of a piece
of code using the right grammar offered by the programming language used to
write that piece of code. Remember: Humans should be the first client of your
code, then the compiler.

Other languages also offer _Option-_ like solutions, like Java's
_java.util.Optional_ , C++'s _std::optional_ , Haskell's _Maybe_ , __ etc.
Check those alternatives and others that your favorite programming language
may provide to you.

As always in life, I meant Software Engineering, there is no "one solution for
all the problems", and this principle applies for _Option_ too. Sometimes a
different approach can be more suitable, for instance, Null Object Pattern,
_Either_ , _Try_ , etc. Therefore, it worthwhile to known other tools, so we
can be ready to solve the problem with the right solution, correctly and
elegantly.

#### References

[1] Martin Odersky. "Programming in Scala".

[2] Dean Wampler, Alex Payne. "Programming Scala: Scalability = Functional
Programming + Objects".

[3] Scott Meyers. "Effective C++".

* * *

 _Note:_

> Although the adoption of Scala has been suspended at eGym, this article can
be helpful for providing a language-agnostic way to express the absence of
value, instead of _null._ Hence, improving your interfaces.

> The technologies used at eGym are listed on our [Tech Radar,](https://tech-
radar.co.ts.egym.com) but that's a topic for future articles.


***
*Originally published at https://medium.com/@rvarago*
