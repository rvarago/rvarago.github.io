---
layout:	"post"
title:	"Property-based Testing in Golang"
---

* * *

![](/assets/img/2020-03-12-propertybased-testing-in-golang_0.png)

An enterprise Gopher with a tie and long beard ready to fight bugs alongside
you.
[Gopherize.me](https://gopherize.me/gopher/1695ff01da3dda465e7874f291c3a1e6554a2f90)

> Testing is an important aspect of software development. Let's see how
Property-based Testing can help us to test our Go programs.

A few weeks ago, I gave a talk about Property-based Testing using Golang on
our awesome Engineering Summit 2020 @ eGym, which I tidied up, included more
details and now I want to share with you in this blog post.

* * *

Programming is an activity carried out by humans, and hence prone to mistakes.
Mistakes can happen and sporadically they do.

However, as professionals, we have techniques, practices, and tools that help
us to catch mistakes as early as possible and therefore reduce the likelihood
of deploying them to production. We have: code review, pair programming, a
myriad of programming languages, type-checkers, linters, static analyzers,
sanitizers, mathematical proofs, CI, automated testing (how about TDD? 😉) -
just to name a few.

In this post, we're going to briefly talk about tests, particularly property-
based Testing, which is a way of writing tests based on general governing
principles that our code should uphold for all valid inputs that it can
receive.

To put it in perspective, let's first quickly glance over other possible
approaches to writing unit tests.

### Unit Tests

We commonly write unit tests to increase our confidence that our unit-under-
test correctly fulfils its designed specification.

They provide fast feedback that quickly tells us whether we have committed a
mistake, so we have a chance to readily fix it before it propagates to later
stages of our development pipeline, risking popping up at the customer site.

A well-written suite of unit tests also serves as live documentation that
exposes how the code should behave or, at least, how we expect it to behave.

Commonly, we write unit tests by listing particular examples of inputs that
are fed into our code-under-test (function, object, module, etc), and then we
assert that the obtained output matches what we were expecting.

More precisely, it says:

> Given an input **_x_** _in_ ** _X_** , expected output **_y_** _in_ ** _Y_**
, and a function to be tested **_f: X - > Y_**:

> Assert that **_f(x) == y_**

To illustrate the idea, let's look at the contrived, yet illustrative, Go
function that we want to test. Its purpose is to simply add two integers
together and then return the integer result:

    
    
    func Add(x, y int) int {   
      return x + y  
    }

#### Example-based Testing

We may manually write examples of the form { _input_ , _expectedOutput_ }, and
then assert that _expectedOutput_ equals to obtained _output_ :

    
    
    expectedOutput := 1  
      
    if output := Add(0, 1); output != expectedOutput {  
      t.Errorf("Add fail, obtained: %v, expected: %v.", output, expectedOutput)  
    }

#### Table-driven Testing

Perhaps we could factor out the examples into a table to separate the examples
from the code that exercises them. Thus, making it easier to add new examples:

    
    
    type InputPair struct {  
      left int  
      right int  
    }  
      
    type Example struct {  
      input InputPair  
      expectedOutput int  
    }  
      
    examples := []Example {  
      {InputPair{0, 0}, 0},  
      {InputPair{1, 0}, 1},  
    }  
      
    for _, e := range examples {  
      if output := Add(e.input.left, e.input.right); output != e.expectedOutput {  
         t.Errorf("Add was incorrect, obtained: %v, expected: %v.", output, e.expectedOutput)  
      }  
    }

#### But, how many examples do we need?

When defining our examples, we can partition the set of valid inputs and
outputs into regions of interest and sample these regions to get the elements
that will compose our examples. With these regions, we try to build a
collection of samples (i.e. examples) that is representative enough for our
goals.

Of course, we aim to achieve a reasonable coverage (whatever that means to
your project) and test against edge cases, or the _known_ edge cases that we
are aware of, to be more precise.

But the question that arises is: how many examples do we need?

Maybe 1, 2, 10, 100, 1000, more? I firmly believe that our previous tests
didn't have enough examples to give us satisfactory coverage, it's far from
that 😅.

Moreover, we have to keep some compromises in mind, presumably:

  1. Adding loads of examples might be tedious or even infeasible
  2. We may not know all the edge cases beforehand

I found the second point very interesting because sometimes it happens that we
may not be completely aware of all possible edge cases that can emerge from
the complex interactions within our systems. Possibly, due to legacy code,
essential and/or accidental complexities, non-localized behaviours, etc. If
that wasn't true, then perhaps bugs would show up less frequently than they
do.

### Property-based Testing

Disclaimer:

> Property-based Testing is a complementary approach to the other methods
previously seen. It does not substitute them, it rather collaborates with
them.

Property-based Testing is a technique of writing tests that originated in the
Haskell library [QuickCheck](https://hackage.haskell.org/package/QuickCheck).
And there are also libraries available for several other programming
languages, for instance, [Go](https://golang.org/pkg/testing/quick/),
[Java](https://jqwik.net/), [Scala](https://www.scalacheck.org/),
[C++](https://github.com/emil-e/rapidcheck),
[Swift](https://github.com/typelift/SwiftCheck),
[Rust,](https://github.com/BurntSushi/quickcheck) etc. However, I haven't used
all these libraries, so I cannot give more details about them.

To benefit from Property-based Testing, we have to shift a little bit our
point of view on tests, and instead of thinking in terms of specific examples,
we focus on abstract properties of our code that shall hold for all valid
inputs, e.g.:

  * Under what _preconditions_ should an abstraction (function, object, module, etc) lead to a given _postcondition_?
  * What are the _invariants_ that should be preserved?

We now have to define statements, i.e. properties, that must be true not for
only **some** specific examples, but rather for **all** of them.

Properties are usually more concerned about the principles that govern the
general behaviour of the code, it enforces discipline on abstractions that
must obey a set of rules.

That being said, a property is a predicate, i.e. evaluates to true or false,
that must hold for all elements within a given set of examples:

> For all **_x_** _in_ ** _X_** , the predicate **_p: X - > Bool_** must be
**true**

Some examples of properties are:

  * After sorting a non-empty list in ascending order, the minimum element is at the 1st position
  * For a [Functor](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html), mapping the composition of functions is the same as composing the individual mappings
  * For all view models, the sub-views within a UI should not overlap
  * In a state machine, going from state _A_ to state _B_ should trigger a given side-effect
  * Two implementations of the same algorithm (perhaps after a refactoring, optimization, etc) should always manifest the same externally observable effect for all valid inputs

#### What does it buy us?

Once we have a property, we don't have to write the examples ourselves.
Instead, we let the library in charge of generating as many pseudo-random
examples as we want.

Each example will attempt to falsify the property, in which case the whole
test fails.

Among the benefits of Property-based Testing, we can list:

  1. It can generate many more examples than we would have normally done manually
  2. Theoretically, it can generate examples that cover all possible combinations of inputs
  3. It's particularly useful to compare different implementations of the same algorithm
  4. It may generate examples that find edge cases that we didn't even know about

The fourth item is nice as we had previously said that we may not know all
edge cases in advance. And Property-based Testing might increase the
likelihood of observing them by leveraging the power of computers to enlarge
the input space that is covered and introduce pseudo-random variability on its
generated inputs.

Furthermore, Property-based Testing libraries usually ship with a powerful
capability named _shrinkage_.

Roughly speaking, shrinkage is the ability to reduce an example that fails a
property to its minimum (or simplest) instance that still fails the property.

As an example, we might test whether a property holds for lists, and we found
out that, for a list with size 10000, the property fails.

At first, the library would have reported this 10000-elements list to us and
then we would have to debug the code to understand the reason why it failed
for this particular example, which can be painful as the list is fairly large.

But perhaps the same property would also have failed for a smaller list, maybe
100, 10, or even 1 element would have been enough to falsify the property.

It would be more convenient to start our debugging from this 1-element list,
instead of the one with 10000 elements that we initially had.

That's the basic idea behind shrinkage.

Instead of eagerly reporting the failed example to the user, the library tries
its best to reduce the example to the smallest possible instance that also
fails the property. And only then it reports this such an instance. It
essentially filters the noise out, leaving us with an example with a better
signal-to-noise ratio, which is hopefully easier to reason about and therefore
fix it.

#### How can we do it in Go?

We are going to use Go, which ships with the library
[**_testing/quick_**](https://golang.org/pkg/testing/quick/) ** _._** This
library has reached a feature-frozen status and it has its limitations; for
example, it doesn 't support shrinkage. Yet, it's quite simple and easy to
use, being enough to make a point. As an alternative, there is a library
called [Gopter,](https://github.com/leanovate/gopter) which we won't be
covering in this article.

Before writing the code, we have to define the properties to check against our
_Add_ function.

Since the addition of integers forms a well-known Algebraic structure, it has
to obey some well-known rules too.

In particular, given **_a_** , **_b_** , **_c_** in **_X_** , some possible
properties that must hold are:

>  ** _Identity element_** :

>  _a + 0 == 0 + a == a_

>  _= > Add(a, 0) == Add(0, a) == a_

>  ** _Associativity_** :

>  _(a + b) + c == a + (b + c)_

>  _= > Add(a, (Add(b, c)) == Add(Add(a, b), c)_

Translating these statements to Go code, we have the following properties:

    
    
    identityElement := func(a int) bool {  
      left := Add(a, 0)  
      right := Add(0, a)  
      return left == a && right == a && left == right  
    }  
      
    associativity := func(a, b, c int) bool {  
      return Add(a, Add(b, c)) == Add(Add(a, b), c)  
    }

Finally, we can use the function _quick.Check(prop, config)_ to generate the
examples for us.

The _config_ parameter allows us to customize the library, for instance, to
configure the number of examples to be generated. But we aren 't using this
parameter in the following snippet, instead, we are passing _nil_ to use the
default values:

    
    
    if err := quick.Check(identityElement, nil); err != nil {  
      t.Errorf("identity element fail: %v", err)  
    }  
      
    if err := quick.Check(associativity, nil); err != nil {  
      t.Errorf("associativity fail: %v", err)  
    }

And that's it, if _err_ isn 't equal to _nil_ , then the property is falsified
and hence the test fails.

The full code is:

#### Symmetrical functions and user-defined types

Property-based Testing also comes in handy when we are dealing with
"symmetrical functions", where one function takes us from **A** to **B** ,
while a second function inverts the process, taking us from **B** back to
**A**.

For example, given a _User u_ , and the symmetrical functions:

  *  ** _encode_** that encodes a _User_ into a given representation
  *  ** _decode_** that reverses the previous **_encode_** operation

Then we have that the following property shall hold:

> decode(encode(u)) == u

This is quite a powerful statement! Think of serialization pipelines (CSV,
etc), API <-> domain objects, etc. Whenever we have "mirrored" operations, we
might have an opportunity to profit from Property-Based Testing.

To enable the library to generate instances of our type _User_ , the type
needs to provide an instance of an arbitrary that knows how to construct
composite objects from its components that themselves provide instances of
arbitrary.

In _testing/quick_ this is done by implementing the interface _Generate_ **
__** that has the following signature:

    
    
    Generate(r *rand.Rand, size int) reflect.Value

Where _r_ is a pseudo-random numbers generator, and _size_ is the size of the
example to be generated.

To exemplify its usage, let's provide a possible instance for a small _User_
type that is __ composed of _Id_ and _Name_ , with __ the additional
complication that we are only interested in names formed by alphanumeric
characters, which can be done by sampling the _alphabet_ string:

    
    
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

Now the library knows how to generate instances of _User_ s for us 😺.

A toy implementation of _encode_ could serialize the _User_ into a pipe-
separated format. And _decode_ reverses the operation, with a simple error
handling for the _Id_ which is expected to be a valid integer:

    
    
    func encode(u *User) string {  
       return fmt.Sprintf("%v|%v", u.Id, u.Name)  
     }  
      
    func decode(s string) (*User, error) {  
       fields := strings.Split(s, "|")  
      
       id, err := strconv.Atoi(fields[0])  
      
       if err != nil {  
          return nil, fmt.Errorf("failed to parse field Id: %v", err)  
       }  
      
       name := fields[1]  
      
       u := &User {  
          Id: id,  
          Name: name,  
       }  
      
       return u, nil  
    }

With this, we can test the symmetrical property between _encode_ and _decode_
via the following property-based test:

    
    
    func TestEncodeDecode(t *testing.T) {  
       symmetrical := func(u User) bool {  
          roundTripUser, err := decode(encode( &u))  
          return err == nil && *roundTripUser == u  
       }  
      
       if err := quick.Check(symmetrical, nil); err != nil {  
          t.Errorf("symmetrical encode -> decode fail: %v", err)  
       }  
    }

 _symmetrical_ is our property and it checks whether _decode_ correctly
reverses _encode_ to give us back a _User_ equal to one that we initially
provided to _encode_ , and no error occurs during the process.

If we had accidentally changed the separator in _decode  _without properly
updating its counterpart in  _encode_ , maybe by using  _;_ instead of _|_ ,
then the test would have failed and reported the failure to us, so we could
promptly fix it.

### Conclusions

Property-based Testing can help us to catch bugs in cases we didn't even think
about or knew about their existence.

It's particularly helpful to compare two models of the same concept. Say we
have two implementations of the same feature, perhaps one is better written or
faster than the other, and we want to make sure that both give the same result
for all valid inputs, otherwise the behaviour would have been accidentally
changed. A direct property might then be:

> for all **input, algorithmA(input) == algorithmB(input)**

Property-based Testing encourages us to think in terms of abstract behaviours
of our code, which potentially could lead to better API designs as we make
preconditions, postconditions, and invariants explicitly defined.

It can be even more effective to catch bugs when combined with suitable type
design techniques, for instance, [Algebraic Data Types](https://code.egym.de/a
-brief-introduction-to-the-algebra-of-types-df92f0820e5).

As I said before, I don't believe it replaces the other methods of unit
testing, it rather complements them. Perhaps it's nicer to have a combination
of all the approaches to increase our safety-net and reduce the chances of
introducing bugs. Furthermore, Property-based Testing libraries cannot even
prove that a property is, in fact, correct; it simply tries to find examples
to falsify it.

The ability of shrinkage is quite powerful as it can filter the noise out from
a failed example and so give us a simpler instance that is easier to
reproduce, reason about, and debug.

I must say that it is not always straightforward to specify properties, it
takes time, practice, and it might be trickier in some cases than in others.
But I encourage you to give it try, even if you end up not using it for all
projects, it would show you new ways to look at your code and become a new
tool in your toolbox to pull when needed, nonetheless.

### References

  * [John Hughes -- Experiences with QuickCheck: Testing the Hard Stuff and Staying Sane](https://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quviq-testing.pdf)
  * [John Hughes -- Don't Write Tests](https://www.youtube.com/watch?v=hXnS_Xjwk2Y)
  * <https://golang.org/pkg/testing/quick/>


***
*Originally published at https://medium.com/@rvarago*
