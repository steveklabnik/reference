# Traits

> **<sup>Syntax</sup>**\
> _Trait_ :\
> &nbsp;&nbsp; `unsafe`<sup>?</sup> `trait` [IDENTIFIER]&nbsp;
>              [_Generics_]<sup>?</sup>
>              ( `:` [_TypeParamBounds_]<sup>?</sup> )<sup>?</sup>
>              [_WhereClause_]<sup>?</sup> `{`\
> &nbsp;&nbsp;&nbsp;&nbsp; _TraitItem_<sup>\*</sup>\
> &nbsp;&nbsp; `}`
>
> _TraitItem_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; _TraitFunc_\
> &nbsp;&nbsp; &nbsp;&nbsp; | _TraitMethod_\
> &nbsp;&nbsp; &nbsp;&nbsp; | _TraitConst_\
> &nbsp;&nbsp; &nbsp;&nbsp; | _TraitType_\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_MacroInvocationSemi_]\
> &nbsp;&nbsp; )
>
> _TraitFunc_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _TraitFunctionDecl_ ( `;` | [_BlockExpression_] )
>
> _TraitMethod_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _TraitMethodDecl_ ( `;` | [_BlockExpression_] )
>
> _TraitFunctionDecl_ :\
> &nbsp;&nbsp; [_FunctionQualifiers_] `fn` [IDENTIFIER]&nbsp;[_Generics_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` _TraitFunctionParameters_<sup>?</sup> `)`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_FunctionReturnType_]<sup>?</sup> [_WhereClause_]<sup>?</sup>
>
> _TraitMethodDecl_ :\
> &nbsp;&nbsp; [_FunctionQualifiers_] `fn` [IDENTIFIER]&nbsp;[_Generics_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` [_SelfParam_] (`,` _TraitFunctionParam_)<sup>\*</sup> `,`<sup>?</sup> `)`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_FunctionReturnType_]<sup>?</sup> [_WhereClause_]<sup>?</sup>
>
> _TraitFunctionParameters_ :\
> &nbsp;&nbsp; _TraitFunctionParam_ (`,` _TraitFunctionParam_)<sup>\*</sup> `,`<sup>?</sup>
>
> _TraitFunctionParam_<sup>[†](#parameter-patterns)</sup> :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> ( [_Pattern_] `:` )<sup>?</sup> [_Type_]
>
> _TraitConst_ :\
> &nbsp;&nbsp; `const` [IDENTIFIER] `:` [_Type_]&nbsp;( `=` [_Expression_] )<sup>?</sup> `;`
>
> _TraitType_ :\
> &nbsp;&nbsp; `type` [IDENTIFIER] ( `:` [_TypeParamBounds_]<sup>?</sup> )<sup>?</sup> `;`

A _trait_ describes an abstract interface that types can implement. This
interface consists of [associated items], which come in three varieties:

- [functions](associated-items.md#associated-functions-and-methods)
- [types](associated-items.md#associated-types)
- [constants](associated-items.md#associated-constants)

All traits define an implicit type parameter `Self` that refers to "the type
that is implementing this interface". Traits may also contain additional type
parameters. These type parameters, including `Self`, may be constrained by
other traits and so forth [as usual][generics].

Traits are implemented for specific types through separate [implementations].

Items associated with a trait do not need to be defined in the trait, but they
may be. If the trait provides a definition, then this definition acts as a
default for any implementation which does not override it. If it does not, then
any implementation must provide a definition.

## Trait bounds

Generic items may use traits as [bounds] on their type parameters.

## Generic Traits

Type parameters can be specified for a trait to make it generic. These appear
after the trait name, using the same syntax used in [generic functions].

```rust
trait Seq<T> {
    fn len(&self) -> u32;
    fn elt_at(&self, n: u32) -> T;
    fn iter<F>(&self, f: F) where F: Fn(T);
}
```

## Object Safety

Object safe traits can be the base trait of a [trait object]. A trait is
*object safe* if it has the following qualities (defined in [RFC 255]):

* It must not require `Self: Sized`
* All associated functions must either have a `where Self: Sized` bound, or
    * Not have any type parameters (although lifetime parameters are allowed),
      and
    * Be a [method] that does not use `Self` except in the type of the receiver.
* It must not have any associated constants.
* All supertraits must also be object safe.

## Supertraits

**Supertraits** are traits that are required to be implemented for a type to
implement a specific trait. Furthermore, anywhere a [generic][generics] or [trait object]
is bounded by a trait, it has access to the associated items of its supertraits.

Supertraits are declared by trait bounds on the `Self` type of a trait and
transitively the supertraits of the traits declared in those trait bounds. It is
an error for a trait to be its own supertrait.

The trait with a supertrait is called a **subtrait** of its supertrait.

The following is an example of declaring `Shape` to be a supertrait of `Circle`.

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle : Shape { fn radius(&self) -> f64; }
```

And the following is the same example, except using [where clauses].

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape { fn radius(&self) -> f64; }
```

This next example gives `radius` a default implementation using the `area`
function from `Shape`.

```rust
# trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape {
    fn radius(&self) -> f64 {
        // A = pi * r^2
        // so algebraically,
        // r = sqrt(A / pi)
        (self.area() /std::f64::consts::PI).sqrt()
    }
}
```

This next example calls a supertrait method on a generic parameter.

```rust
# trait Shape { fn area(&self) -> f64; }
# trait Circle : Shape { fn radius(&self) -> f64; }
fn print_area_and_radius<C: Circle>(c: C) {
    // Here we call the area method from the supertrait `Shape` of `Circle`.
    println!("Area: {}", c.area());
    println!("Radius: {}", c.radius());
}
```

Similarly, here is an example of calling supertrait methods on trait objects.

```rust
# trait Shape { fn area(&self) -> f64; }
# trait Circle : Shape { fn radius(&self) -> f64; }
# struct UnitCircle;
# impl Shape for UnitCircle { fn area(&self) -> f64 { std::f64::consts::PI } }
# impl Circle for UnitCircle { fn radius(&self) -> f64 { 1.0 } }
# let circle = UnitCircle;
let circle = Box::new(circle) as Box<dyn Circle>;
let nonsense = circle.radius() * circle.area();
```

## Unsafe traits

Traits items that begin with the `unsafe` keyword indicate that *implementing* the
trait may be [unsafe]. It is safe to use a correctly implemented unsafe trait.
The [trait implementation] must also begin with the `unsafe` keyword.

[`Sync`] and [`Send`] are examples of unsafe traits.

## Parameter patterns

In function and method declarations, the parameter pattern is required.

If the function or method has a body, any irrefutable pattern is allowed.

If the function or method has no body, the pattern must have one of the
following forms:

* [IDENTIFIER]
* `mut` [IDENTIFIER]
* [`_`][WildcardPattern]
* `&` [IDENTIFIER]
* `&&` [IDENTIFIER]

```rust,edition2018
trait T {
    fn f1((a, b): (i32, i32)) {}
    fn f2(_: (i32, i32));  // Cannot use tuple pattern without a body.
}
```

`mut` [IDENTIFIER] is deprecated and will become a hard error in the future.
<!-- https://github.com/rust-lang/rust/issues/35203 -->

> **Edition Differences**: In the 2015 edition, the pattern for a trait
> function or method parameter is optional:
>
> ```rust,edition2015
> trait T {
>     fn f(i32);  // Parameter identifiers are not required.
> }
> ```
>
> Further, the limitations on the pattern for functions or methods without a
> body also apply to functions or methods with a body.


[IDENTIFIER]: ../identifiers.md
[WildcardPattern]: ../patterns.md#wildcard-pattern
[_BlockExpression_]: ../expressions/block-expr.md
[_Expression_]: ../expressions.md
[_FunctionQualifiers_]: functions.md
[_FunctionReturnType_]: functions.md
[_Generics_]: generics.md
[_MacroInvocationSemi_]: ../macros.md#macro-invocation
[_OuterAttribute_]: ../attributes.md
[_Pattern_]: ../patterns.md
[_SelfParam_]: associated-items.md#methods
[_TypeParamBounds_]: ../trait-bounds.md
[_Type_]: ../types.md#type-expressions
[_WhereClause_]: generics.md#where-clauses
[bounds]: ../trait-bounds.md
[trait object]: ../types/trait-object.md
[RFC 255]: https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md
[associated items]: associated-items.md
[method]: associated-items.md#methods
[implementations]: implementations.md
[generics]: generics.md
[where clauses]: generics.md#where-clauses
[generic functions]: functions.md#generic-functions
[unsafe]: ../unsafety.md
[trait implementation]: implementations.md#trait-implementations
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
