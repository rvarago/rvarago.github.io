---
layout:	"post"
title:	"Object instantiation, immutability, and expression-oriented style"
---

Declarative Programming, encourages us writing programs by composing
expressions rather than statements as we usually do in Imperative Programming.
Let's see how it can help us.

* * *

![](/assets/img/2019-06-26-object-instantiation-immutability-and-expressionoriented-style_0.png)

Don't you agree that λ is a very stylish letter? 🆒

> How to use the glorious expression-oriented to handle object instantiation
based on conditions without giving up immutability.

Functional Programming (FP), or more generally Declarative Programming,
encourages writing programs by composing expressions rather than statements
that mutate state as we usually do in Imperative Programming.

Let's take a look at how we can benefit from this idea in our codebase even if
it doesn't completely adhere to FP.

#### Expressions, statements… What?? 😕

Roughly speaking, an expression is a piece of code that can be evaluated to
result in a value. Whereas a statement is an action to be carried out.

Some examples of expressions in C++:

    
    
    1+2
    
    
    condition ? value_if_true : value_if_false
    
    
    [capture_list](arguments) { return something; }

And statements:

    
    
    if (condition) {  
      // Then this  
    }  
    else {  
      // Or that  
    }
    
    
    while (condition) {  
      // Do this  
    }

#### How about instantiating an object based on some conditions❓

Consider the following example:

    
    
    struct glorious_type {  
      glorious_type() = default;  
    }

We had a type proudly 👑 named _glorious_type_ that has a default constructor,
so we can create an instance of it without needing to provide any argument
whatsoever. Something like this:

    
    
    auto const my_glorious_type = glorious_type{}

This instance is marked as _const_ , so we can't change it after being
initialized.

However, suppose that in addition to the default constructor, we also have two
different ways to initialize _glorious_type_ via invoking the builders
_build_glorious_A_ and _build_glorious_B_.

Moreover, consider that we have a piece of code in which we want to initialize
_my_glorious_type_ with one of the builders based on some condition.

