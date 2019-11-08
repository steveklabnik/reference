# Use declarations

> **<sup>Syntax:</sup>**\
> _UseDeclaration_ :\
> &nbsp;&nbsp; `use` _UseTree_ `;`
>
> _UseTree_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; ([_SimplePath_]<sup>?</sup> `::`)<sup>?</sup> `*`\
> &nbsp;&nbsp; | ([_SimplePath_]<sup>?</sup> `::`)<sup>?</sup> `{` (_UseTree_ ( `,`  _UseTree_ )<sup>\*</sup> `,`<sup>?</sup>)<sup>?</sup> `}`\
> &nbsp;&nbsp; | [_SimplePath_]&nbsp;( `as` ( [IDENTIFIER] | `_` ) )<sup>?</sup>

A _use declaration_ creates one or more local name bindings synonymous with
some other [path]. Usually a `use` declaration is used to shorten the path
required to refer to a module item. These declarations may appear in [modules]
and [blocks], usually at the top.

[path]: ../paths.md
[modules]: modules.md
[blocks]: ../expressions/block-expr.md

Use declarations support a number of convenient shortcuts:

* Simultaneously binding a list of paths with a common prefix, using the
  glob-like brace syntax `use a::b::{c, d, e::f, g::h::i};`
* Simultaneously binding a list of paths with a common prefix and their common
  parent module, using the `self` keyword, such as `use a::b::{self, c, d::e};`
* Rebinding the target name as a new local name, using the syntax `use p::q::r
  as x;`. This can also be used with the last two features:
  `use a::b::{self as ab, c as abc}`.
* Binding all paths matching a given prefix, using the asterisk wildcard syntax
  `use a::b::*;`.
* Nesting groups of the previous features multiple times, such as
  `use a::b::{self as ab, c, d::{*, e::f}};`

An example of `use` declarations:

```rust
use std::option::Option::{Some, None};
use std::collections::hash_map::{self, HashMap};

fn foo<T>(_: T){}
fn bar(map1: HashMap<String, usize>, map2: hash_map::HashMap<String, usize>){}

fn main() {
    // Equivalent to 'foo(vec![std::option::Option::Some(1.0f64),
    // std::option::Option::None]);'
    foo(vec![Some(1.0f64), None]);

    // Both `hash_map` and `HashMap` are in scope.
    let map1 = HashMap::new();
    let map2 = hash_map::HashMap::new();
    bar(map1, map2);
}
```

## `use` Visibility

Like items, `use` declarations are private to the containing module, by
default. Also like items, a `use` declaration can be public, if qualified by
the `pub` keyword. Such a `use` declaration serves to _re-export_ a name. A
public `use` declaration can therefore _redirect_ some public name to a
different target definition: even a definition with a private canonical path,
inside a different module. If a sequence of such redirections form a cycle or
cannot be resolved unambiguously, they represent a compile-time error.

An example of re-exporting:

```rust
# fn main() { }
mod quux {
    pub use quux::foo::{bar, baz};

    pub mod foo {
        pub fn bar() { }
        pub fn baz() { }
    }
}
```

In this example, the module `quux` re-exports two public names defined in
`foo`.

## `use` Paths

A path in a `use` declaration must start with an identifier or one of the
[path qualifiers] `crate`, `self`, `super`, or `::`.

If the path starts with `crate`, `self`, or `super`, it is resolved in the
same way as any other path.

If the path starts with `::` or an identifier, the resolution is different in
different Rust editions.

