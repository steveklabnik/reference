# Operator expressions

> **<sup>Syntax</sup>**\
> _OperatorExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_BorrowExpression_]\
> &nbsp;&nbsp; | [_DereferenceExpression_]\
> &nbsp;&nbsp; | [_ErrorPropagationExpression_]\
> &nbsp;&nbsp; | [_NegationExpression_]\
> &nbsp;&nbsp; | [_ArithmeticOrLogicalExpression_]\
> &nbsp;&nbsp; | [_ComparisonExpression_]\
> &nbsp;&nbsp; | [_LazyBooleanExpression_]\
> &nbsp;&nbsp; | [_TypeCastExpression_]\
> &nbsp;&nbsp; | [_AssignmentExpression_]\
> &nbsp;&nbsp; | [_CompoundAssignmentExpression_]

Operators are defined for built in types by the Rust language. Many of the
following operators can also be overloaded using traits in `std::ops` or
`std::cmp`.

## Overflow

Integer operators will panic when they overflow when compiled in debug mode.
The `-C debug-assertions` and `-C overflow-checks` compiler flags can be used
to control this more directly. The following things are considered to be
overflow:

* When `+`, `*` or `-` create a value greater than the maximum value, or less
  than the minimum value that can be stored. This includes unary `-` on the
  smallest value of any signed integer type.
* Using `/` or `%`, where the left-hand argument is the smallest integer of a
  signed integer type and the right-hand argument is `-1`.
* Using `<<` or `>>` where the right-hand argument is greater than or equal to
  the number of bits in the type of the left-hand argument, or is negative.

## Borrow operators

> **<sup>Syntax</sup>**\
> _BorrowExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; (`&`|`&&`) [_Expression_]\
> &nbsp;&nbsp; | (`&`|`&&`) `mut` [_Expression_]

The `&` (shared borrow) and `&mut` (mutable borrow) operators are unary prefix
operators. When applied to a [place expression], this expressions produces a
reference (pointer) to the location that the value refers to. The memory
location is also placed into a borrowed state for the duration of the reference.
For a shared borrow (`&`), this implies that the place may not be mutated, but
it may be read or shared again. For a mutable borrow (`&mut`), the place may not
be accessed in any way until the borrow expires. `&mut` evaluates its operand in
a mutable place expression context. If the `&` or `&mut` operators are applied
to a [value expression], then a [temporary value] is created.

These operators cannot be overloaded.

```rust
{
    // a temporary with value 7 is created that lasts for this scope.
    let shared_reference = &7;
}
let mut array = [-2, 3, 9];
{
    // Mutably borrows `array` for this scope.
    // `array` may only be used through `mutable_reference`.
    let mutable_reference = &mut array;
}
```

