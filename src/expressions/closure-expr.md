# Closure expressions

> **<sup>Syntax</sup>**\
> _ClosureExpression_ :\
> &nbsp;&nbsp; `move`<sup>?</sup>\
> &nbsp;&nbsp; ( `||` | `|` _ClosureParameters_<sup>?</sup> `|` )\
> &nbsp;&nbsp; ([_Expression_] | `->` [_TypeNoBounds_]&nbsp;[_BlockExpression_])
>
> _ClosureParameters_ :\
> &nbsp;&nbsp; _ClosureParam_ (`,` _ClosureParam_)<sup>\*</sup> `,`<sup>?</sup>
>
> _ClosureParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> [_Pattern_]&nbsp;( `:` [_Type_] )<sup>?</sup>

A _closure expression_, also know as a lambda expression or a lambda, defines a closure and denotes it as a value, in a single expression.
A closure expression is a pipe-symbol-delimited (`|`) list of irrefutable [patterns] followed by an expression.
Type annotations may optionally be added for the type of the parameters or for the return type.
If there is a return type, the expression used for the body of the closure must be a normal [block].
A closure expression also may begin with the `move` keyword before the initial `|`.

A closure expression denotes a function that maps a list of parameters onto the expression that follows the parameters.
Just like a [`let` binding], the parameters are irrefutable [patterns], whose type annotation is optional and will be inferred from context if not given.
Each closure expression has a unique, anonymous type.

Closure expressions are most useful when passing functions as arguments to other functions, as an abbreviation for defining and capturing a separate function.

Significantly, closure expressions _capture their environment_, which regular [function definitions] do not.
Without the `move` keyword, the closure expression [infers how it captures each variable from its environment](../types/closure.md#capture-modes), preferring to capture by shared reference, effectively borrowing all outer variables mentioned inside the closure's body.
If needed the compiler will infer that instead mutable references should be taken, or that the values should be moved or copied (depending on their type) from the environment.
A closure can be forced to capture its environment by copying or moving values by prefixing it with the `move` keyword.
This is often used to ensure that the closure's type is `'static`.

The compiler will determine which of the [closure traits](../types/closure.md#call-traits-and-coercions) the closure's type will implement by how it acts on its captured variables.
The closure will also implement the closure traits [`Send`](../special-types-and-traits.md#send) and/or [`Sync`](../special-types-and-traits.md#sync) if all of its captured types do.
Closure traits allow functions to accept closures using generics, even though the exact types can't be named.

In this example, we define a function `ten_times` that takes a higher-order function argument, and we then call it with a closure expression as an argument, followed by a closure expression that moves values from its environment.

```rust
fn ten_times<F>(f: F) where F: Fn(i32) {
    for index in 0..10 {
        f(index);
    }
}

ten_times(|j| println!("hello, {}", j));
// With type annotations
ten_times(|j: i32| -> () { println!("hello, {}", j) });

let word = "konnichiwa".to_owned();
ten_times(move |j| println!("{}, {}", word, j));
```

## Attributes on closure parameters

Attributes on closure parameters follow the same rules and restrictions as [regular function parameters].

[block]: block-expr.md
[function definitions]: ../items/functions.md
[patterns]: ../patterns.md
[regular function parameters]: ../items/functions.md#attributes-on-function-parameters

[_Expression_]: ../expressions.md
[_BlockExpression_]: block-expr.md
[_TypeNoBounds_]: ../types.md#type-expressions
[_Pattern_]: ../patterns.md
[_Type_]: ../types.md#type-expressions
[`let` binding]: ../statements.md#let-statements
[_OuterAttribute_]: ../attributes.md
