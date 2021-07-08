# Visibility and Privacy

> **<sup>Syntax<sup>**\
> _Visibility_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `pub`\
> &nbsp;&nbsp; | `pub` `(` `crate` `)`\
> &nbsp;&nbsp; | `pub` `(` `self` `)`\
> &nbsp;&nbsp; | `pub` `(` `super` `)`\
> &nbsp;&nbsp; | `pub` `(` `in` [_SimplePath_] `)`

The terms "visibility" and "privacy" are complimentary words used to convey the
question, "Can this item be used at this location?"

Using an item means referencing any item in any way such as invoking a
function, creating a struct or adding a `use` statement. The "location" of an
item generally means, "What crate is the item in? And (sometimes) what module,
or path of modules, within the crate?" But sometimes there is more to it. For
example, the location of a struct field is, obviously, within some struct. Or,
since a struct may be declared inside of a function, the location of a struct
may be within a function.

The Rust compiler does a "privacy check" on every item usage to ensure that the
item is visible to that location. If you use an item where it cannot be used,
you will get a privacy error such as, "function \`function\_name\` is private".

By default, everything in Rust is private, with two exceptions: Associated
items in a Trait and Enum variants. Items declared as `pub` are considered
public. For example:

```rust
# fn main() {}
// private struct
struct Foo;

// public struct
pub struct Bar {
    // public field
    pub field_a: i32,
    // private field
    field_b: i32,
}

// public enum
pub enum State {
    // public enum variants
    PubliclyAccessibleState,
    PubliclyAccessibleState2,
}
```

The visibility rules for private and public items are as follows:

1. If an item is private, it may be accessed by the current module and its
   descendants.
2. If an item is public, then it can be accessed from some module `m` if you
   can access all the item's ancestor modules from `m`. You may also be able to
   access the item through re-exports. See below.

---

Here are a few scenarios to demonstrate how you might apply these rules.

***I am developing a library, so I need a public API with hidden implementation details***

Anything you want exposed as part of your library's public API must be `pub`
from the root down to the item. Any private item in the chain will disallow
external accesses.

***I need a "helper module" with items that are only visible to the crate***

Create a private module at the root of the crate. Within that module, create a
"public API" by marking some items with `pub`. Since everything in the crate is
a descendant of the root, the entire crate will have access to this private
module.

> **Note:** Consider marking items with `pub(crate)` to ensure that they are
> not re-exported (see below).

***I am writing unit tests***

When writing unit tests for a module, a common idiom is to have an immediate
child of the module with tests named `mod test`. This module could access any
items of the parent module through the second case, meaning that internal
implementation details could also be seamlessly tested from the child module.

---

Here is an example program to exemplify the three scenarios outlined above:

```rust
// This module is private, meaning that no external crate can access this
// module. Because it is private at the root of this current crate, however, any
// module in the crate may access any publicly visible item in this module.
mod crate_helper_module {

    // This function can be used by anything in the current crate
    pub fn crate_helper() {}

    // This function *cannot* be used by anything else in the crate. It is not
    // publicly visible outside of the `crate_helper_module`, so only this
    // current module and its descendants may access it.
    fn implementation_detail() {}
}

// This function is "public to the root" meaning that it's available to external
// crates linking against this one.
pub fn public_api() {}

// Similarly to 'public_api', this module is public so external crates may look
// inside of it.
pub mod submodule {
    use crate::crate_helper_module;

    pub fn my_method() {
        // Any item in the local crate may invoke the helper module's public
        // interface through a combination of the two rules above.
        crate_helper_module::crate_helper();
    }

    // This function is hidden to any module which is not a descendant of
    // `submodule`
    fn my_implementation() {}

    #[cfg(test)]
    mod test {

        #[test]
        fn test_my_implementation() {
            // Because this module is a descendant of `submodule`, it's allowed
            // to access private items inside of `submodule` without a privacy
            // violation.
            super::my_implementation();
        }
    }
}

# fn main() {}
```

## `pub(in path)`, `pub(crate)`, `pub(super)`, and `pub(self)`

In addition to public and private, Rust allows users to declare an item as
visible only within a given scope. The rules for `pub` restrictions are as
follows:
- `pub(in path)` makes an item visible within the provided `path`. `path` must
be an ancestor module of the item whose visibility is being declared.
- `pub(crate)` makes an item visible within the current crate.
- `pub(super)` makes an item visible to the parent module. This is equivalent
  to `pub(in super)`.
- `pub(self)` makes an item visible to the current module. This is equivalent
to `pub(in self)` or not using `pub` at all.

> **Edition Differences**: Starting with the 2018 edition, paths for
> `pub(in path)` must start with `crate`, `self`, or `super`. The 2015 edition
> may also use paths starting with `::` or modules from the crate root.

Here's an example:

```rust,edition2015
pub mod outer_mod {
    pub mod inner_mod {
        // This function is visible within `outer_mod`
        pub(in crate::outer_mod) fn outer_mod_visible_fn() {}
        // Same as above, this is only valid in the 2015 edition.
        pub(in outer_mod) fn outer_mod_visible_fn_2015() {}

        // This function is visible to the entire crate
        pub(crate) fn crate_visible_fn() {}

        // This function is visible within `outer_mod`
        pub(super) fn super_mod_visible_fn() {
            // This function is visible since we're in the same `mod`
            inner_mod_visible_fn();
        }

        // This function is visible only within `inner_mod`,
        // which is the same as leaving it private.
        pub(self) fn inner_mod_visible_fn() {}
    }
    pub fn foo() {
        inner_mod::outer_mod_visible_fn();
        inner_mod::crate_visible_fn();
        inner_mod::super_mod_visible_fn();

        // This function is no longer visible since we're outside of `inner_mod`
        // Error! `inner_mod_visible_fn` is private
        //inner_mod::inner_mod_visible_fn();
    }
}

fn bar() {
    // This function is still visible since we're in the same crate
    outer_mod::inner_mod::crate_visible_fn();

    // This function is no longer visible since we're outside of `outer_mod`
    // Error! `super_mod_visible_fn` is private
    //outer_mod::inner_mod::super_mod_visible_fn();

    // This function is no longer visible since we're outside of `outer_mod`
    // Error! `outer_mod_visible_fn` is private
    //outer_mod::inner_mod::outer_mod_visible_fn();

    outer_mod::foo();
}

fn main() { bar() }
```

> **Note:** This syntax only adds another restriction to the visibility of an
> item. It does not guarantee that the item is visible within all parts of the
> specified scope. To access an item, all of its parent items up to the
> current scope must still be visible as well.

## Re-exporting and Visibility

Rust allows publicly re-exporting items through a `pub use` directive. Because
this is a public directive, this allows the item to be used in the current
module through the rules above. It essentially allows public access into the
re-exported item. For example, this program is valid:

```rust
pub use self::implementation::api;

mod implementation {
    pub mod api {
        pub fn f() {}
    }
}

# fn main() {}
```

This means that any external crate referencing `implementation::api::f` would
receive a privacy violation, while the path `api::f` would be allowed.

When re-exporting a private item, it can be thought of as allowing the "privacy
chain" being short-circuited through the reexport instead of passing through
the namespace hierarchy as it normally would.

[_SimplePath_]: paths.md#simple-paths
