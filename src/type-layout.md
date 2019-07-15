# Type Layout

The layout of a type is its size, alignment, and the relative offsets of its
fields. For enums, how the discriminant is laid out and interpreted is also part
of type layout.

Type layout can be changed with each compilation. Instead of trying to document
exactly what is done, we only document what is guaranteed today.

## Size and Alignment

All values have an alignment and size.

The *alignment* of a value specifies what addresses are valid to store the value
at. A value of alignment `n` must only be stored at an address that is a
multiple of n. For example, a value with an alignment of 2 must be stored at an
even address, while a value with an alignment of 1 can be stored at any address.
Alignment is measured in bytes, and must be at least 1, and always a power of 2.
The alignment of a value can be checked with the [`align_of_val`] function.

The *size* of a value is the offset in bytes between successive elements in an
array with that item type including alignment padding. The size of a value is
always a multiple of its alignment. The size of a value can be checked with the
[`size_of_val`] function.

Types where all values have the same size and alignment known at compile time
implement the [`Sized`] trait and can be checked with the [`size_of`] and
[`align_of`] functions. Types that are not [`Sized`] are known as [dynamically
sized types]. Since all values of a `Sized` type share the same size and
alignment, we refer to those shared values as the size of the type and the
alignment of the type respectively.

## Primitive Data Layout

The size of most primitives is given in this table.

Type | `size_of::<Type>()`
- | - | -
bool | 1
u8 | 1
u16 | 2
u32 | 4
u64 | 8
u128 | 16
i8 | 1
i16 | 2
i32 | 4
i64 | 8
i128 | 16
f32 | 4
f64 | 8
char | 4

`usize` and `isize` have a size big enough to contain every address on the
target platform. For example, on a 32 bit target, this is 4 bytes and on a 64
bit target, this is 8 bytes.

Most primitives are generally aligned to their size, although this is
platform-specific behavior. In particular, on x86 u64 and f64 are only
aligned to 32 bits.

## Pointers and References Layout

Pointers and references have the same layout. Mutability of the pointer or
reference does not change the layout.

Pointers to sized types have the same size and alignment as `usize`.

Pointers to unsized types are sized. The size and alignment is guaranteed to be
at least equal to the size and alignment of a pointer.

> Note: Though you should not rely on this, all pointers to <abbr
> title="Dynamically Sized Types">DSTs</abbr> are currently twice the size of
> the size of `usize` and have the same alignment.

## Array Layout

Arrays are laid out so that the `nth` element of the array is offset from the
start of the array by `n * the size of the type` bytes. An array of `[T; n]`
has a size of `size_of::<T>() * n` and the same alignment of `T`.

## Slice Layout

Slices have the same layout as the section of the array they slice.

> Note: This is about the raw `[T]` type, not pointers (`&[T]`, `Box<[T]>`,
> etc.) to slices.

## `str` Layout
String slices are a UTF-8 representation of characters that have the same layout as slices of type `[u8]`.

## Tuple Layout

Tuples do not have any guarantees about their layout.

The exception to this is the unit tuple (`()`) which is guaranteed as a
zero-sized type to have a size of 0 and an alignment of 1.

## Trait Object Layout

Trait objects have the same layout as the value the trait object is of.

> Note: This is about the raw trait object types, not pointers (`&Trait`,
> `Box<Trait>`, etc.) to trait objects.

## Closure Layout

Closures have no layout guarantees.

## Representations

All user-defined composite types (`struct`s, `enum`s, and `union`s) have a
*representation* that specifies what the layout is for the type. The possible
representations for a type are:

- [Default]
- [`C`]
- The [primitive representations]
- [`transparent`]

The representation of a type can be changed by applying the `repr` attribute
to it. The following example shows a struct with a `C` representation.

```rust
#[repr(C)]
struct ThreeInts {
    first: i16,
    second: i8,
    third: i32
}
```

The alignment may be raised or lowered with the `align` and `packed` modifiers
respectively. They alter the representation specified in the attribute.
If no representation is specified, the default one is altered.

