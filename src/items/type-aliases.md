# Type aliases

> **<sup>Syntax</sup>**\
> _TypeAlias_ :\
> &nbsp;&nbsp; `type` [IDENTIFIER]&nbsp;[_GenericParams_]<sup>?</sup>
>              ( `:` [_TypeParamBounds_] )<sup>?</sup>
>              [_WhereClause_]<sup>?</sup> ( `=` [_Type_] )<sup>?</sup> `;`

A _type alias_ defines a new name for an existing [type] in the [type namespace] of the module or block where it is located.
Type aliases are declared with the keyword `type`.
Every value has a single, specific type, but may implement several different traits, or be compatible with several different type constraints.

For example, the following defines the type `Point` as a synonym for the type
`(u8, u8)`, the type of pairs of unsigned 8 bit integers:

```rust
type Point = (u8, u8);
let p: Point = (41, 68);
```

A type alias to a tuple-struct or unit-struct cannot be used to qualify that type's constructor:

```rust,compile_fail
struct MyStruct(u32);

use MyStruct as UseAlias;
type TypeAlias = MyStruct;

let _ = UseAlias(5); // OK
let _ = TypeAlias(5); // Doesn't work
```

A type alias without the [_Type_] specification may only appear as an
[associated type] in a [trait].

A type alias with [_TypeParamBounds_] may only specified when used as
an [associated type] in a [trait].

[IDENTIFIER]: ../identifiers.md
[_GenericParams_]: generics.md
[_TypeParamBounds_]: ../trait-bounds.md
[_WhereClause_]: generics.md#where-clauses
[_Type_]: ../types.md#type-expressions
[associated type]: associated-items.md#associated-types
[trait]: traits.md
[type]: ../types.md
[type namespace]: ../names/namespaces.md
