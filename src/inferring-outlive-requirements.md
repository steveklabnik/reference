# Infer Outlives Requirements

The Rust compiler is able to infer certain lifetime
outlives requirements based on lifetime rules, rather
than having to state them explicitly.

## Motivation

In the following piece of code, we have the outlives
relation `T: 'a`; which can be read as `T` outlives `'a`.
However, looking at the definition of the `struct`, we
see that the field `bar` is a reference to `T` with a lifetime
of `'a`. Thus for this code to be valid, the compiler already
enforces the constraint `T: 'a`.

The compiler will also infer the outlives requirements
so that the `where`-clause is optional.

```rust
struct Foo<'a, T>
where
    T: 'a, // <-- optional
{
    bar: &'a T,
}
```

For more more information on the motivation and design, please
see RFC [#44493].

[#44493]: https://github.com/rust-lang/rust/issues/44493


## Examples

Here are a few examples were the compiler is able to
infer outlives requirements rather than having to state
them explicitly.

A `struct`, `enum` and `union` with a *reference* to a type parameter:
```rust
// The constraint `T: 'a` is inferred.
struct Foo<'a, T> {
    bar: &'a T,
}
```

```rust
// The constraint `T: 'a` is inferred.
union Foo<'a, T> {
    bar: &'a T,
}
```

```rust
// The constraint `T: 'a` is inferred.
enum Foo<'a, T> {
    One(Bar<'a, T>)
}

struct Bar<'a, T> {
    field: &'a T
}
```

An *explicit outlives clause* on the nested type:
```rust
// The constraint `T: 'b` is inferred.
struct Foo<'a, T> {
    bar: Bar<'a, T>
}

// `Bar<'b, K>` has an explicit `where` clause:
struct Bar<'b, K> where K: 'b {
    x: &'b (),
    y: K,
}
```

A `struct` with an inferred *lifetime outlives requirements*:
```rust
// The region constraint `b': 'a` is inferred.
struct Foo<'a, 'b, T> {
    x: &'a &'b T
}
```

*Associated types*:
```rust
// The constraint `<T as std::iter::Iterator>::Item : 'a` is inferred.
struct Foo<'a, T: Iterator> {
    bar: &'a T::Item
}
```

```rust
// The constraint `B: 'a` is inferred.
struct Foo<'a, A, B> where A: Bar<'a, B> {
    field: <A as Bar<'a, B>>::Type
}

trait Bar<'k, K> where K: 'k {
    type Type;
}

```

*Trait objects*:
```rust
// The constraint `T': 'a` is inferred.
struct Foo<'a, T> {
    field: Box<dyn Bar<'a, T>>
}

// contains explicit where clause
trait Bar<'b, K> where K: 'b {
}
```

