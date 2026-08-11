# 18 Appendix C: Comparison with C and Rust

Aero sits between C and Rust: its syntax and memory feel like C, its type safety and borrowing ideas come from Rust, and it adds its own trade-offs (Arena, Tensor). Looking at it side by side with the language you know best is the fastest way to get up to speed.

## Variables

| Language | Syntax | Characteristics |
| --- | --- | --- |
| C | `int x = 1;` | Type must be written explicitly |
| Rust | `let x = 1;` | Type inference, immutable by default (`mut` to make it mutable) |
| Aero | `let x = 1;` | Type inference, **mutable by default**, no `const` |

Aero's `let` variables can be reassigned directly with `x = 2`, which is like C and unlike Rust (Rust requires `let mut x`).

## Integer Types

| C | Rust | Aero |
| --- | --- | --- |
| `int` | `i32` | `i32` |
| `long long` | `i64` | `i64` |
| — | `usize` | none (use `i64`) |
| `_Bool` | `bool` | `bool` |

Aero only has four basic types: `i32`/`i64`/`bool`/`str`. There's no `unsigned` and no floating point (in the 0.1 version). Integer literals default to `i64` and can be annotated to fit `i32`.

## Strings

| C | Rust | Aero |
| --- | --- | --- |
| `char*` (NUL-terminated) | `&str` (length-prefixed) | `str` (under the hood it's just `char*`) |

Aero's `str` is the same thing as C's `char*`, and can be passed directly to C functions (Chapter 12). The cost is that `len()` has to scan the whole string (O(n)). Aero provides richer string operations than C: concatenation, six comparisons, substring, search, and number conversions.

## Memory Management

| C | Rust | Aero |
| --- | --- | --- |
| `malloc` / `free`, all on you | Ownership + borrow checking, guaranteed at compile time | **Arena + borrow checking**, zero runtime pauses |

- C leaves memory entirely to you; get it wrong and you leak, crash, or corrupt memory.
- Rust uses ownership and borrowing, with the compiler watching over you, but the learning curve is steep.
- Aero takes a third path: **borrow checking (learned from Rust) + Arena (bulk-freed at block end)**. You don't need to `free` each thing, and there are no GC pauses. Runtime allocations (strings) still use the `malloc` family; free them with `str_free` when done.

## Borrowing

Aero's borrow rules are a simplified version of Rust's rules:

| Rule | Rust | Aero 0.1 |
| --- | --- | --- |
| Multiple immutable borrows coexist | ✅ | ✅ |
| Mutable borrow is exclusive | ✅ | ✅ |
| Can't write to source while borrowed | ✅ | ✅ |
| NLL (borrows live until last use) | ✅ | ✅ |
| References as return values | ✅ | ❌ (not yet supported) |
| References stored in arrays / nested | ✅ | ❌ (not yet supported) |

Aero implemented the most commonly used rules first; advanced scenarios are left for later versions.

## Functions and Generics

```c
// C: for every type, copy it again or use macros
int max_int(int a, int b) { return a > b ? a : b; }
```

```rust
// Rust: generics + trait bounds
fn max<T: Ord>(a: T, b: T) -> T { if a > b { a } else { b } }
```

```aero
// Aero: generics, checks operation legality at instantiation
fn max<T>(a: T, b: T) -> T {
    if (a > b) { return a; }
    return b;
}
```

All three are handled at compile time (both Aero and Rust use monomorphization). The difference: Rust uses trait bounds to constrain "what operations T supports", while Aero 0.1 simplifies this to "only check at instantiation", which is looser and surfaces errors later.

## Arrays

| C | Rust | Aero |
| --- | --- | --- |
| `int a[3]`, length is part of the type | `[i32; 3]`, length is part of the type | `[i64; 3]`, length is part of the type |

Neither C nor Aero does bounds checking (C by tradition, Aero by 0.1 trade-off); Rust's `[]` indexing does bounds checking by default (in debug). For tuples, Aero's `(10, true)` is similar to Rust's `(i64, bool)`, but Aero accesses it with `t[0]` (Rust uses `t.0`).

## An Overview Table

| Feature | C | Rust | Aero |
| --- | --- | --- | --- |
| Type inference | No | Yes | Yes |
| Garbage collection | No | No | No |
| Borrow checking | No | Yes | Yes (simplified) |
| Automatic memory management | No (manual) | Ownership | Arena |
| Built-in string operations | Few | Many | Many |
| Generics | No (macros) | Yes (traits) | Yes (loose) |
| FFI to C | Native | extern | extern "C" |
| AI matrices | No | No | `tensor` + `matmul` |
| Array bounds checking | No | Debug only | No (0.1) |
| Floating point | Yes | Yes | No (0.1) |

## If You're Coming from C

- Forget the daily `malloc`/`free`: common scenarios use Arena, strings use `str_free`.
- Type inference will spoil you, but once a type is set it won't silently narrow.
- `if (x)` no longer works; write `if (x != 0)`.
- You'll get stopped by the borrow checker a few times -- get used to it, it's catching real bugs.

## If You're Coming from Rust

- `let` is mutable by default, no need to write `mut` (but mutable borrows still need `&mut`).
- There's no concept of ownership transfer: arrays and tuples both have "copy" semantics, much simpler.
- The borrow rules are a subset of Rust's: references can't be returned, can't be nested.
- Memory uses Arena, no `Box`/`Vec`; strings need manual `str_free`.

## Closing

At this point, you've gone through all of Aero 0.1's capabilities: variables, control flow, functions, generics, arrays and tuples, strings, borrowing, Arena, Tensor, FFI, file IO, command line, packages and testing. What's left is to write code.

Aero is still young and missing quite a few things (floating point, break/continue, const, containers, classes, modules...). But the skeleton is right: do more work at compile time, keep the runtime clean. This book will update alongside the language.
