# Keywords

Rust divides keywords into three categories:

* [strict](#strict-keywords)
* [reserved](#reserved-keywords)
* [weak](#weak-keywords)

## Strict keywords

These keywords can only be used in their correct contexts. They cannot
be used as the names of:

* [Items]
* [Variables] and function parameters
* Fields and [variants]
* [Type parameters]
* Lifetime parameters or [loop labels]
* [Macros] or [attributes]
* [Macro placeholders]
* [Crates]

> **<sup>Lexer:<sup>**\
> KW_AS             : `as`\
> KW_ASYNC          : `async`\
> KW_AWAIT          : `await`\
> KW_BREAK          : `break`\
> KW_CONST          : `const`\
> KW_CONTINUE       : `continue`\
> KW_CRATE          : `crate`\
> KW_DYN            : `dyn` \
> KW_ELSE           : `else`\
> KW_ENUM           : `enum`\
> KW_EXTERN         : `extern`\
> KW_FALSE          : `false`\
> KW_FN             : `fn`\
> KW_FOR            : `for`\
> KW_IF             : `if`\
> KW_IMPL           : `impl`\
> KW_IN             : `in`\
> KW_LET            : `let`\
> KW_LOOP           : `loop`\
> KW_MATCH          : `match`\
> KW_MOD            : `mod`\
> KW_MOVE           : `move`\
> KW_MUT            : `mut`\
> KW_PUB            : `pub`\
> KW_REF            : `ref`\
> KW_RETURN         : `return`\
> KW_SELFVALUE      : `self`\
> KW_SELFTYPE       : `Self`\
> KW_STATIC         : `static`\
> KW_STRUCT         : `struct`\
> KW_SUPER          : `super`\
> KW_TRAIT          : `trait`\
> KW_TRUE           : `true`\
> KW_TYPE           : `type`\
> KW_UNSAFE         : `unsafe`\
> KW_USE            : `use`\
> KW_WHERE          : `where`\
> KW_WHILE          : `while`

> **Edition Differences**: In the 2015 edition, `async` and `await` are not
> keywords, and `dyn` is not a strict keyword.


## Reserved keywords

These keywords aren't used yet, but they are reserved for future use. They have
the same restrictions as strict keywords. The reasoning behind this is to make
current programs forward compatible with future versions of Rust by forbidding
them to use these keywords.

> **<sup>Lexer</sup>**\
> KW_ABSTRACT       : `abstract`\
> KW_BECOME         : `become`\
> KW_BOX            : `box`\
> KW_DO             : `do`\
> KW_FINAL          : `final`\
> KW_MACRO          : `macro`\
> KW_OVERRIDE       : `override`\
> KW_PRIV           : `priv`\
> KW_TRY            : `try`\
> KW_TYPEOF         : `typeof`\
> KW_UNSIZED        : `unsized`\
> KW_VIRTUAL        : `virtual`\
> KW_YIELD          : `yield`

> **Edition Differences**: In the 2015 edition, `try` is not a (reserved)
> keyword.


## Weak keywords

These keywords have special meaning only in certain contexts. For example, it
is possible to declare a variable or method with the name `union`.

* `union` is used to declare a [union] and is only a keyword when used in a
  union declaration.
* `'static` is used for the static lifetime and cannot be used as a generic
  lifetime parameter

  ```compile_fail
  // error[E0262]: invalid lifetime parameter name: `'static`
  fn invalid_lifetime_parameter<'static>(s: &'static str) -> &'static str { s }
  ```

> **<sup>Lexer</sup>**\
> KW_UNION          : `union`\
> KW_STATICLIFETIME : `'static`
>

> **Edition Differences**: In the 2015 edition, [`dyn`] is an additional weak
> keyword: it is a keyword when used in a type position followed by a path
> that does not start with `::`.


[items]: items.md
[Variables]: variables.md
[Type parameters]: types/parameters.md
[loop labels]: expressions/loop-expr.md#loop-labels
[Macros]: macros.md
[attributes]: attributes.md
[Macro placeholders]: macros-by-example.md
[Crates]: crates-and-source-files.md
[union]: items/unions.md
[variants]: items/enumerations.md
[`dyn`]: types/trait-object.md
