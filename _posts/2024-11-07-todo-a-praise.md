---
layout: post
title: todo!(), a praise
permalink: /:title/
tags: [rust]
---

> The `todo!` is a humble Rust macro that I often use when envisioning
> the "shape" of programs.

------------------------------------------------------------------------

When I break a complex problem into smaller parts in the hope that each
small problem admits a simple (as in ["single
fold"](https://www.youtube.com/watch?v=LKtk3HCgTa8)) solution, I like to
get immediate feedback on the progress, whether it's by compiling the
code, running it, running its tests, or often just type-checking it.

I see a couple of benefits, for instance:

1.  It increases my confidence that what I'm aiming for makes sense.
2.  I get better support from tools (e.g. auto-complete and filling up
    details they can infer from the context) when they aren't confused
    due to transient errors.

So it's important to me to minimise the time I spend in invalid,
transient states where programs don't even type-check.

I start by writing code against the interfaces I wish I had now and
leaving the details for future iterations, for instance:

- Matching on an enumeration, *but not implementing the arms*.
- Calling into *undeclared/undefined procedures realising lower-level
  mechanisms* from higher-level procedures describing policies.

Throughout this process, when writing Rust, the `todo!` macro, although
it's small (micro?) feature, has proven to be incredibly useful!

This is because I can always write `todo!=` when I need a placeholder to
satisfy the type-checker while I'm still figuring things out **without
interrupting my thought process/breaking my flow** due to type-errors
that I know I need to fix but won't right now. E.g. the type-checker
yells at me because it expected an \`i32\`, but got an \`()\`, exactly
because I'm thinking about the best produce such a value of type
\`i32\`.

# Example

I used to implement networking protocols, frequently modelled as
state-machines and translated into enumerations.

Then, after pulling bytes off a socket, I'd decide what to do by
matching on current state. At this point, although I might not have
thought all details through, I have a rough sketch as a starting point
nonetheless.

Let's say we have this overly simplified `enum`:

``` rust
enum State {
    InHeader,
    InPayload,
    InSignature
}
```

Our goal is to implement `process`, but we might not exactly how just
yet:

``` rust
fn process(input: &[u8], state: &mut State) {
    match state {}
}
```

So we play around and match on `state`, perhaps by asking
[rust-analyzer](https://rust-analyzer.github.io/manual.html#add_missing_match_arms),
and we get:

``` rust
fn process(input: &[u8], state: &mut State) {
    match state {
        State::InHeader => todo!(),
        State::InPayload => todo!(),
        State::InSignature => todo!()
    }
}
```

It's a humble macro with nice synergies with rust-analyzer for daily
coding.

Lastly, it's no exclusivity of Rust, e.g. Scala has
[???](https://www.scala-lang.org/api/2.13.3/scala/Predef$.html#???:Nothing)
and
