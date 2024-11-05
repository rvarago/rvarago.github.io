---
layout: "post"
title:  "Property-based Testing in Golang"
tags:   tests golang
---

> Testing is a critical part of software development. Let's see how property-based testing can help us to test our Go programs.

* * *

|![](/assets/img/2020-03-12-propertybased-testing-in-golang_0.png)|
|:--:| 
| *An enterprise Gopher with a tie and long beard ready to fight bugs alongside us. [Gopherize.me](https://gopherize.me/gopher/1695ff01da3dda465e7874f291c3a1e6554a2f90)*|

> A few weeks ago, I gave a talk about Property-based Testing using Golang on the Engineering Summit 2020 @ eGym, which I tidied up, included more details and now I want to share with you in this blog post.

* * *

Programming is an activity carried out by humans, and hence prone to mistakes. We can make mistakes and, sporadically, we do.

However, as professionals, we have techniques, practices, and tools that help us to catch mistakes as early as possible and therefore reduce the likelihood of deploying them to production. We have: code review, pair programming, a myriad of programming languages, type-checkers, linters, static analyzers, sanitizers, proof assistants, automated testing (how about TDD? 😉), CI, and the list goes on.

In this post, we're going to briefly talk about unit tests, particularly property-based testing, which is a way of writing tests based on the general governing principles that our program should uphold for all valid inputs that it can ever process.

Before, we're going to quickly glance over other approaches to writing unit tests, shall we?

## Unit Tests

We commonly write unit tests to increase our confidence that our unit-under-test correctly satisfies the specification.

They provide us with fast feedback, which tells us whether we have committed a mistake. So we have a chance to readily fix it before it propagates to later stages of our development pipeline, and eventually landing into production.

A well-written suite of unit tests also serves as live documentation, exposing how the program should behave.

Further, a failing unit test should look like a good bug report, pointing us to the problem.

Commonly, we write unit tests by listing particular examples of inputs that are fed into our unit-under-test (function, object, module, or whatever that is), and then asserting that the obtained output matches what we were expecting.

More precisely, it says:

> Given an input _x_ _in_ _X_, an expected output _y_ _in_ _Y_, and an unit-under-test _f: X - > Y_:
>
> Assert that _f(x) == y_

To reify this idea, we will look at a contrived, yet illustrative, Go function that computes the sum of two integers:
    
```go
func Add(x, y int) int {   
  return x + y  
}
```

Our goal is to test it.

### Example-based Testing

We may manually write examples in the form {_input_, _expectedOutput_}, and then assert that _expectedOutput_ equals to obtained _output_:

```go    
expectedOutput := 1  
  
if output := Add(0, 1); output != expectedOutput {  
  t.Errorf("Add fail, obtained: %v, expected: %v.", output, expectedOutput)  
}
```

### Table-driven Testing

Perhaps we could factor out the examples into a table, separating the examples from the code that exercises them:

```go
type InputPair struct {  
  left int  
  right int  
}  
  
type Example struct {  
  input InputPair  
  expectedOutput int  
}  
  
examples := []Example {  
  {InputPair{0, 0}, 0},  
  {InputPair{1, 0}, 1},  
}  
  
for _, e := range examples {  
  if output := Add(e.input.left, e.input.right); output != e.expectedOutput {  
      t.Errorf("Add was incorrect, obtained: %v, expected: %v.", output, e.expectedOutput)  
  }  
}
```

At first, the implementation may look a bit more complicate. However, it's easier to add new examples as we would only need to insert new entries in the `examples` variable.

### But, How Many Examples Do We Need?

We can partition the set of valid inputs and outputs into regions of interest and sample these regions to get the elements that will compose our examples. Within these regions, we try to build a collection of samples (i.e. examples) that is representative enough for our goals.

Of course, we aim to achieve a reasonable coverage (whatever that means to your project) and be confident that our code does what it should, or perhaps even more critical, that it does not do what it should not. Further, we want to test against edge cases, or rather the _known_ edge cases that we are aware of.

The immediate question is then:

> How many examples do we need?

Maybe 1, 2, 10, 100, 1000, more? I firmly believe that our previous tests didn't have enough examples to give us satisfactory coverage. At least, I am not happy with that.

Moreover, we have to keep some compromises in mind. Presumably:

  1. Adding loads of examples might be tedious or even infeasible.
  2. We may not know all the edge cases beforehand.

I found the second point very interesting because sometimes it happens that we may not be completely aware of all possible edge cases that can emerge from the complex interactions within our systems. Possibly, due to legacy code, essential and/or accidental complexities, non-localized behaviours, etc. If that wasn't true, then perhaps bugs would show up less often than they do.

## Property-based Testing

Disclaimer:

> Property-based Testing is a complementary approach to other testing approaches. It does not replace them, it rather collaborates with them.

Property-based testing is a technique of writing tests popularized by the Haskell library [QuickCheck](https://hackage.haskell.org/package/QuickCheck).

Many other programming languages also have, more or less, similar libraries, for instance:

* [Go](https://golang.org/pkg/testing/quick/).
* [Java](https://jqwik.net/).
* [Scala](https://www.scalacheck.org/).
* [C++](https://github.com/emil-e/rapidcheck).
* [Swift](https://github.com/typelift/SwiftCheck).
* [Rust](https://github.com/BurntSushi/quickcheck).

> Disclaimer: I haven't used all of these libraries and therefore I cannot and will not give more details about them.

Property-based testing is different from the more traditional approaches and hence might need us to shift our mindset a little. Rather than think in terms of specific examples (points), we focus on abstract properties defined by our module (transformations) that shall hold for all possible valid inputs, e.g.:

  * Under what **preconditions** should a module (function, object, module, etc) lead to a given **postcondition**?
  * What are the **invariants** that should be preserved?

We now have to define statements, i.e. properties, that must be true not for only **some** specific examples, but rather for **all** examples that we can ever come up with.

Properties are usually more concerned with the general principles governing the behaviour of the code, enforcing discipline on abstractions that must obey a set of rules.

That all said, a property is a predicate (evaluates to true or false), which must hold (be true) for all elements within a given set of examples:

> For all _x_ _in_ _X_, the predicate _p: X - > Bool_ evaluate be true.

As examples of properties, we have:

  * Given a sorting algorithm: for all non-empty lists, if we sort them ascending order, then we should have the least element at the 1st position.
  * Given a [Functor](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html): for all parameters of the functions _f_ and _g_, if we map the composition of _f_ and _g_, then we should end up with the same result as composing the individual mappings: _map(f) ∘ map(g) == map(f ∘ g)_.
  * Given a layout manager: for all view models within a UI, if we apply the layout manager, they should not see sub-views overlapping.
  * Given two implementations of the same algorithm: for all possible inputs, if we call both algorithms with the same input, then we should obtain the same outcome.

### Benefits of Property-based Testing

Once we have a property, we don't have to write the examples ourselves. Instead, we let the computer in charge of generating as many pseudo-random examples as we want and check our properties on our behalf.

Each example will attempt to falsify the property. If it cannot find an example where the property is false, then we say that the test has succeeded. Otherwise, if it found at least one example for which the property is false, then the test has failed.

Among the benefits of property-based testing, we can list:

  1. It can generate many more examples than we would likely have done manually.
  2. Theoretically, it can generate examples that cover all possible combinations of inputs.
  3. It may generate examples for edge cases that we hadn't even thought about.

The fourth item is particularly nice. As we said before, we may not know all edge cases in advance.

Property-based testing relies on the power of computers to generate as many examples as possible, thus enlarging the input space covered by the test and introducing pseudo-random variability on the automatically generated inputs.

Furthermore, property-based testing libraries commonly ship with a killer feature: _shrinkage_.

#### Shrinkage

Roughly speaking, shrinkage is the ability to reduce an example that falsifies a property to its minimum (i.e. simplest) instance that still falsifies the property. The motivation is that, the smaller an input is, the simpler it should be to reason about, reproduce, debug, and then fix the bug. 

As an example, say we want to test if a property holds for lists. After writing a property-based test, we then found that our property does not hold for a list with 10000 elements. At first, the library would have reported this 10000-elements list to us and then we would have to debug our implementation to understand the reason why it failed for this particular example, which can be painful as the list is reasonably big.

Perhaps the same property would also have failed for smaller lists. Maybe a list with 100, 10, or even 1 element would have been enough to falsify the property.

It would be more much more convenient to start our debugging session with this 1-element list, instead of the one with 10000 elements that we initially obtained.

That's the motivation for shrinkage in a nutshell.

> Instead of eagerly reporting the failed example to the user, the library tries its best to reduce it to the smallest possible instance for which the property still does not hold.

Shrinkage essentially filters the noise out of the signal of interest, leaving us with a better example, which is hopefully easier to reason about.

### Property-based Testing in Go

We are going to use Go, which ships the simple [testing/quick](https://golang.org/pkg/testing/quick/) in the standard library.

>`testing/quick` has reached a feature-frozen status. Moreover, it has quite some limitations (e.g. lack of shrinkage). Yet, it's quite simple and easy to use, hence enough to make the point that I want to make here. [Gopter](https://github.com/leanovate/gopter) seems to be a good alternative, but I won't be discussing it in this post.

Before writing the code, we have to derive some properties that we want to verify against our `Add` function.

Given the addition of integers forms a well-known Algebraic structure, it must obey some well-known rules of Maths.

In particular, given _a_, _b_, _c_ in _X_, we might come up with the following properties:

>  **Identity element**:
>
>  _a + 0 == 0 + a == a_
> ⟹
>  _Add(a, 0) == Add(0, a) == a_


>  **Associativity**:
>
>  _(a + b) + c == a + (b + c)_
>  ⟹
>  _Add(a, (Add(b, c)) == Add(Add(a, b), c)_

Translating these statements into Go code, we have the following properties represented by the predicates `identityElement` and `associativity`:

```go
identityElement := func(a int) bool {  
  left := Add(a, 0)  
  right := Add(0, a)  
  return left == a && right == a && left == right  
}  
  
associativity := func(a, b, c int) bool {  
  return Add(a, Add(b, c)) == Add(Add(a, b), c)  
}
```

Equipped with these predicates, we invoke `quick.Check(prop, config)` to generate a bunch of examples for us. The `config` parameter allows us to customize the library (e.g, the number of generated examples). However, we aren't using this parameter in the following snippet, but passing `nil` and accepting the defaults:

```go
if err := quick.Check(identityElement, nil); err != nil {  
  t.Errorf("identity element failed: %v", err)  
}  
  
if err := quick.Check(associativity, nil); err != nil {  
  t.Errorf("associativity failed: %v", err)  
}
```

If `err` is different from `nil`, then the property has been falsified and therefore the test failed.

The full code:

<script src="https://gist.github.com/rvarago/ac12b55ca227064bb4ae07ca89abdbf6.js"></script>

## Symmetrical Functions and User-Defined Types

Property-based testing also comes in handy when we deal with "symmetrical functions", where one function takes us from **A** to **B**, while the second function reverses the process, taking us from **B** back to **A**.

For example, given a `User u`, and the symmetrical functions:

  *  `encode`: encodes the `User` into some different, yet lossless, representation.
  *  `decode`: Reverses the previous `encode` operation, again lossless.

Then we probably want following property to be satisfied:

> decode(encode(u)) == u

That is quite a powerful statement!

Think of serialization pipelines (CSV, etc), API <-> domain objects, etc. Whenever we have "mirrored" operations, we might have a "trivial" property and therefore the opportunity to profit from property-based testing.

To be able to generate pseudo-random instances of our custom-type `User`, we must provide an instance of an **arbitrary** for `User`.

An instance of an arbitrary knows how to construct composite objects from their components, which themselves provide instances of arbitrary.

In `testing/quick`, we do this by implementing the interface `Generate`, which has the following shape:
        
```go
Generate(r *rand.Rand, size int) reflect.Value
```

Where `r` is a pseudo-random number generator, and `size` is the size of the example to be generated.

As an example, let's provide an instance for a `User` type with `Id` and `Name`. To make things more interesting, let's also restrict generated names to the alphanumeric characters defined by `alphabet`.

```go    
type User struct {  
  Id int  
  Name string  
}  
  
func (User) Generate(r *rand.Rand, size int) reflect.Value {  
  const alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz01233456789"  
  
  var buffer bytes.Buffer  
  for i := 0; i < size; i++ {  
    index := rand.Intn(len(alphabet))  
    buffer.WriteString(string(alphabet[index]))  
  }

  u := User {  
    Id: rand.Int(),  
    Name: buffer.String(),  
  }  
  
  return reflect.ValueOf(u)  
}
```

With this implementation of `Generate` for `User`, the library now can generate instances of `User` for us!

Finally, a toy implementation of `encode` could serialize a `User` into a pipe-separated format. Whereas `decode` would reverse the operation, with a simple error-handling for the `Id`, which we expect to be a valid integer:

```go    
func encode(u *User) string {  
    return fmt.Sprintf("%v|%v", u.Id, u.Name)  
  }  
  
func decode(s string) (*User, error) {  
    fields := strings.Split(s, "|")  
  
    id, err := strconv.Atoi(fields[0])  
  
    if err != nil {  
      return nil, fmt.Errorf("failed to parse field Id: %v", err)  
    }  
  
    name := fields[1]  
  
    u := &User {  
      Id: id,  
      Name: name,  
    }  
  
    return u, nil  
}
```

We can now check the symmetrical property between `encode` and `decode` with the following test:

```go    
func TestEncodeDecode(t *testing.T) {  
    symmetrical := func(u User) bool {  
      roundTripUser, err := decode(encode( &u))  
      return err == nil && *roundTripUser == u  
    }  
  
    if err := quick.Check(symmetrical, nil); err != nil {  
      t.Errorf("symmetrical encode -> decode fail: %v", err)  
    }  
}
```

`symmetrical` is our property, and it checks whether `decode` correctly reverses `encode`, returning a `User` that is equal to one that we had provided to `encode`. Further, no error should occur during the process.

If we had accidentally changed the separator used by `decode` _without_ properly updating its counterpart in `encode`, then the test would have failed and reported the failure so that we could fix it.

## Conclusion

Property-based testing can help us catching bugs for cases that we hadn't even imaged could ever be possible.

It's particularly useful when comparing two models of the same concept. Say two implementations of the same feature, but one is better written or faster than the other, and we want to swap them. However, we only want to swap the implementation once we are confident that they always yield the same result for all valid inputs. We might come up with the following property to assist us:

> for all **input, algorithmA(input) == algorithmB(input)**

By letting the library generate a bunch of examples, we may be confident enough to proceed with our swapping.

Property-based testing encourages us to think in terms of the general behaviours that our program should exhibit. Ultimately, this means that we have to make pre-conditions, post-conditions and invariants more explicit, which most likely lead to better API designs.

The technique could be made even more effective to catch bugs when combined with suitable strategies for type design, e.g, [Algebraic Data Types]({{ site.baseurl }}{% link _posts/2019-12-19-algebraic-data-types-and-data-modelling.md %}).

Shrinkage is quite a powerful concept as it can decrease the time between a test failure report and its proper fix.

As I said before, I don't believe property-based testing replaces other methods of unit-testing, it rather complements them.

Furthermore, property-based testing libraries cannot prove that a property is, in fact, correct. They just try very hard to find examples to falsify it.

> From my perspective, it's far better to have different approaches combined and thus increase our whole safety-net and reduce the chances of bugs slipping into production. 

Lastly, I must say that it is not always straightforward to specify properties. That takes time, practice, and it might be trickier in some cases than in others.
Nevertheless, I certainly encourage you to give property-based testing a go. Even if you end up not using it within all your projects, it will at least show you new ways to look at your code and hopefully be a new tool in your toolbox to pull in whenever wanted.

## References

[1] [John Hughes -- Experiences with QuickCheck: Testing the Hard Stuff and Staying Sane](https://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quviq-testing.pdf).

[2] [John Hughes -- Don't Write Tests](https://www.youtube.com/watch?v=hXnS_Xjwk2Y).

[3] [Go's Quick](https://golang.org/pkg/testing/quick/).

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
