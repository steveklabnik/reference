# Code generation attributes

The following [attributes] are used for controlling code generation.

## Optimization hints

The `cold` and `inline` [attributes] give suggestions to generate code in a
way that may be faster than what it would do without the hint. The attributes
are only hints, and may be ignored.

Both attributes can be used on [functions]. When applied to a function in a
[trait], they apply only to that function when used as a default function for
a trait implementation and not to all trait implementations. The attributes
have no effect on a trait function without a body.

### The `inline` attribute

The *`inline` [attribute]* suggests that a copy of the attributed function
should be placed in the caller, rather than generating code to call the
function where it is defined.

> ***Note***: The `rustc` compiler automatically inlines functions based on
> internal heuristics. Incorrectly inlining functions can make the program
> slower, so this attribute should be used with care.

There are three ways to use the inline attribute:

* `#[inline]` *suggests* performing an inline expansion.
* `#[inline(always)]` *suggests* that an inline expansion should always be
  performed.
* `#[inline(never)]` *suggests* that an inline expansion should never be
  performed.

> ***Note***: `#[inline]` in every form is a hint, with no *requirements*
> on the language to place a copy of the attributed function in the caller.

### The `cold` attribute

The *`cold` [attribute]* suggests that the attributed function is unlikely to
be called.

## The `no_builtins` attribute

The *`no_builtins` [attribute]* may be applied at the crate level to disable
optimizing certain code patterns to invocations of library functions that are
assumed to exist.

## The `target_feature` attribute

The *`target_feature` [attribute]* may be applied to a function to
enable code generation of that function for specific platform architecture
features. It uses the [_MetaListNameValueStr_] syntax with a single key of
`enable` whose value is a string of comma-separated feature names to enable.

```rust
# #[cfg(target_feature = "avx2")]
#[target_feature(enable = "avx2")]
unsafe fn foo_avx2() {}
```

Each [target architecture] has a set of features that may be enabled. It is an
error to specify a feature for a target architecture that the crate is not
being compiled for.

It is [undefined behavior] to call a function that is compiled with a feature
that is not supported on the current platform the code is running on.

Functions marked with `target_feature` are not inlined into a context that
does not support the given features. The `#[inline(always)]` attribute may not
be used with a `target_feature` attribute.

### Available features

The following is a list of the available feature names.

#### `x86` or `x86_64`

Executing code with unsupported features is UB on this platform.
Hence this platform requires that `#[target_feature]` is only applied to [`unsafe`
functions][unsafe function].

Feature     | Implicitly Enables | Description
------------|--------------------|-------------------
`aes`       | `sse2`   | [AES] — Advanced Encryption Standard
`avx`       | `sse4.2` | [AVX] — Advanced Vector Extensions
`avx2`      | `avx`    | [AVX2] — Advanced Vector Extensions 2
`bmi1`      |          | [BMI1] — Bit Manipulation Instruction Sets
`bmi2`      |          | [BMI2] — Bit Manipulation Instruction Sets 2
`fma`       | `avx`    | [FMA3] — Three-operand fused multiply-add
`fxsr`      |          | [`fxsave`] and [`fxrstor`] — Save and restore x87 FPU, MMX Technology, and SSE State
`lzcnt`     |          | [`lzcnt`] — Leading zeros count
`pclmulqdq` | `sse2`   | [`pclmulqdq`] — Packed carry-less multiplication quadword
`popcnt`    |          | [`popcnt`] — Count of bits set to 1
`rdrand`    |          | [`rdrand`] — Read random number
`rdseed`    |          | [`rdseed`] — Read random seed
`sha`       | `sse2`   | [SHA] — Secure Hash Algorithm
`sse`       |          | [SSE] — Streaming <abbr title="Single Instruction Multiple Data">SIMD</abbr> Extensions
`sse2`      | `sse`    | [SSE2] — Streaming SIMD Extensions 2
`sse3`      | `sse2`   | [SSE3] — Streaming SIMD Extensions 3
`sse4.1`    | `ssse3`  | [SSE4.1] — Streaming SIMD Extensions 4.1
`sse4.2`    | `sse4.1` | [SSE4.2] — Streaming SIMD Extensions 4.2
`ssse3`     | `sse3`   | [SSSE3] — Supplemental Streaming SIMD Extensions 3
`xsave`     |          | [`xsave`] — Save processor extended states
`xsavec`    |          | [`xsavec`] — Save processor extended states with compaction
`xsaveopt`  |          | [`xsaveopt`] — Save processor extended states optimized
`xsaves`    |          | [`xsaves`] — Save processor extended states supervisor

<!-- Keep links near each table to make it easier to move and update. -->

[AES]: https://en.wikipedia.org/wiki/AES_instruction_set
[AVX]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions
[AVX2]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX2
[BMI1]: https://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets
[BMI2]: https://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets#BMI2
[FMA3]: https://en.wikipedia.org/wiki/FMA_instruction_set
[`fxsave`]: https://www.felixcloutier.com/x86/fxsave
[`fxrstor`]: https://www.felixcloutier.com/x86/fxrstor
[`lzcnt`]: https://www.felixcloutier.com/x86/lzcnt
[`pclmulqdq`]: https://www.felixcloutier.com/x86/pclmulqdq
[`popcnt`]: https://www.felixcloutier.com/x86/popcnt
[`rdrand`]: https://en.wikipedia.org/wiki/RdRand
[`rdseed`]: https://en.wikipedia.org/wiki/RdRand
[SHA]: https://en.wikipedia.org/wiki/Intel_SHA_extensions
[SSE]: https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions
[SSE2]: https://en.wikipedia.org/wiki/SSE2
[SSE3]: https://en.wikipedia.org/wiki/SSE3
[SSE4.1]: https://en.wikipedia.org/wiki/SSE4#SSE4.1
[SSE4.2]: https://en.wikipedia.org/wiki/SSE4#SSE4.2
[SSSE3]: https://en.wikipedia.org/wiki/SSSE3
[`xsave`]: https://www.felixcloutier.com/x86/xsave
[`xsavec`]: https://www.felixcloutier.com/x86/xsavec
[`xsaveopt`]: https://www.felixcloutier.com/x86/xsaveopt
[`xsaves`]: https://www.felixcloutier.com/x86/xsaves