> **Edition Differences**:
>
> In the 2015 edition, if a path in a `use` declaration starts with `::` or an
> identifier, it is resolved from the crate root (as if it started with the
> `crate` path qualifier).
>
> To make it possible to use a crate name in a `use` path, use an
> [`extern crate`] declaration to place it in the crate root.
>
> Examples of what will and will not work in the 2015 edition:
>
> ```rust,edition2015
>
> # #![allow(unused_imports)]
> use std::path::{self, Path, PathBuf};  // good: std is at the root of the crate
> use crate::foo::baz::foobaz;    // good: foo is at the root of the crate
> use foo::bar;    // good: foo is at the root of the crate
>
> mod foo {
>
>     use std::path::Path;  // good: std is at the root of the crate
>     // use example::iter;      // bad: example is not at the root of the crate
>     use self::baz::foobaz;  // good: self refers to module 'foo'
>     use crate::foo::bar::foobar;   // good: foo is at the root of the crate
>     use foo::bar::foobar as fb2; // good: foo is at the root of the crate
>     use ::foo::bar::foobar as fb3; // good: foo is at the root of the crate
>
>     mod example {
>         pub mod iter {}
>     }
>
>     pub mod bar {
>         pub fn foobar() { }
>     }
>
>     pub mod baz {
>         use super::bar::foobar; // good: super refers to module 'foo'
>         pub fn foobaz() { }
>     }
> }
>
> fn main() {}
> ```
>
> Beginning in the 2018 edition, a path in a `use` declaration which starts
> with `::` is resolved from the [extern prelude] instead of the crate root.
>
> Also, beginning in the 2018 edition, a path in a `use` declaration which
> starts with an identifier is resolved in the same way as any other path,
> except that it is an error if the path has more than one possible resolution
> (unless they resolve to the same thing); 'inner' bindings do not shadow
> 'outer' ones.
>
> It is an error if the path resolves to a local variable or generic parameter.
>
> Examples of what will and will not work in the 2018 edition:
>
> ```rust,edition2018
> # #![allow(unused_imports)]
> mod foo {
>     use ::std::path::Path;       // good: std is in the extern prelude
>     use std::path::Path as P2;   // good: std is found in the extern prelude
>     use example::iter;           // good: example is in scope
>     use crate::foo::bar::foobar; // good: foo is at the root of the crate
>     // use ::foo::bar::foobar;   // bad: foo is not in the extern prelude
>     // use foo::bar::foobar;     // bad: foo is not in scope
>
>     mod example {
>         pub mod iter {}
>     }
>
>     pub mod bar {
>         pub fn foobar() { }
>     }
> }
>
> fn main() {
>     fn f() {}
>     use f as g; // good: f is in scope
>     let x = 3;
>     // use x as y; // bad: local variables are not available
> }
> ```
>
> Examples for the ambiguity rule:
> ```rust,edition2018
> # #![allow(unused_imports)]
> // use std::fs; // bad: this is ambiguous.
> use ::std::fs;  // good: imports from the `std` crate, not the module below.
> use self::std::fs as self_fs;  // good: imports the module below.
>
> mod std {
>     pub mod fs {}
> }
>
> # fn main() {}
> ```

## Underscore Imports

Items can be imported without binding to a name by using an underscore with
the form `use path as _`. This is particularly useful to import a trait so
that its methods may be used without importing the trait's symbol, for example
if the trait's symbol may conflict with another symbol. Another example is to
link an external crate without importing its name.

Asterisk glob imports will import items imported with `_` in their unnameable
form.

```rust
mod foo {
    pub trait Zoo {
        fn zoo(&self) {}
    }

    impl<T> Zoo for T {}
}

use self::foo::Zoo as _;
struct Zoo;  // Underscore import avoids name conflict with this item.

fn main() {
    let z = Zoo;
    z.zoo();
}
```

The unique, unnameable symbols are created after macro expansion so that
macros may safely emit multiple references to `_` imports. For example, the
following should not produce an error:

```rust
macro_rules! m {
    ($item: item) => { $item $item }
}

m!(use std as _;);
// This expands to:
// use std as _;
// use std as _;
```

[IDENTIFIER]: ../identifiers.md
[_SimplePath_]: ../paths.md#simple-paths
[`extern crate`]: extern-crates.md
[extern prelude]: extern-crates.md#extern-prelude
[path qualifiers]: ../paths.md#path-qualifiers
