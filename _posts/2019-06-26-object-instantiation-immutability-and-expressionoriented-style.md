---
layout: "post"
title:  "Object Instantiation, Immutability, and the Expression-Oriented Style"
tags:   fp c++ haskell
---

> Declarative Programming encourages us writing programs by composing expressions rather than statements as we usually do in Imperative Programming. Let's briefly see how we can from this style of programming.

* * *

|![](/assets/img/2019-06-26-object-instantiation-immutability-and-expressionoriented-style_0.png)|
|:--:| 
| *Don't you agree that λ is a very stylish letter?*|

> How to use the expression-oriented style to handle object instantiation based on conditions without giving up immutability.

Functional Programming (FP), or more generally Declarative Programming, encourages us to write programs by composing expressions, rather than statements that mutate state as we usually do in Imperative Programming.

Let's take a look at how we can benefit from this idea, regardless if we don't fully adhere to FP.

## Expressions, Statement, and What?

Roughly speaking, an expression evaluates to a value, whereas a statement is an action to be carried out and optionally yield a value.

These are expressions in C++:

> 1+2
>
> condition ? value_if_true : value_if_false
>
> [capture_list](arguments) { return something; } // lambda!1!!11!!!111

And these are statements:
    
>    
> if (condition) {  
>     // Then do this  
> }  
> else {  
>     // Or do that  
>}
>   
> while (condition) {  
>   // Do this  
>}

## How About Instantiating an Object-Based on Some Conditions❓

Consider the following example:

```cpp    
struct glorious_type {  
  glorious_type() = default;  
}
```

We had a type proudly (and not creatively) named `glorious_type` and it has a default constructor so that we can create an instance of it without needing to provide any argument whatsoever. Something like this is perfectly legal:
    
```cpp
auto const my_glorious_type = glorious_type{};
```

This instance is marked as `const`, meaning that we can't change it once initialized (well, we can always ~~`const_cast`~~, just don't!).

However, suppose that, in addition to the default constructor, we also have two different ways of initializing an instance of `glorious_type`, with the builder: `build_glorious_A` and `build_glorious_B` (naming seems quite hard, indeed).

Moreover, consider that we have a piece of code where we want to initialize `my_glorious_type` with one of the two builders based on some condition.

Thus, we may end up with something that resembles this:

```cpp    
auto my_glorious_type = glorious_type{};  
if (super_complex_condition) {  
  my_glorious_type = build_glorious_A();  
else {  
  my_glorious_type = build_glorious_B();  
}
```

This code compiles and it's perfectly legal! 🎉

However, notice that we had to initialize `my_glorious_type` and then assign it depending on `super_complex_condition`. Fundamentally, this occurs because `if` and `else` are C++ statements, instead of expressions. This is acknowledged by the fact that we had to give up `const` 😞.

Even so, the mutation is constrained to this small piece of code, we'd have to keep track of `my_glorious_type` all over its whole scope to understand where and how it changes, which may be a large function. Given that the object is not marked as `const`, nothing prevents us from, consciously or not, mutating it again afterwards. The compiler can't enforce the guarantee on our behalf, we've explicitly disabled since we removed `const`. Tacking possible mutations adds to our cognitive load whenever we try to reason about we do with `my_glorious_type` throughout its entire scope.

Furthermore, what happens if `glorious_type` doesn't have a default constructor? The snippet would then fail to compile.

We would have to initialize the variable to some "fake empty state" (whatever that means), and then update it to the right state based on the
predicate that has been satisfied. However, defining such a fake empty state may be prohibitive (e.g. it involves a computationally expensive process) or simply impassible (e.g. it triggers unintended side-effects).

Perhaps even more important: this feels unnatural and might confuse readers trying to make sense of our code. And confusion opens up the door for bugs to come in and say hi 🐛.

## Turning Statements into Expressions 🏃

In C++, `if` and `else` have a sibling that is also used to make decisions, but instead of being a statement, it's an expression. That's the ternary operator and it fits our bill quite nicely:
    
```cpp
auto const my_glorious_type = super_complex_condition
  ? build_glorious_A()
  : build_glorious_B;
```

There's no need to initialize `my_glorious_type` to some awkward and ephemeral state. Beyond that, we've also got `const` back, we shall embrace immutability once again.

The solution serves well to our use-case but it, unfortunately, doesn't scale very well.

Consider the case where instead of having a simple binary condition, we have several conditions and thus builders, say `build_glorious_A`, `build_glorious_B`, `build_glorious_C`, and `build_glorious_D`.

Of course, we could nest ternary operators as we please, but that'd hurt readability. Even using a bunch `if`, `elif`, `elif`, and `else` might not be the best options as far as readability is considered. This seems like a job for `switch-case`:

```cpp
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
    // Do whatever it has to do here.  
}
```

This code compiles and once again it's perfectly legal, yup! 🎉

Hang on...Oh no! 😢

`switch-cases` are also statements. This means that we had to go back to initialize `my_glorious_type` to a weird value and then modify it based on the matched case. We had to give up `const` again.

As before, the code compiles just because the `glorious_type` happens to provide a default constructor.

I am not happy and we shall move forward.

