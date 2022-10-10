---
layout: post
title: "Pattern-Matching without a Match"
tags: java ocaml plt design-pattern visitor
---

> Emulating pattern-matching in a language without native support for it.

---

| ![Match](/assets/img/2022-10-12-emulating-pattern-match.png) |
| :----------------------------------------------------------: |
| _Match as a function (a + b) -> (a -> r) -> (b -> r) -> r._  |

---

Pattern-matching, also known as case-analysis, allows us to deconstruct terms into their components with a syntax (and semantic) suitable for data-oriented queries and/or define algorithms over types of different yet related shapes.

That eases the navigation and processing of data structures. Particularly shining when we want to apply an open set of functions on an [Algebraic Data Type]({{ site.baseurl }}{% link _posts/2019-12-19-algebraic-data-types-and-data-modelling.md %}) with a closed set of immutable variants of a known schema, henceforth separating the data from the behaviour we may later (in time and/or space) associate with it. This comes as a subtle, yet fundamental, contrast with objects. In terms of objects, there's usually a closed set of _virtual_ functions operating on an open set of types, typically dispatched through an interface; henceforth consolidating data and its associated behaviour as objects. Much of this contrast is nicely summarized in the so-called [Expression Problem](https://en.wikipedia.org/wiki/Expression_problem).

Pattern-matching is commonly reified as a language feature in programming languages that encourage the Functional style of programming (e.g. Haskell, Scala, OCaml), even though it's oftentimes feasible to emulate it in Imperative languages (e.g. C++, Java, C#) with the Visitor pattern or similar encoding. Nevertheless, given the benefits of having native pattern-matching constructs, we've been seeing some imperative languages pushing to include pattern-matching capabilities in their set of features. For example, there's Rust and perhaps [Java](https://openjdk.org/jeps/405) or [C++](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1371r1.pdf) in the future.