```rust
// Default representation, alignment lowered to 2.
#[repr(packed(2))]
struct PackedStruct {
    first: i16,
    second: i8,
    third: i32
}

// C representation, alignment raised to 8
#[repr(C, align(8))]
struct AlignedStruct {
    first: i16,
    second: i8,
    third: i32
}
```

> Note: As a consequence of the representation being an attribute on the item,
> the representation does not depend on generic parameters. Any two types with
> the same name have the same representation. For example, `Foo<Bar>` and
> `Foo<Baz>` both have the same representation.

The representation of a type can change the padding between fields, but does
not change the layout of the fields themselves. For example, a struct with a
`C` representation that contains a struct `Inner` with the default
representation will not change the layout of `Inner`.

### The Default Representation

Nominal types without a `repr` attribute have the default representation.
Informally, this representation is also called the `rust` representation.

There are no guarantees of data layout made by this representation.

### The `C` Representation

The `C` representation is designed for dual purposes. One purpose is for
creating types that are interoperable with the C Language. The second purpose is
to create types that you can soundly perform operations on that rely on data
layout such as reinterpreting values as a different type.

Because of this dual purpose, it is possible to create types that are not useful
for interfacing with the C programming language.

This representation can be applied to structs, unions, and enums.

#### \#[repr(C)] Structs

The alignment of the struct is the alignment of the most-aligned field in it.

The size and offset of fields is determined by the following algorithm.

Start with a current offset of 0 bytes.

For each field in declaration order in the struct, first determine the size and
alignment of the field. If the current offset is not a multiple of the field's
alignment, then add padding bytes to the current offset until it is a multiple
of the field's alignment. The offset for the field is what the current offset
is now. Then increase the current offset by the size of the field.

Finally, the size of the struct is the current offset rounded up to the nearest
multiple of the struct's alignment.

Here is this algorithm described in pseudocode.

```rust,ignore
struct.alignment = struct.fields().map(|field| field.alignment).max();

let current_offset = 0;

for field in struct.fields_in_declaration_order() {
    // Increase the current offset so that it's a multiple of the alignment
    // of this field. For the first field, this will always be zero.
    // The skipped bytes are called padding bytes.
    current_offset += field.alignment % current_offset;

    struct[field].offset = current_offset;

    current_offset += field.size;
}

struct.size = current_offset + current_offset % struct.alignment;
```

> Note: This algorithm can produce zero-sized structs. This differs from
> C where structs without data still have a size of one byte.

#### \#[repr(C)] Unions

A union declared with `#[repr(C)]` will have the same size and alignment as an
equivalent C union declaration in the C language for the target platform.
The union will have a size of the maximum size of all of its fields rounded to
its alignment, and an alignment of the maximum alignment of all of its fields.
These maximums may come from different fields.

```rust
#[repr(C)]
union Union {
    f1: u16,
    f2: [u8; 4],
}

assert_eq!(std::mem::size_of::<Union>(), 4);  // From f2
assert_eq!(std::mem::align_of::<Union>(), 2); // From f1

#[repr(C)]
union SizeRoundedUp {
   a: u32,
   b: [u16; 3],
}

assert_eq!(std::mem::size_of::<SizeRoundedUp>(), 8);  // Size of 6 from b,
                                                      // rounded up to 8 from
                                                      // alignment of a.
assert_eq!(std::mem::align_of::<SizeRoundedUp>(), 4); // From a
```

#### \#[repr(C)] Enums

For [C-like enumerations], the `C` representation has the size and alignment of
the default `enum` size and alignment for the target platform's C ABI.

> Note: The enum representation in C is implementation defined, so this is
> really a "best guess". In particular, this may be incorrect when the C code
> of interest is compiled with certain flags.

<div class="warning">