Even though `&&` is a single token ([the lazy 'and' operator](#lazy-boolean-operators)),
when used in the context of borrow expressions it works as two borrows:

```rust
// same meanings:
let a = &&  10;
let a = & & 10;

// same meanings:
let a = &&&&  mut 10;
let a = && && mut 10;
let a = & & & & mut 10;
```

## The dereference operator

> **<sup>Syntax</sup>**\
> _DereferenceExpression_ :\
> &nbsp;&nbsp; `*` [_Expression_]

The `*` (dereference) operator is also a unary prefix operator. When applied to
a [pointer](types/pointer.html) it denotes the pointed-to location. If
the expression is of type `&mut T` and `*mut T`, and is either a local
variable, a (nested) field of a local variable or is a mutable [place
expression], then the resulting memory location can be assigned to.
Dereferencing a raw pointer requires `unsafe`.

On non-pointer types `*x` is equivalent to `*std::ops::Deref::deref(&x)` in an
[immutable place expression context](expressions.html#mutability) and
`*std::ops::DerefMut::deref_mut(&mut x)` in a mutable place expression context.

```rust
let x = &7;
assert_eq!(*x, 7);
let y = &mut 9;
*y = 11;
assert_eq!(*y, 11);
```

## The question mark operator

> **<sup>Syntax</sup>**\
> _ErrorPropagationExpression_ :\
> &nbsp;&nbsp; [_Expression_] `?`

The question mark operator (`?`) unwraps valid values or returns erroneous
values, propagating them to the calling function. It is a unary postfix
operator that can only be applied to the types `Result<T, E>` and `Option<T>`.

When applied to values of the `Result<T, E>` type, it propagates errors. If
the value is `Err(e)`, then it will return `Err(From::from(e))` from the
enclosing function or closure. If applied to `Ok(x)`, then it will unwrap the
value to evaluate to `x`.

```rust
# use std::num::ParseIntError;
fn try_to_parse() -> Result<i32, ParseIntError> {
    let x: i32 = "123".parse()?; // x = 123
    let y: i32 = "24a".parse()?; // returns an Err() immediately
    Ok(x + y)                    // Doesn't run.
}

let res = try_to_parse();
println!("{:?}", res);
# assert!(res.is_err())
```

When applied to values of the `Option<T>` type, it propagates `None`s. If the
value is `None`, then it will return `None`. If applied to `Some(x)`, then it
will unwrap the value to evaluate to `x`.

```rust
fn try_option_some() -> Option<u8> {
    let val = Some(1)?;
    Some(val)
}
assert_eq!(try_option_some(), Some(1));

fn try_option_none() -> Option<u8> {
    let val = None?;
    Some(val)
}
assert_eq!(try_option_none(), None);
```

`?` cannot be overloaded.

## Negation operators

> **<sup>Syntax</sup>**\
> _NegationExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `-` [_Expression_]\
> &nbsp;&nbsp; | `!` [_Expression_]

These are the last two unary operators. This table summarizes the behavior of
them on primitive types and which traits are used to overload these operators
for other types. Remember that signed integers are always represented using
two's complement. The operands of all of these operators are evaluated in
[value expression context][value expression] so are moved or copied.

| Symbol | Integer     | `bool`      | Floating Point | Overloading Trait  |
|--------|-------------|-------------|----------------|--------------------|
| `-`    | Negation*   |             | Negation       | `std::ops::Neg`    |
| `!`    | Bitwise NOT | Logical NOT |                | `std::ops::Not`    |

\* Only for signed integer types.

Here are some example of these operators

```rust
let x = 6;
assert_eq!(-x, -6);
assert_eq!(!x, -7);
assert_eq!(true, !false);
```

## Arithmetic and Logical Binary Operators

> **<sup>Syntax</sup>**\
> _ArithmeticOrLogicalExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Expression_] `+` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `-` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `*` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `/` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `%` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `&` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `|` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `^` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `<<` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `>>` [_Expression_]

Binary operators expressions are all written with infix notation. This table
summarizes the behavior of arithmetic and logical binary operators on
primitive types and which traits are used to overload these operators for other
types. Remember that signed integers are always represented using two's
complement. The operands of all of these operators are evaluated in [value
expression context][value expression] so are moved or copied.

| Symbol | Integer                 | `bool`      | Floating Point | Overloading Trait  |
|--------|-------------------------|-------------|----------------|--------------------|
| `+`    | Addition                |             | Addition       | `std::ops::Add`    |
| `-`    | Subtraction             |             | Subtraction    | `std::ops::Sub`    |
| `*`    | Multiplication          |             | Multiplication | `std::ops::Mul`    |
| `/`    | Division*                |             | Division       | `std::ops::Div`    |
| `%`    | Remainder               |             | Remainder      | `std::ops::Rem`    |
| `&`    | Bitwise AND             | Logical AND |                | `std::ops::BitAnd` |
| <code>&#124;</code> | Bitwise OR | Logical OR  |                | `std::ops::BitOr`  |
| `^`    | Bitwise XOR             | Logical XOR |                | `std::ops::BitXor` |
| `<<`   | Left Shift              |             |                | `std::ops::Shl`    |
| `>>`   | Right Shift**            |             |                | `std::ops::Shr`    |

\* Integer division rounds towards zero.

\*\* Arithmetic right shift on signed integer types, logical right shift on
unsigned integer types.

Here are examples of these operators being used.

```rust
assert_eq!(3 + 6, 9);
assert_eq!(5.5 - 1.25, 4.25);
assert_eq!(-5 * 14, -70);
assert_eq!(14 / 3, 4);
assert_eq!(100 % 7, 2);
assert_eq!(0b1010 & 0b1100, 0b1000);
assert_eq!(0b1010 | 0b1100, 0b1110);
assert_eq!(0b1010 ^ 0b1100, 0b110);
assert_eq!(13 << 3, 104);
assert_eq!(-10 >> 2, -3);
```

## Comparison Operators

> **<sup>Syntax</sup>**\
> _ComparisonExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Expression_] `==` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `!=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `>` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `<` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `>=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `<=` [_Expression_]

Comparison operators are also defined both for primitive types and many type in
the standard library. Parentheses are required when chaining comparison
operators. For example, the expression `a == b == c` is invalid and may be
written as `(a == b) == c`.

Unlike arithmetic and logical operators, the traits for
overloading the operators the traits for these operators are used more
generally to show how a type may be compared and will likely be assumed to
define actual comparisons by functions that use these traits as bounds. Many
functions and macros in the standard library can then use that assumption
(although not to ensure safety). Unlike the arithmetic and logical operators
above, these operators implicitly take shared borrows of their operands,
evaluating them in [place expression context][place expression]:

```rust,ignore
a == b;
// is equivalent to
::std::cmp::PartialEq::eq(&a, &b);
```

This means that the operands don't have to be moved out of.

| Symbol | Meaning                  | Overloading method         |
|--------|--------------------------|----------------------------|
| `==`   | Equal                    | `std::cmp::PartialEq::eq`  |
| `!=`   | Not equal                | `std::cmp::PartialEq::ne`  |
| `>`    | Greater than             | `std::cmp::PartialOrd::gt` |
| `<`    | Less than                | `std::cmp::PartialOrd::lt` |
| `>=`   | Greater than or equal to | `std::cmp::PartialOrd::ge` |
| `<=`   | Less than or equal to    | `std::cmp::PartialOrd::le` |

Here are examples of the comparison operators being used.

```rust
assert!(123 == 123);
assert!(23 != -12);
assert!(12.5 > 12.2);
assert!([1, 2, 3] < [1, 3, 4]);
assert!('A' <= 'B');
assert!("World" >= "Hello");
```

## Lazy boolean operators

> **<sup>Syntax</sup>**\
> _LazyBooleanExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Expression_] `||` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `&&` [_Expression_]