#### `wasm32` or `wasm64`

Executing code with unsupported features is allowed (i.e., is not UB) on this platform.
Hence this platform allows `#[target_feature]` to be applied to both safe and
[`unsafe` functions][unsafe function].

Feature     | Description
------------|-------------------
`simd128`   | [WebAssembly simd proposal][simd128]

[simd128]: https://github.com/webassembly/simd

### Additional information

See the [`target_feature` conditional compilation option] for selectively
enabling or disabling compilation of code based on compile-time settings. Note
that this option is not affected by the `target_feature` attribute, and is
only driven by the features enabled for the entire crate.

See the [`is_x86_feature_detected`] macro in the standard library for runtime
feature detection on the x86 platforms.

> Note: `rustc` has a default set of features enabled for each target and CPU.
> The CPU may be chosen with the [`-C target-cpu`] flag. Individual features
> may be enabled or disabled for an entire crate with the
> [`-C target-feature`] flag.

## The `track_caller` attribute

The `track_caller` attribute may be applied to any function with [`"Rust"` ABI][rust-abi]
with the exception of the entry point `fn main`. When applied to functions and methods in
trait declarations, the attribute applies to all implementations. If the trait provides a
default implementation with the attribute, then the attribute also applies to override implementations.

When applied to a function in an `extern` block the attribute must also be applied to any linked
implementations, otherwise undefined behavior results. When applied to a function which is made
available to an `extern` block, the declaration in the `extern` block must also have the attribute,
otherwise undefined behavior results.

### Behavior

Applying the attribute to a function `f` allows code within `f` to get a hint of the [`Location`] of
the "topmost" tracked call that led to `f`'s invocation. At the point of observation, an
implementation behaves as if it walks up the stack from `f`'s frame to find the nearest frame of an
*unattributed* function `outer`, and it returns the [`Location`] of the tracked call in `outer`.

```rust
#[track_caller]
fn f() {
    println!("{}", std::panic::Location::caller());
}
```

> Note: `core` provides [`core::panic::Location::caller`] for observing caller locations. It wraps
> the [`core::intrinsics::caller_location`] intrinsic implemented by `rustc`.

> Note: because the resulting `Location` is a hint, an implementation may halt its walk up the stack
> early. See [Limitations](#limitations) for important caveats.

#### Examples

When `f` is called directly by `calls_f`, code in `f` observes its callsite within `calls_f`:

```rust
# #[track_caller]
# fn f() {
#     println!("{}", std::panic::Location::caller());
# }
fn calls_f() {
    f(); // <-- f() prints this location
}
```

When `f` is called by another attributed function `g` which is in turn called by `calls_g`, code in
both `f` and `g` observes `g`'s callsite within `calls_g`:

```rust
# #[track_caller]
# fn f() {
#     println!("{}", std::panic::Location::caller());
# }
#[track_caller]
fn g() {
    println!("{}", std::panic::Location::caller());
    f();
}

fn calls_g() {
    g(); // <-- g() prints this location twice, once itself and once from f()
}
```

When `g` is called by another attributed function `h` which is in turn called by `calls_h`, all code
in `f`, `g`, and `h` observes `h`'s callsite within `calls_h`:

```rust
# #[track_caller]
# fn f() {
#     println!("{}", std::panic::Location::caller());
# }
# #[track_caller]
# fn g() {
#     println!("{}", std::panic::Location::caller());
#     f();
# }
#[track_caller]
fn h() {
    println!("{}", std::panic::Location::caller());
    g();
}

fn calls_h() {
    h(); // <-- prints this location three times, once itself, once from g(), once from f()
}
```

And so on.

### Limitations

This information is a hint and implementations are not required to preserve it.

In particular, coercing a function with `#[track_caller]` to a function pointer creates a shim which
appears to observers to have been called at the attributed function's definition site, losing actual
caller information across virtual calls. A common example of this coercion is the creation of a
trait object whose methods are attributed.

> Note: The aforementioned shim for function pointers is necessary because `rustc` implements
> `track_caller` in a codegen context by appending an implicit parameter to the function ABI, but
> this would be unsound for an indirect call because the parameter is not a part of the function's
> type and a given function pointer type may or may not refer to a function with the attribute. The
> creation of a shim hides the implicit parameter from callers of the function pointer, preserving
> soundness.

[_MetaListNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[`-C target-cpu`]: ../../rustc/codegen-options/index.html#target-cpu
[`-C target-feature`]: ../../rustc/codegen-options/index.html#target-feature
[`is_x86_feature_detected`]: ../../std/macro.is_x86_feature_detected.html
[`target_feature` conditional compilation option]: ../conditional-compilation.md#target_feature
[attribute]: ../attributes.md
[attributes]: ../attributes.md
[functions]: ../items/functions.md
[target architecture]: ../conditional-compilation.md#target_arch
[trait]: ../items/traits.md
[undefined behavior]: ../behavior-considered-undefined.md
[unsafe function]: ../unsafe-functions.md
[rust-abi]: ../items/external-blocks.md#abi
[`core::intrinsics::caller_location`]: ../../core/intrinsics/fn.caller_location.html
[`core::panic::Location::caller`]: ../../core/panic/struct.Location.html#method.caller
[`Location`]: ../../core/panic/struct.Location.html
