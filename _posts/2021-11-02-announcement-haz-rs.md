---
layout: post
title: "Announcement: haz-rs"
tags: rust
---

> A thin abstraction over polymorphic environments for Rust.

---

I've recently put together a rather small Rust crate [haz](https://github.com/rvarago/haz-rs) meant to support passing a subset of data around while satisfying the following requirements:

- not commit to a particular representation,
- not yield access to more information than what called functions need,
- not require to pass each field explicitly at call-site.

Essentially, the library reduces to the `Has<Component>` trait:

```rust
trait Has<Component> {
    fn access(&self) -> &Component;
}
```

Whereby implementing `Has<Component>` for `Container` we state that `Container` can lend read-only access to `Component`.

Moreover, there's a macro to ease implementation and a few helper functions offering different syntax styles for usage.

I'm yet to decide whether that's an idea which I should further pursue. Nevertheless, the code is openly available should you want to give it a go.

Please check out the documentation at [github](https://github.com/rvarago/haz-rs) or [crates.io](https://crates.io/crates/haz) for more information.

I'll forward to your feedback!