The operators `||` and `&&` may be applied to operands of boolean type. The
`||` operator denotes logical 'or', and the `&&` operator denotes logical
'and'. They differ from `|` and `&` in that the right-hand operand is only
evaluated when the left-hand operand does not already determine the result of
the expression. That is, `||` only evaluates its right-hand operand when the
left-hand operand evaluates to `false`, and `&&` only when it evaluates to
`true`.

```rust
let x = false || true; // true
let y = false && panic!(); // false, doesn't evaluate `panic!()`
```

## Type cast expressions

> **<sup>Syntax</sup>**\
> _TypeCastExpression_ :\
> &nbsp;&nbsp; [_Expression_] `as` [_TypeNoBounds_]

A type cast expression is denoted with the binary operator `as`.

Executing an `as` expression casts the value on the left-hand side to the type
on the right-hand side.

An example of an `as` expression:

```rust
# fn sum(values: &[f64]) -> f64 { 0.0 }
# fn len(values: &[f64]) -> i32 { 0 }
fn average(values: &[f64]) -> f64 {
    let sum: f64 = sum(values);
    let size: f64 = len(values) as f64;
    sum / size
}
```

`as` can be used to explicitly perform [coercions](type-coercions.html), as
well as the following additional casts. Here `*T` means either `*const T` or
`*mut T`.

| Type of `e`           | `U`                   | Cast performed by `e as U`       |
|-----------------------|-----------------------|----------------------------------|
| Integer or Float type | Integer or Float type | Numeric cast                     |
| Field-less enum       | Integer type          | Enum cast                        |
| `bool` or `char`      | Integer type          | Primitive to integer cast        |
| `u8`                  | `char`                | `u8` to `char` cast              |
| `*T`                  | `*V` where `V: Sized` \* | Pointer to pointer cast       |
| `*T` where `T: Sized` | Numeric type          | Pointer to address cast          |
| Integer type          | `*V` where `V: Sized` | Address to pointer cast          |
| `&[T; n]`             | `*const T`            | Array to pointer cast            |
| [Function pointer](types/function-pointer.html) | `*V` where `V: Sized` | Function pointer to pointer cast |
| Function pointer      | Integer               | Function pointer to address cast |
| Closure \*\*          | Function pointer      | Closure to function pointer cast |

\* or `T` and `V` are compatible unsized types, e.g., both slices, both the
same trait object.

\*\* only for closures that do not capture (close over) any local variables