Warning: There are crucial differences between an `enum` in the C language and
Rust's C-like enumerations with this representation. An `enum` in  C is
mostly a `typedef` plus some named constants; in other words, an object of an
`enum` type can hold any integer value. For example, this is often used for
bitflags in `C`. In contrast, Rust’s C-like enumerations can only legally hold
the discriminant values, everything else is undefined behaviour. Therefore,
using a C-like enumeration in FFI to model a C `enum` is often wrong.

</div>

It is an error for [zero-variant enumerations] to have the `C` representation.

For all other enumerations, the layout is unspecified.

Likewise, combining the `C` representation with a primitive representation, the
layout is unspecified.

### Primitive representations

The *primitive representations* are the representations with the same names as
the primitive integer types. That is: `u8`, `u16`, `u32`, `u64`, `u128`,
`usize`, `i8`, `i16`, `i32`, `i64`, `i128`, and `isize`.

Primitive representations can only be applied to enumerations.

For [C-like enumerations], they set the size and alignment to be the same as the
primitive type of the same name. For example, a C-like enumeration with a `u8`
representation can only have discriminants between 0 and 255 inclusive.

It is an error for [zero-variant enumerations] to have a primitive
representation.

For all other enumerations, the layout is unspecified.

Likewise, combining two primitive representations together is unspecified.

### The alignment modifiers

The `align` and `packed` modifiers can be used to respectively raise or lower
the alignment of `struct`s and `union`s. `packed` may also alter the padding
between fields.

The alignment is specified as an integer parameter in the form of
`#[repr(align(x))]` or `#[repr(packed(x))]`. The alignment value must be a
power of two from 1 up to 2<sup>29</sup>. For `packed`, if no value is given,
as in `#[repr(packed)]`, then the value is 1.

For `align`, if the specified alignment is less than the alignment of the type
without the `align` modifier, then the alignment is unaffected.

For `packed`, if the specified alignment is greater than the type's alignment
without the `packed` modifier, then the alignment and layout is unaffected.
The alignments of each field, for the purpose of positioning fields, is the
smaller of the specified alignment and the alignment of the field's type.

The `align` and `packed` modifiers cannot be applied on the same type and a
`packed` type cannot transitively contain another `align`ed type. `align` and
`packed` may only be applied to the [default] and [`C`] representations.

The `align` modifier can also be applied on an `enum`.
When it is, the effect on the `enum`'s alignment is the same as if the `enum`
was wrapped in a newtype `struct` with the same `align` modifier.

<div class="warning">

***Warning:*** Dereferencing an unaligned pointer is [undefined behavior] and
it is possible to [safely create unaligned pointers to `packed` fields][27060].
Like all ways to create undefined behavior in safe Rust, this is a bug.

</div>

### The `transparent` Representation

The `transparent` representation can only be used on `struct`s that have a
single non-zero sized field and any number of zero-sized fields, including
[`PhantomData<T>`].

Structs with this representation have the same layout and ABI as the single
non-zero sized field.

This is different than the `C` representation because
a struct with the `C` representation will always have the ABI of a `C` `struct`
while, for example, a struct with the `transparent` representation with a
primitive field will have the ABI of the primitive field.

Because this representation delegates type layout to another type, it cannot be
used with any other representation.

[`align_of_val`]: ../std/mem/fn.align_of_val.html
[`size_of_val`]: ../std/mem/fn.size_of_val.html
[`align_of`]: ../std/mem/fn.align_of.html
[`size_of`]: ../std/mem/fn.size_of.html
[`Sized`]: ../std/marker/trait.Sized.html
[dynamically sized types]: dynamically-sized-types.html
[C-like enumerations]:  items/enumerations.html#explicit-discriminants
[zero-variant enumerations]: items/enumerations.html#zero-variant-enums
[undefined behavior]: behavior-considered-undefined.html
[27060]: https://github.com/rust-lang/rust/issues/27060
[`PhantomData<T>`]: special-types-and-traits.html#phantomdatat
[Default]: #the-default-representation
[`C`]: #the-c-representation
[primitive representations]: #primitive-representations
[`transparent`]: #the-transparent-representation
