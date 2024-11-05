---
layout: post
title: "Haskell, the little things (1 of N) - where clauses"
tags: haskell
---

> This is the (1 of N) installment of a series of short posts where I list some "smaller" features of Haskell that I've found neat and valuable.

---

## Introduction

Haskell is a remarkable functional programming language. It's also well-known for pushing the boundaries of "what programming languages can be" by offering a mix of features not typically seen in other languages, such as Type-Families, GADTs, and Linear Types.

However, Haskell also equips developers with loads of, perhaps "smaller", tools that we may take for granted and thus never explicitly think of them as such, and yet they are usually present in our code. This series aims to list some of those "smaller" features that I find particularly helpful.

Today, we'll talk about `where` clauses and their support for writing functions scoped within functions.

> Disclaimer: This is a series based on my personal, subjective preferences.
> Your mileage may vary.

## Where clauses

As a functional language, it's no surprise that Haskell has a syntax that makes it easy to write functions. It's not uncommon to have modules composed of _a lot_ of functions, typically small ones, perhaps just one line long, that work together via composition to solve complex problems.

Sometimes we want to break down a high-level function `f` into smaller low-level functions `g, h, ...`, but perhaps these low-level functions aren't interesting enough to deserve standing out in the module.

Maybe because they're tightly specific to `f` only, or too short, or ambiguous with other low-level functions that would emerge from other outer-functions within the module, etc.

Thus, we want to let these low-level functions available only to `f`. So, we will make their scopes as small as possible, and that is **within the scope of `f` itself**.

> From now on, we refer to such low-level functions as "local functions" (inner, contained, child), as in "local with regard to another (outer, containing, parent) function".

With local functions, we:

- keep the reader's attention on the high-level algorithm that we deem relevant and not overwhelmed with details.
- avoid duplicating knowledge that needs to stay in sync.
- hide identifiers from an outer scope where they might not make sense.
- retain locality by keeping the local function closer to the call site.
- may opt-in for shorter names, as the point of definition and point of use are close.

We can write local functions with`let` expression or `where` clauses. And each construct has its differences and characteristics. But in this post, we will only talk about `where`, by showing three scenarios in which it might help.

### Threading the same arguments through recursive calls

Consider an implementation of `map`:

```haskell
map' :: (a -> b) -> [a] -> [b]
map' _ [] = []
map' f (x : xs) = f x : map' f xs
```

We threaded `f` through the recursive call `map' f xs`. However, `f` doesn't change across recursive invocations to `map'`.

Perhaps we prefer not to _explicitly_ pass it. We can write a `go` helper that closes over `f` to achieve that:

```haskell
map'' :: (a -> b) -> [a] -> [b]
map'' f = go
  where
    go [] = []
    go (x : xs) = f x : go xs
```

> Here and elsewhere, I named the helper as `go`, but any other name (maybe a more descriptive one?) would have done the trick.

Now, `map''` isn't recursive any longer. Instead, it pushes the recursive step into `go` and this one captures `f` from the outer-scope freeing us from explicitly passing it as an argument of the recursion call `go xs`.

### Making recursive functions tail-call

Consider an implementation of `length`:

```haskell
length' :: [a] -> Integer
length' [] = 0
length' (_ : xs) = 1 + length' xs
```

Perhaps we prefer tail recursion. We can write a `go` helper that increments the accumulator and pass it as a parameter:

```haskell
length'' :: [a] -> Integer
length'' = go 0
  where
    go acc [] = acc
    go acc (_ : xs) = go (1 + acc) xs
```

Now, we accumulate the list's length in the `acc` parameter that we first increment and then pass the incremented value as an argument of the recursive call to `go`.

### Breaking down complex functionality

Say that want to generate a simple identifier for a list of strings that should be insensitive to order. For brevity, let's stick with the format:

> Given the `prefix` and the list of `parameter_1`, ..., `parameter_N`, the identifier shall be `prefix:{parameter_1|...|parameter_N}`.
>
> Example:
> `prefix` <- `device` and `parameter_1` <-`Debian` and `parameter_2` <- `Arch Linux`, the identifier shall be `device:{Arch Linux,Debian}`.

We may implement it as:

```haskell
import Data.List (intercalate, sort)

generateId :: String -> [String] -> String
generateId prefix parameters = prefix <> ":" <> "{" <> intercalate "|" (sort parameters) <> "}"
```

Perhaps we prefer to split up the implementation into smaller pieces arbitrarily as:

```haskell
import Data.List (intercalate, sort)

generateId' :: String -> [String] -> String
generateId' prefix parameters = prefix <> ":" <> suffix
  where
    suffix = "{" <> append parameters <> "}"
    append parameters = intercalate "|" (normalize parameters)
    normalize parameterrs = sort parameters
```

Or -- don't explicitly pass the `parameters` along:

```haskell
import Data.List (intercalate, sort)

generateId'' :: String -> [String] -> String
generateId'' prefix parameters = prefix <> ":" <> suffix
  where
    suffix = "{" <> appended <> "}"
    appended = intercalate "|" normalized
    normalized = sort parameters
```

Or -- move everything into the `where` clause:

```haskell
import Data.List (intercalate, sort)

generateId''' :: String -> [String] -> String
generateId''' prefix parameters = formatted
  where
    formatted = prefix <> ":" <> suffix
    suffix = "{" <> appended <> "}"
    appended = intercalate "|" normalized
    normalized = sort parameters
```

Etc.

> Disclaimer: These are just some of the ways to express the same contrived algorithm. The point is not to cover all styles or decide which style is the best, but rather to illustrate how we may express it with `where` clauses.

So, we broke up the implementation of `generateId` in terms of smaller pieces that helped us to organize the functionality. Yet they might not be suitable to live at the same scope as the `generateId`, but rather within it.

## Conclusion

We've shown three examples of local functions defined with `where` clauses. I'd say that whether to use it or not depends on the case in hand, as sometimes we might be better off with `let` expressions, lambdas, "fully-fledged" functions, unfolding the logic in-place, etc.

I'm particularly into `where` clauses because not only does it allow me to scope helpers into functions and keep definition and usage closer, but also because I can refer (use, call) to the local functions before actually defining them. That helps me to concentrate on the high-level algorithm (policy) and only when I need, do I scroll down to check the details (mechanisms). By that time, I already know how they are used (context) and I suppose that helps me to stay focused and work more effectively -- with less back and forth.

There's more to `where` clauses, such as manually ascribing types to functions defined within it (instead of letting GHC infers them) and how that interacts with `ScopedTypeVariables` or maybe shortening the examples with `LambdaCase`, etc. Nevertheless, those are topics for another day.

# References

[1] [Haskell Wiki - Let vs Where](https://wiki.haskell.org/Let_vs._Where)