### Semantics

* Numeric cast
    * Casting between two integers of the same size (e.g. i32 -> u32) is a no-op
    * Casting from a larger integer to a smaller integer (e.g. u32 -> u8) will
      truncate
    * Casting from a smaller integer to a larger integer (e.g. u8 -> u32) will
        * zero-extend if the source is unsigned
        * sign-extend if the source is signed
    * Casting from a float to an integer will round the float towards zero
        * **[NOTE: currently this will cause Undefined Behavior if the rounded
          value cannot be represented by the target integer type][float-int]**.
          This includes Inf and NaN. This is a bug and will be fixed.
    * Casting from an integer to float will produce the floating point
      representation of the integer, rounded if necessary (rounding strategy
      unspecified)
    * Casting from an f32 to an f64 is perfect and lossless
    * Casting from an f64 to an f32 will produce the closest possible value
      (rounding strategy unspecified)
        * **[NOTE: currently this will cause Undefined Behavior if the value
          is finite but larger or smaller than the largest or smallest finite
          value representable by f32][float-float]**. This is a bug and will
          be fixed.
* Enum cast
    * Casts an enum to its discriminant, then uses a numeric cast if needed.
* Primitive to integer cast
    * `false` casts to `0`, `true` casts to `1`
    * `char` casts to the value of the code point, then uses a numeric cast if needed.
* `u8` to `char` cast
    * Casts to the `char` with the corresponding code point.

[float-int]: https://github.com/rust-lang/rust/issues/10184
[float-float]: https://github.com/rust-lang/rust/issues/15536

## Assignment expressions

> **<sup>Syntax</sup>**\
> _AssignmentExpression_ :\
> &nbsp;&nbsp; [_Expression_] `=` [_Expression_]

An _assignment expression_ consists of a [place expression] followed by an
equals sign (`=`) and a [value expression]. Such an expression always has
the [`unit` type].

Evaluating an assignment expression [drops](destructors.html) the left-hand
operand, unless it's an uninitialized local variable or field of a local variable,
and [either copies or moves](expressions.html#moved-and-copied-types) its
right-hand operand to its left-hand operand. The left-hand operand must be a
place expression: using a value expression results in a compiler error, rather
than promoting it to a temporary.

```rust
# let mut x = 0;
# let y = 0;
x = y;
```

## Compound assignment expressions

> **<sup>Syntax</sup>**\
> _CompoundAssignmentExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Expression_] `+=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `-=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `*=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `/=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `%=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `&=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `|=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `^=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `<<=` [_Expression_]\
> &nbsp;&nbsp; | [_Expression_] `>>=` [_Expression_]

The `+`, `-`, `*`, `/`, `%`, `&`, `|`, `^`, `<<`, and `>>` operators may be
composed with the `=` operator. The expression `place_exp OP= value` is
equivalent to `place_expr = place_expr OP val`. For example, `x = x + 1` may be
written as `x += 1`. Any such expression always has the [`unit` type].
These operators can all be overloaded using the trait with the same name as for
the normal operation followed by 'Assign', for example, `std::ops::AddAssign`
is used to overload `+=`. As with `=`, `place_expr` must be a [place
expression].

```rust
let mut x = 10;
x += 4;
assert_eq!(x, 14);
```

[place expression]: expressions.html#place-expressions-and-value-expressions
[value expression]: expressions.html#place-expressions-and-value-expressions
[temporary value]: expressions.html#temporary-lifetimes
[float-int]: https://github.com/rust-lang/rust/issues/10184
[float-float]: https://github.com/rust-lang/rust/issues/15536
[`unit` type]: types/tuple.html

[_BorrowExpression_]: #borrow-operators
[_DereferenceExpression_]: #the-dereference-operator
[_ErrorPropagationExpression_]: #the-question-mark-operator
[_NegationExpression_]: #negation-operators
[_ArithmeticOrLogicalExpression_]: #arithmetic-and-logical-binary-operators
[_ComparisonExpression_]: #comparison-operators
[_LazyBooleanExpression_]: #lazy-boolean-operators
[_TypeCastExpression_]: #type-cast-expressions
[_AssignmentExpression_]: #assignment-expressions
[_CompoundAssignmentExpression_]: #compound-assignment-expressions

[_Expression_]: expressions.html
[_TypeNoBounds_]: types.html#type-expressions