## `std::optional<glorious_type>` Comes to the Rescue 🚒

Yes, I have to admit that I happen to like `std::optional<T>`, even being far from perfect. It's a super useful vocabulary type that helps us to be more expressive while embracing type-safety in plenty of scenarios. That's why I've been writing about it or variations thereof in other programming languages a few times in different contexts:

  * [The Beauty of Total Functions.]({{ site.baseurl }}{% link _posts/2019-05-26-the-beauty-of-total-functions.md %})
  * [A Brief Introduction to the Algebra of Types.]({{ site.baseurl }}{% link _posts/2019-03-22-a-brief-introduction-to-the-algebra-of-types.md %})
  * [From null to Option in Scala.]({{ site.baseurl }}{% link _posts/2018-10-29-from-null-to-option-in-scala.md %})
  
Among the features that `std::optional<T>` provides, we can use it to augment the possible values of `T` to include an "empty state" represented by a default-constructed `std::optional<T>{}` (even better, `std::nullopt`), which is the semantic we were looking for to represent the fake and ephemeral state that `my_glorious_type` has to be in after being declared and before being modified based on the matching condition:

> We want to represent an empty `T`, but `T` itself doesn't have an empty state.

Thus, we can change our example to:

```cpp    
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
```

This code compiles even if we had removed the default constructor of `glorious_type`! 🎉

On the flip-side, it still insists on mutating the state. So, we can't have it marked as `const`, and hence no immutability for us.

That's sad 😭.

I'm a fan of `std::optional<T>`. It's a powerful abstraction and it has helped us to make the code to compile. However, it's not enough, it's probably not the right abstraction at this moment. We need to find another way to achieve all of our design goals.

But how? How?? How??? 💔

Don't panic dear reader.

We're relentless programmers and everything is possible or can be made possible. We want immutability and we shall not rest until we have it 💪🏻.

Good news is that we are getting closer, but not just close enough.

## Lambda Expressions Comes to an Even More Glorious Rescue 🦸‍♂️

Rest assured, C++ has got us covered. We don't need to give up immutability to achieve our goal, regardless of `my_glorious_type` having a default constructor or not.

> Yes, we can do it! 🔈

That's one of the use-cases that makes lambda expressions so cool. They give us the ability to write anonymous and local functions that can restrict a scope.

Evaluating a lambda expression gives value back.

> Therefore, lambda expressions bring us the power to turn statements into expressions.

Does that sound familiar? Yes, as the title of this post might have hinted, lambda expressions fits beautifully in our use case.

Here's how:

```cpp
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
```

This code compiles! 🎉 But this time, it brings us everything we wanted before. `const`ness is back in town 🎉🎉🎉.

The trick is: we created a lambda expression and then immediately invoked it.

The `condition` variable was captured into the lambda's body (it could've also been provided as an argument). Now, here comes the core idea: rather than mutating `my_glorious_type` inside each case, we returned an instance of the type that was then used to construct `my_glorious_type` once and for all.

Since we didn't use the awkward fake empty state anymore, the solution compiles regardless of having a default constructor.

Furthermore, we got `const` back. And as a bonus, we could remove unnecessary `break` statements, given that we're only returning from each case inside the lambda.

Alright, that may sound scary and the code may look odd at first sight.

It might be a matter of taste and opinion, but I'd argue that this code is at most as complex as before, with the same level of details: we just had to wrap the whole block inside a lambda and then invoke it.

On the bright side, we removed a few `break` statements and several assignments that could lead to subtle and hard to track bugs. It turns out that we wrote less code, which means smaller penetration surface for bugs.

Regarding performance:

> Yes, we do have another level of indirection, but modern compilers have become pretty good at inlining this kind of code and there's a chance of the whole lambda will be inlined. Furthermore, by bringing `const` back, we've made more promises to the compilers and increased its visibility, therefore it might do a better job at optimizing our code than when we had a series of assignments. Whenever in doubt: measure.

## Conclusion

We saw how we can apply some concepts of Declarative Programming and the idea of expression-oriented style to help us to solve a common problem in
programming. And the benefits of thinking in terms of expressions rather than statements.

To sum up, we saw that using lambda expressions:

  * The initialization code works even for types that don't have a default constructor available.
  * The initialized variable can be marked as `const`, given that it's immutable.

Lambda expressions bring us a lot of power to turn statements into expressions.

We could also achieve pretty much the same outcome using a named function or a function object. However, that would happen at the price of some syntactic overhead and unnecessary introduction of a symbol (the function name) to a larger scope than it has to be visible.

Moreover, it's just a technical artefact that only makes sense to the piece of code responsible for the object initialization. Therefore, it's a good idea to keep it closely located to the point where it's used.

Your mileage may vary.

## References

[1] [The Beauty of Total Functions.]({{ site.baseurl }}{% link _posts/2019-05-26-the-beauty-of-total-functions.md %})

[2] [A Brief Introduction to the Algebra of Types.]({{ site.baseurl }}{% link _posts/2019-03-22-a-brief-introduction-to-the-algebra-of-types.md %})

[3] [From null to Option in Scala.]({{ site.baseurl }}{% link _posts/2018-10-29-from-null-to-option-in-scala.md %})

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
