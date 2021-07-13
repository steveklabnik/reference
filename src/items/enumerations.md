# Enumerations

> **<sup>Syntax</sup>**\
> _Enumeration_ :\
> &nbsp;&nbsp; `enum`
>    [IDENTIFIER]&nbsp;
>    [_GenericParams_]<sup>?</sup>
>    [_WhereClause_]<sup>?</sup>
>    `{` _EnumItems_<sup>?</sup> `}`
>
> _EnumItems_ :\
> &nbsp;&nbsp; _EnumItem_ ( `,` _EnumItem_ )<sup>\*</sup> `,`<sup>?</sup>
>
> _EnumItem_ :\
> &nbsp;&nbsp; _OuterAttribute_<sup>\*</sup> [_Visibility_]<sup>?</sup>\
> &nbsp;&nbsp; [IDENTIFIER]&nbsp;( _EnumItemTuple_ | _EnumItemStruct_
>                                | _EnumItemDiscriminant_ )<sup>?</sup>
>
> _EnumItemTuple_ :\
> &nbsp;&nbsp; `(` [_TupleFields_]<sup>?</sup> `)`
>
> _EnumItemStruct_ :\
> &nbsp;&nbsp; `{` [_StructFields_]<sup>?</sup> `}`
>
> _EnumItemDiscriminant_ :\
> &nbsp;&nbsp; `=` [_Expression_]

An *enumeration*, also referred to as an *enum*, is a simultaneous definition of a
nominal [enumerated type] as well as a set of *constructors*, that can be used
to create or pattern-match values of the corresponding enumerated type.

Enumerations are declared with the keyword `enum`.
The `enum` declaration defines the enumeration type in the [type namespace] of the module or block where it is located.

An example of an `enum` item and its use:

```rust
enum Animal {
    Dog,
    Cat,
}

let mut a: Animal = Animal::Dog;
a = Animal::Cat;
```

Enum constructors can have either named or unnamed fields:

```rust
enum Animal {
    Dog(String, f64),
    Cat { name: String, weight: f64 },
}

let mut a: Animal = Animal::Dog("Cocoa".to_string(), 37.2);
a = Animal::Cat { name: "Spotty".to_string(), weight: 2.7 };
```

In this example, `Cat` is a _struct-like enum variant_, whereas `Dog` is simply
called an enum variant. Each enum instance has a _discriminant_ which is an
integer associated to it that is used to determine which variant it holds. An
opaque reference to this discriminant can be obtained with the
[`mem::discriminant`] function.

Variant constructors are similar to [struct] definitions, and can be referenced by a path from the enumeration name, including in [use declarations].
Each variant defines its type in the [type namespace], though that type cannot be used as a type specifier.
Each variant also defines a constructor in the [value namespace].

A struct-like variant can be instantiated with a [struct expression].
A tuple-like variant can be instantiated with a [call expression] or a [struct expression].
A unit-like variant can be instantiated with a [path expression] or a [struct expression].
For example:

```rust
enum Examples {
    UnitLike,
    TupleLike(i32),
    StructLike { value: i32 },
}

use Examples::*; // Creates aliases to all variants.
let x = UnitLike; // Path expression of the const item.
let x = UnitLike {}; // Struct expression.
let y = TupleLike(123); // Call expression.
let y = TupleLike { 0: 123 }; // Struct expression using integer field names.
let z = StructLike { value: 123 }; // Struct expression.
```

## Custom Discriminant Values for Fieldless Enumerations

If there is no data attached to *any* of the variants of an enumeration,
then the discriminant can be directly chosen and accessed.

These enumerations can be cast to integer types with the `as` operator by a
[numeric cast]. The enumeration can optionally specify which integer each
discriminant gets by following the variant name with `=` followed by a [constant
expression]. If the first variant in the declaration is unspecified, then it is
set to zero. For every other unspecified discriminant, it is set to one higher
than the previous variant in the declaration.

```rust
enum Foo {
    Bar,            // 0
    Baz = 123,      // 123
    Quux,           // 124
}

let baz_discriminant = Foo::Baz as u32;
assert_eq!(baz_discriminant, 123);
```

Under the [default representation], the specified discriminant is interpreted as
an `isize` value although the compiler is allowed to use a smaller type in the
actual memory layout. The size and thus acceptable values can be changed by
using a [primitive representation] or the [`C` representation].

It is an error when two variants share the same discriminant.

```rust,compile_fail
enum SharedDiscriminantError {
    SharedA = 1,
    SharedB = 1
}

enum SharedDiscriminantError2 {
    Zero,       // 0
    One,        // 1
    OneToo = 1  // 1 (collision with previous!)
}
```

It is also an error to have an unspecified discriminant where the previous
discriminant is the maximum value for the size of the discriminant.

```rust,compile_fail
#[repr(u8)]
enum OverflowingDiscriminantError {
    Max = 255,
    MaxPlusOne // Would be 256, but that overflows the enum.
}

#[repr(u8)]
enum OverflowingDiscriminantError2 {
    MaxMinusOne = 254, // 254
    Max,               // 255
    MaxPlusOne         // Would be 256, but that overflows the enum.
}
```

## Zero-variant Enums

Enums with zero variants are known as *zero-variant enums*. As they have
no valid values, they cannot be instantiated.

```rust
enum ZeroVariants {}
```

Zero-variant enums are equivalent to the [never type], but they cannot be
coerced into other types.

```rust,compile_fail
# enum ZeroVariants {}
let x: ZeroVariants = panic!();
let y: u32 = x; // mismatched type error
```

## Variant visibility

Enum variants syntactically allow a [_Visibility_] annotation, but this is
rejected when the enum is validated. This allows items to be parsed with a
unified syntax across different contexts where they are used.

```rust
macro_rules! mac_variant {
    ($vis:vis $name:ident) => {
        enum $name {
            $vis Unit,

            $vis Tuple(u8, u16),

            $vis Struct { f: u8 },
        }
    }
}

// Empty `vis` is allowed.
mac_variant! { E }

// This is allowed, since it is removed before being validated.
#[cfg(FALSE)]
enum E {
    pub U,
    pub(crate) T(u8),
    pub(super) T { f: String }
}
```

[_Expression_]: ../expressions.md
[_GenericParams_]: generics.md
[_StructFields_]: structs.md
[_TupleFields_]: structs.md
[_Visibility_]: ../visibility-and-privacy.md
[_WhereClause_]: generics.md#where-clauses
[`C` representation]: ../type-layout.md#the-c-representation
[`mem::discriminant`]: ../../std/mem/fn.discriminant.html
[call expression]: ../expressions/call-expr.md
[constant expression]: ../const_eval.md#constant-expressions
[default representation]: ../type-layout.md#the-default-representation
[enumerated type]: ../types/enum.md
[IDENTIFIER]: ../identifiers.md
[never type]: ../types/never.md
[numeric cast]: ../expressions/operator-expr.md#semantics
[path expression]: ../expressions/path-expr.md
[primitive representation]: ../type-layout.md#primitive-representations
[struct expression]: ../expressions/struct-expr.md
[struct]: structs.md
[type namespace]: ../names/namespaces.md
[use declarations]: use-declarations.md
[value namespace]: ../names/namespaces.md