> Actually, Java has had some narrow [support](https://www.artima.com/weblogs/viewpost.jsp?thread=168839) for pattern-matching under the covers of `catch` clauses where we match against a set of exceptions and react upon a successful match depending on the surrounding context.

Even though support for native pattern-matching streamlines a tighter integration with the rest of the programming language and thus may offer a more comprehensive and ergonomic experience with higher-levels of expressiveness, sometimes we may get away with an appropriate encoding whereby we emulate pattern-matching in language where it lacks (or maybe only before the feature has landed). Today, I shall briefly present one such encoding with the Visitor pattern expressed in the Java programming language. And, as a bonus, we shall also see the [Scott encoding](https://en.wikipedia.org/wiki/Mogensen%E2%80%93Scott_encoding) of data types from which we gain support for pattern-matching in a perhaps surprising form.

> IMPORTANT: Throughout this exposition, we must keep in mind that this is an exercise only meant for learning and not necessarily building on idiomatic (or efficient, battle-tested, production-ready) uses of Java; or maybe, at least not to this date.

# Pattern-Matching Algebraic Data Types

For illustrative purposes only, let's entertain ourselves with a rather contrived use-case for which we shall come up with solutions in Java (OpenJDK version "17.0.4" 2022-07-19) in terms of pattern-matching:

> Assume for a moment that we want to model a singly-linked list as an inductively defined Algebraic Data Type (defined by cases) and then write a couple of external ("free-standing", **not** attached to a class/object as methods) functions over it. For instance, given a list of integers we want to compute the sum of its elements (or maybe we could map the elements with a function, of filter elements, or compute its length, etc; but for brevity we shall only implement the summation).

In terms of a simplified algebraic notation, we may describe our list `L` as:

```
L(a) = 1 + a * L(a)
```

> This reads as a singly-linked list `L` of homogeneous elements of type `a` is either (`+`, also known as variant, sum, or co-product) empty (`1` also known as unit) or it's composed of the pair (`*` also known as product) of a term of type `a` (also known as head) and the rest of the list `L` (also known as tail) of elements of type `a`.

Or, in a typical ML language such as OCaml:

```ocaml
type 'a list = Nil | Cons of 'a * 'a list
```

Finally, an equivalent (with labelled fields) representation in Java:

```java
// IMPORTANT: This representation is only meant for pedagogical purposes.
// For a production system, we are most certainly better served with a JDK collection.
sealed interface SList<T> {
    record Nil<T> () implements SList<T> {}

    record Cons<T> (T head, SList<T> tail) implements SList<T> {}
}
```

> The `sealed` marker informs the type-checker (and the human reader) that there's a closed, known set of types satisfying the `SList<T>` interface; and equipped with this knowledge, the checker can ensure niceties such as exhaustive coverage, etc.

With the type defined, we move on to actually implementing `sum` within which we pattern-match.

## Native Record Patterns

Our goal is to write the function `sum`:

```
sum : L(integer) -> integer
```

We may implement it in OCaml straightly by pattern-matching on the `list` parameter:

```ocaml
let rec sum list = match list with
  | Nil -> 0
  | Cons (head, tail) -> head + sum tail
```

> IMPORTANT: This, as well as all the other recursive implementations in this text, are not tail-recursive and consequently susceptible to stack-overflows and performance penalties.

Or, with a sugared syntax, a little terser as:

```ocaml
let rec sum = function
  | Nil -> 0
  | Cons (head, tail) -> head + sum tail
```

> The snippet above, albeit shorter, expands to the longer `match .. with ..` we'd seen previously.

Back to Java. As a first attempt, if we assume that we have access to "JEP 405 Record Patterns (Preview)", we may complete the assignment with something like this:

```java
// WARNING: The following snippet has not been type-checked with a real compiler implementing JEP 405
// and therefore might not work and/or it's subject to change before stabilization.
class SListOps {
    static Integer sum(SList<Integer> list) {
        return switch (list) {
            case SList.Nil<Integer>() -> 0;
            case SList.Cons<Integer>(head, tail) -> head + sum(tail);
        };
    }
}
```

Notice the recursive definition of the algorithm. We match on our `list` parameter of type `SList<Integer>` and:

- When we obtain `Nil<Integer>`, this means we've reached the base case; and therefore we map this term to `0`
- When we obtain `Cons<Integer>(head, tail)`, this means we're in the recursive case; and therefore we sum the `head` of the current list with the outcome of recursing over the `tail` (itself a `SList<Integer>`)

> There's a connection between the way we've defined the data structure and the way we've operated on it, where the operation "undoes" what the definition had done. More on this in a future post.

## Visitor Encoding

> I'll be showing just **one** among multiple ways to encode the Visitor pattern.

While we wait for JEP 405 to graduate, or even afterwards, we might meet our goal with the help of the Visitor pattern thereby making up for the missing support for pattern-matching.

We can do that by observing the correspondence between a single function on `1 + a * L(a)` (operating on the variant) and a pair of functions on `1` and `a * L(a)` (operating directly on the components). We shall call such a function accepting the pair as `match` - an alternative name would have been `visit`, but I'm sticking to `match` for the sake of consistency:

```java
<R> R match(Supplier<R> caseNil, BiFunction<T, SList<T>, R> caseCons)
```

- When we match on `Nil<T>()`, then we should invoke `caseNil`
- When we match on `Cons<T>(head, tail)`, then we should apply `caseCons` to `head` and `tail`

In this setup, the next step would be to attach `match` as a method of `SList<T>` and then implement it for `Nil<T>` and `Cons<T>` following the logic previously explained:

```java
sealed interface SList<T> {
    <R> R match(Supplier<R> caseNil, BiFunction<T, SList<T>, R> caseCons);

    record Nil<T> () implements SList<T> {
        @Override
        public <R> R match(Supplier<R> caseNil, BiFunction<T, SList<T>, R> _caseCons) {
            return caseNil.get();
        }
    }

    record Cons<T> (T head, SList<T> tail) implements SList<T> {
        @Override
        public <R> R match(Supplier<R> _caseNil, BiFunction<T, SList<T>, R> caseCons) {
            return caseCons.apply(head, tail);
        }
    }
}
```

Thus, we may now implement `sum` in terms of `match` as:

```java
class SListOps {
    static Integer sum(SList<Integer> list) {
        return list.match(() -> 0, (head, tail) -> head + sum(tail));
    }
}
```

And use it like this:

```java
final var list = new SList.Cons<>(1, new SList.Cons<>(2, new SList.Nil<>()));
System.out.println(SListOps.sum(list)); // prints 3
```

# Scott Encoding

Thus far we have always started with a data structure defined as an Algebraic Data Type and from that, we encoded a way to pattern-match it.

The Scott encoding works a little backwards if taken from that perspective, whereby we identify pattern-matching as an **universal operation** and therefore able to fully encode a data structure; hence we may elide the records entirely and **take the `match` _as_ the data structure itself!**

```java
// WARNING: This does not compile, see below.
@FunctionalInterface
interface SList<T> {
    <R> R match(Supplier<R> caseNil, BiFunction<T, SList<T>, R> caseCons);

    static <T> SList<T> nil() {
        return (caseNil, _caseCons) -> caseNil.get();
    }

    static <T> SList<T> cons(T head, SList<T> tail) {
        return (_caseNil, caseCons) -> caseCons.apply(head, tail);
    }
}
```

No records! Actually, there are no data structures in the conventional sense of the word any longer, only factory methods as opposed to records and closures filling in the role of data containers by capturing `head` and `tail`.

Unfortunately, as stated in the comment, at the moment this snippet does not compile. This happens because Java forbids lambda expressions from implementing generic interface **methods** (when the method introduces type parameters). However, we can work around it by expanding the sugared syntax for lambdas into anonymous inner classes:

```java
interface SList<T> {
    <R> R match(Supplier<R> caseNil, BiFunction<T, SList<T>, R> caseCons);

    static <T> SList<T> nil() {
        return new SList<>() {
            @Override
            public <R> R match(Supplier<R> caseNil, BiFunction<T, SList<T>, R> _caseCons) {
                return caseNil.get();
            }
        };
    }

    static <T> SList<T> cons(T head, SList<T> tail) {
        return new SList<>() {
            @Override
            public <R> R match(Supplier<R> _caseNil, BiFunction<T, SList<T>, R> caseCons) {
                return caseCons.apply(head, tail);
            }
        };
    }
}
```

That's a bit more verbose, alas it compiles!

With the working syntax ready, nothing changes in the `sum` function. The only step left is to patch the call-site to use the new factories instead of instantiating records:

```java
final var list = SList.cons(1, SList.cons(2, SList.nil()));
System.out.println(SListOps.sum(list));  // prints 3
```

# Conclusion

We have seen how we may emulate pattern-matching in a language without native support for it. Firstly with the Visitor pattern and then we even "elided" the whole classical data structures by representing them directly as a pattern-match with the Scott encoding of data types. Parallel to that, we had the chance of exploring a popular recursive algorithm and expressed the same concept with alternative notations whilst also obtained familiarity with different ways to encode the same solution and a glimpse of what might become a new feature of Java.

To be expressible, the Scott encoding imposes some requirements on the host language. For instance, if the language is statically-typed, then such a type-system must admit lambdas closing over lexical scopes (lexical closures) and the ability to define algebraic data types inductively (recursive data types); and Java (among a couple of other programming languages) satisfies both criteria. There exists alternative encodings with different requirements and pros/cons, such as the Boehm-Berarducci encoding.

As hinted before, emulating an otherwise natively supported language feature comes with its own trade-offs. Namely, we don't get all the expressiveness capabilities (e.g. nested patterns, guard clauses, alias), ergonomics and synergy with the rest of the language; not to mention that we end up writing more code, possibly repeating ourselves across types and projects. On the other hand, to a certain extent that can us get us going should we need to chase this path forward for whatever reason.

Moreover, throughout this discussion, we have purposefully focused solely on abstract notions and ignored critical aspects of real-world systems, such as resource-efficiency demanded by a solution. As an example, even so we can apply the Scott encoding to model a data structure as a pattern-match, this would offer far different performance characteristics with regard to runtime when compared against a memory-efficient data structure optimized for real access patterns and digital computers.

Lastly, it's worthwhile to emphasize that not because we can apply a technique this means that we must or even should. In reality, different scenarios generally require different solutions where we should consider all relevant factors such as surrounding language facilities and their idiomatic usage, supporting tooling, the larger ecosystem, familiarity and preference of the team, existing code-base, etc.

Personally, I wouldn't recommend applying _all_ techniques previously described without due consideration and taking the context into account (e.g. sometimes dispatching through an abstract interface may be more appropriate than matching on a concrete type). Part of the goal of this post leans more on the curiosity/trivia side of professional programming, also serving as an excuse for us to explore a variety of interesting topics where some of them inhabit the more theoretical areas of Computing Science.

All in all, I hope you've enjoyed reading this post as much as I enjoyed writing it!

# References

[1] [Algebraic Data Type]({{ site.baseurl }}{% link _posts/2019-12-19-algebraic-data-types-and-data-modelling.md %})

[2] [Expression Problem C2](https://wiki.c2.com/?ExpressionProblem)

[3] [JEP 405: Record Patterns (Preview)](https://openjdk.org/jeps/405)

[4] [Scott Encoding Stanford](https://crypto.stanford.edu/~blynn/compiler/scott.html)

[5] [Scott encoding in Java](https://matthias.benkard.de/posts/1350)

[6] [Beyond Church encoding: Boehm-Berarducci isomorphism of algebraic data types and polymorphic lambda-terms](https://okmij.org/ftp/tagless-final/course/Boehm-Berarducci.html)
