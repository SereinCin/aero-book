# 18 Appendix C: Comparison with C and Rust

Aero sits between C and Rust: the syntax and memory feel are like C, while type safety and borrow ideas come from Rust, with its own trade-offs (Arena, Tensor). Comparing with the language you're most familiar with is the fastest way to get started.

## Variables

| Language | Syntax | Feature |
| --- | --- | --- |
| C | `int x = 1;` | Must specify type |
| Rust | `let x = 1;` | Type inference, immutable by default (`mut` for mutability) |
| Aero | `let x = 1;` | Type inference, **mutable by default**, no `const` |

Aero's `let` variables can be reassigned directly with `x = 2`, which is like C and unlike Rust (Rust requires `let mut x`).

## Integer Types

| C | Rust | Aero |
| --- | --- | --- |
| `int` | `i32` | `i32` |
| `long long` | `i64` | `i64` |
| — | `usize` | None (use `i64`) |
| `_Bool` | `bool` | `bool` |

Aero has only four basic types: `i32`/`i64`/`bool`/`str`. No `unsigned`, no floating point (version 1.1.0). Integer literals default to `i64` and can be annotated to adapt to `i32`.

## Strings

| C | Rust | Aero |
| --- | --- | --- |
| `char*` (NUL-terminated) | `&str` (length-prefixed) | `str` (underlying `char*`) |

Aero's `str` is the same as C's `char*` and can be passed directly to C functions (Chapter 12). The trade-off is that `len()` requires scanning the string (O(n)). Aero provides richer string operations than C: concatenation, six kinds of comparison, substring, search, and number conversion.

## Memory Management

| C | Rust | Aero |
| --- | --- | --- |
| `malloc` / `free`,全靠自觉 | Ownership + Borrow Check, guaranteed at compile time | **Arena + Borrow Check**, zero runtime pauses |

- C leaves memory entirely to you — mistakes lead to leaks, crashes, or memory corruption.
- Rust uses ownership and borrowing, with the compiler watching, but the learning curve is steep.
- Aero takes a third path: **Borrow Check (from Rust) + Arena (bulk deallocation at block end)**. You don't need to `free` each piece individually, and there are no GC pauses. Runtime allocations (strings) still use `malloc` under the hood, with `str_free` to clean up.

## Borrowing

Aero's borrow rules are a simplified version of Rust's rules:

| Rule | Rust | Aero 1.1.0 |
| --- | --- | --- |
| Multiple immutable borrows coexist | ✅ | ✅ |
| Mutable borrow is exclusive | ✅ | ✅ |
| Cannot write source while borrowed | ✅ | ✅ |
| NLL (borrow lives until last use) | ✅ | ✅ |
| References as return values | ✅ | ❌ (not yet supported) |
| References stored in arrays/nested | ✅ | ❌ (not yet supported) |

Aero implements the most commonly used rules first, leaving advanced scenarios for future versions.

## Functions and Generics

```c
// C: write one version per type, or use macros
int max_int(int a, int b) { return a > b ? a : b; }
```

```rust
// Rust: generics + trait bounds
fn max<T: Ord>(a: T, b: T) -> T { if a > b { a } else { b } }
```

```aero
// Aero: generics, operation validity checked at instantiation
fn max<T>(a: T, b: T) -> T {
    if (a > b) { return a; }
    return b;
}
```

All three are handled at compile time (Aero and Rust both use monomorphization). The difference: Rust uses traits to constrain "what operations T supports," while Aero 1.1.0 simplifies this to "check only at instantiation" — looser constraints, errors caught later.

## Arrays

| C | Rust | Aero |
| --- | --- | --- |
| `int a[3]`, length is part of type | `[i32; 3]`, length is part of type | `[i64; 3]`, length is part of type |

C and Aero do not perform bounds checking (C by tradition, Aero by trade-off for version 1.1.0); Rust's `[]` indexing does bounds checking by default (in debug mode). For tuples, Aero's `(10, true)` is similar to Rust's `(i64, bool)`, but Aero uses `t[0]` for access (Rust uses `t.0`).

## Overview Table

| Feature | C | Rust | Aero |
| --- | --- | --- | --- |
| Type Inference | No | Yes | Yes |
| Garbage Collection | No | No | No |
| Borrow Check | No | Yes | Yes (simplified) |
| Automatic Memory Management | No (manual) | Ownership | Arena |
| Built-in String Operations | Few | Many | Many |
| Generics | No (macros) | Yes (traits) | Yes (relaxed) |
| FFI to C | Native | extern | extern "C" |
| AI Matrix | No | No | `tensor` + `matmul` |
| Array Bounds Check | No | In debug | No (1.1.0) |
| Floating Point | Yes | Yes | No (1.1.0) |

## If You're Coming from C

- Forget about `malloc`/`free` as a daily routine: use Arena for common scenarios, and `str_free` for strings.
- Type inference will spoil you, but once a type is set, it won't silently narrow.
- `if (x)` no longer works — you need to write `if (x != 0)`.
- You'll be stopped by the borrow checker a few times, but you'll get used to it — it catches real bugs.

## If You're Coming from Rust

- `let` is mutable by default — no need to write `mut` (but mutable borrows still require `&mut`).
- There is no concept of ownership transfer: arrays and tuples use "copy" semantics, which is much simpler.
- The borrow rules are a subset of Rust's: references cannot be returned or nested.
- Memory management: Arena provides block allocation with no GC pauses, `Vec` provides dynamic arrays, and strings require manual `str_free`.

## Closing

At this point, you've gone through all of Aero's 1.1.0 capabilities: variables, control flow, functions, generics, arrays and tuples, strings, borrowing, Arena, Tensor, FFI, file I/O, command line, packages and testing. What's left is to write code.

Aero is still young, and there's plenty missing (floating point, break/continue, const, containers, classes, modules...). But the skeleton is right: do more work at compile time, keep the runtime clean. This book will be updated alongside the language.