Thus, we may have something like this:

    
    
    auto my_glorious_type = glorious_type{};  
    if (super_complex_condition) {  
      my_glorious_type = build_glorious_A();  
    else {  
      my_glorious_type = build_glorious_B();  
    }

And it compiles! 🎉

But note that we had to mutate the state by changing _my_glorious_type_ after
it was initialized since _if_ and _else_ in C++ are statements, not
expressions. This is represented by the fact that we had to give up _const_ 😞.

Even that the mutation is, ideally, constrained to this small piece of code,
we'd have to keep track of _my_glorious_type_ statement after statement over
its whole scope  inside a larger function. Because nothing prevents us from,
consciously or not, mutating it again afterwards, given that the object is not
marked as  _const_  (so the compiler can't enforce the guarantee on our
behalf). Therefore the possibility of mutation adds to the cognitive load for
readers trying to make sense of the function where the snippet is.

Furthermore, what happens if _glorious_type_ doesn 't have a default
constructor? As it is right now, the snippet wouldn't compile.

We would have to initialize the variable to some "fake empty state"
whatsoever, and then immediately update it to the right state based on the
selected condition. But setting up such a fake empty state may be prohibitive,
e.g. either it takes too long or triggers unintended side-effects.

And maybe even more importantly: this feels unnatural and might be the cause
of some confusion for readers trying to make sense of the code. And as you may
know, confusion is some sort of an invitation for bugs to come in 🐛.

#### Turning statements into expressions 🏃

In C++, _if_ and _else_ have a sibling that is used to make decisions, but
instead of being a statement, it's an expression. Actually, it fits nicely in
our case.

Let's say hello to the ternary operator 👐 :

    
    
    auto const my_glorious_type = super_complex_condition ?  
      build_glorious_A() :  
      build_glorious_B;

There's no need to initialize the variable to some weird temporary state 👌.
Beyond that, we've also got _const_ back, let 's embrace immutability again 🕶.

The solution works quite nicely for this use case but it doesn't scale well.
Consider the case where instead of having a simple binary condition, we have
several nested conditions: _build_glorious_A_ , _build_glorious_B_ ,
_build_glorious_C_ , and _build_glorious_D_.

Of course, we could nest ternary operators as we please, but it'd hurt
readability. Even using a bunch _if_ , _elif_ , and _else_ might not be good
in terms of readability as well. It seems that we should use _switch-case_ :

    
    
    auto my_glorious_type = glorious_type{};  
    switch (condition) {  
      case A:  
        my_glorious_type = build_glorious_A();  
      break;  
      case B:  
        my_glorious_type = build_glorious_B();  
      break;  
      case C:  
        my_glorious_type = build_glorious_C();  
      break;  
      case D:  
        my_glorious_type = build_glorious_D();  
      break;  
      default:  
        // Do whatever it has to do...  
    }

And it compiles, yup! 🎉

 _But  wait_...Oh no! 😢 A _switch-case_ is also a statement. So we had to go
back to the mutation of the state by updating the variable after being
instantiated, and that 's precisely what we wanted to avoid. No more _const_
for us.

Furthermore, it only compiles in case the type provides a default constructor.
So this isn't a general solution.

####  _std::optional <glorious_type>_ to the rescue 🚒

Yes, I have to admit that I like _std::optional <T>._ It's a very useful
vocabulary type that can help us to be more expressive while embracing type-
safety in a lot of cases. That's why I've been writing about it or variations
of it in other programming languages a few times in different contexts:

  * [The beauty of Total Functions](https://code.egym.de/the-beauty-of-total-functions-e8c35fee2d87)
  * [A brief introduction to the Algebra of Types](https://code.egym.de/a-brief-introduction-to-the-algebra-of-types-df92f0820e5)
  * [From null to Option in Scala](https://code.egym.de/from-null-to-option-in-scala-3436cfeef7b0)

Among the nice things we can do with an _std::optional <T>_ is to augment the
possible values of _T_ to include an  "empty state" represented by the empty
_std::optional <T>_, which is precisely the semantic we're looking for:

> Somehow represent an empty T, which doesn't have an empty state defined by
itself.

Thus, we can change our example to:

    
    
    auto my_glorious_type = std::optional<glorious_type>{};  
    switch (condition) {  
      case A:  
        my_glorious_type = build_glorious_A();  
      break;  
      case B:  
        my_glorious_type = build_glorious_B();  
      break;  
      case C:  
        my_glorious_type = build_glorious_C();  
      break;  
      case D:  
        my_glorious_type = build_glorious_D();  
      break;  
      default:  
        // Do whatever it has to do...  
    }

And it compiles! 🎉

Moreover, it compiles regardless of _glorious_type_ having or not a default
constructor. But on the other hand, it still insists on mutating the state. So
we can 't have it _const_ , and hence no immutability for us. That's sad 😭.

I'm a super fan of _std::optional <T> _because it's a powerful abstraction and
it helped us to achieve compilation. However, it's not enough, it's probably
not the right abstraction to our problem. We need to find another way to
achieve all of our design goals. But how? How?? How??? 💔

Don't panic dear reader, we're relentless programmers and everything is
possible for us. We want immutability and we won't give it up that easily 💪🏻.

#### Lambda expressions to an even more glorious rescue 🦸‍♂️

C++ got your back, so rest assured because we don't have to give up
immutability in order to achieve our goal. And it happens regardless of the
type having or not a default constructor. Yes, we can do it! 🔈

And that's one of the reasons that make lambda expressions really cool. They
give us the ability to write anonymous and local functions that can restrict a
scope.

By evaluating a lambda expression, it gives a value back. So, broadly
speaking, lambda expressions bring us the power to turn statements into
expressions.

Does it sound familiar? Yes, as the title of this post may suggest, lambda
expressions fit perfectly in our use case.

Let's see how:

    
    
    auto const my_glorious_type = [&condition] {  
      switch (condition) {  
        case A:  
          return build_glorious_A();  
        case B:  
          return build_glorious_B();  
        case C:  
          return build_glorious_C();  
        case D:  
          return build_glorious_D();  
      default:  
        // Do whatever it has to do...  
      }  
    }();

And it compiles! 🎉 But this time, it brings us everything we wanted before
🎉🎉🎉.

We've created a lambda expression and immediately invoked it. The _condition_
variable was provided via capture-list (it could 've also have been provided
via argument). Now comes the core idea: instead of mutating the variable, for
each case in the switch, we initialized and returned an instance of the type.
🆒

Since we didn't need the awkward fake empty state anymore, the solution
compiles regardless of having a default constructor or not.

Furthermore, we got our _const_ back, and as a bonus, we removed the now
unnecessary _break_ statements since we 're now returning from the lambda for
each case.

Alright, it may sound scary at first, because it looks like more complex code.

It might be a matter of opinion, but I'd argue that this code is at least as
complex as before and shows the same level of details: we just had to wrap the
block inside a lambda and then invoke it.

And on the bright side, we removed a few _break_ statements and several
assignments  that could lead to subtle and hard to track bugs. Actually, it
turns out that we wrote less code, which means smaller penetration surface for
bugs.

Regarding performance: yes, we do have another level of indirection, but
modern compilers have become pretty good at inlining this kind of code and
there's a chance of the whole lambda be inlined, yielding the same run-time
performance that we'd have without it. A more likely drawback is in terms of
compile-time that may increase, which may or may not be a problem depending on
your codebase.

Anyways, it could be that **after measuring** the performance of your project
you find out that this kind of code does have a significant run-time overhead.
Then, and only then, you may want to revisit the technique or even avoid it.
But before, up to some extent, I 'd prefer readability rather than a "maybe-
optimization". However, don't take my word too serious, whenever in doubt:
measure!

#### Conclusion

We saw how we can apply some concepts of Declarative Programming and the idea
of expression-oriented style to help us to solve a common problem in
programming. And the benefits of thinking in terms of expressions rather than
statements.

To sum up, we saw that using lambda expressions:

  * the initialization code works for types that don't have a default constructor available
  * the variable can be marked as _const_ given __ that it's immutable now

Lambda expressions bring us a lot of power to turn statements into
expressions.

We could also achieve pretty much the same outcome using a named function or
function object.

But it would happen at the price of some syntactic overhead and unnecessary
introduction of a symbol (the function name) to a larger scope than it has to
be visible.

Moreover, it's just a technical artifact that only makes sense to the piece of
code responsible for the object initialization. Therefore, it's a good idea to
keep it closely located to the point where it's used.

#### References

[1] [The beauty of Total Functions](https://code.egym.de/the-beauty-of-total-
functions-e8c35fee2d87)

[[2] A brief introduction to the Algebra of Types](https://code.egym.de/a
-brief-introduction-to-the-algebra-of-types-df92f0820e5)

[[3] From null to Option in Scala](https://code.egym.de/from-null-to-option-
in-scala-3436cfeef7b0)

