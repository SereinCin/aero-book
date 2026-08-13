# Aero Programming Language — The Official Book

> **Aero** is a systems programming language that combines **Python-level development ergonomics** with **C++-level runtime performance**. It uses an **LLVM backend** for both JIT and AOT compilation, ships with a **borrow checker** (compile-time dangling-pointer and data-race prevention), offers **Arena-based region memory management** (zero GC pauses, no per-object free), and provides **native tensor / matrix (matmul) support** for AI computing workloads.

This repository hosts the complete beginner-friendly tutorial book for the Aero programming language. After reading all 18 chapters you will be able to write complete, runnable Aero programs that compile into standalone Windows executables.

- **Aero version covered**: `0.1.1` (2026)
- **Live GitHub Pages site**: https://sereincin.github.io/aero-book/
- **Aero compiler repository**: https://github.com/SereinCin/aero-lang
- **Language**: English (translated from the original Chinese edition)

---

## Table of Contents

| Chapter | Title | Keywords Covered |
| --- | --- | --- |
| 00 | [Preface](00-preface.md) | design goals, three no-GC principles, compile-time vs runtime trade-offs |
| 01 | [Environment and First Program](01-environment-and-first-program.md) | Rust toolchain, LLVM 22, MinGW UCRT64 gcc, `aero run` JIT, `aero build` AOT standalone exe |
| 02 | [Variables and Types](02-variables-and-types.md) | `let` declaration, `i64`/`i32`/`bool`/`str`, type inference, integer literal adaptation, implicit narrowing rejection, block scope |
| 03 | [Operators and Expressions](03-operators-and-expressions.md) | arithmetic, integer division, short-circuit `&&`/`||`, `printf`-style format strings (`%d`, `%s`) |
| 04 | [Control Flow](04-flow-control.md) | `if`/`else if`/`else`, `while` loops, modulus `%`, simulated `break`/`continue` patterns |
| 05 | [Functions](05-functions.md) | `fn` keyword, pass-by-value parameters, return values, recursion, forward-calling (two-pass compilation), no nested definitions |
| 06 | [Generics](06-generics.md) | `<T>` type parameters, monomorphization, multi-type parameters `<A,B>`, generic arrays, compile-time instantiation checking |
| 07 | [Arrays and Tuples](07-arrays-and-tuples.md) | `[T; N]` fixed-length arrays, tuple `(A,B)`, compile-time tuple-index range checking, `divmod` pattern |
| 08 | [Strings](08-strings.md) | `str` type, concatenation `+`, dictionary-order comparison `<`/`>`/`==`, `len`, `substr`, `int_to_str`, `str_to_int`, `str_contains`, `str_find`, `str_cmp`, `str_free` ownership rules |
| 09 | [References and Borrowing](09-references-and-borrowing.md) | `&T` immutable borrow, `&mut T` mutable borrow, NLL non-lexical lifetimes, borrow rules, compile-time safety guarantees |
| 10 | [Arena Memory Management](10-arena-memory-management.md) | `arena(N)`, `alloc(n)`, `reset()`, scope-based auto-release, region allocation vs malloc vs GC, zero fragmentation |
| 11 | [Tensors and Matrices](11-tensors-and-matrices.md) | `tensor(rows, cols)`, subtensors, `matmul`, compile-time dimension validation, GPU kernel `extern "gpu"` hooks |
| 12 | [FFI: Calling C](12-ffi-calling-c.md) | `extern "C"` declarations, symbol alias `= "c_symbol"`, C ABI type compatibility, Aero.toml `[link]` libs & lib_paths, linking MinGW static archives |
| 13 | [File IO and CLI Arguments](13-file-io-and-cli-args.md) | `read_file`, `write_file`, `arg_count`, `arg(i)`, simple grep utility example |
| 14 | [Package Manager and Testing](14-package-manager-and-testing.md) | `aero new`, `Aero.toml`, `src/main.aero`, path-based `[dependencies]`, library packages (no top-level statements), `assert`, `assert_eq`, `aero test` with `test_` prefix convention |
| 15 | [Compilation Pipeline Overview](15-compilation-pipeline.md) | lex → parse → AST → HIR (name resolution + type inference + borrow check) → LLVM IR → O2 optimization → JIT/AOT machine code |
| 16 | [Appendix A — Built-in Function Reference](16-appendix-a-builtin-reference.md) | print, assert/assert_eq, string utilities, matmul, file IO, CLI args, operator precedence table |
| 17 | [Appendix B — Common Errors and Solutions](17-appendix-b-common-errors.md) | parse errors, type mismatch, name resolution, borrow conflicts, tuple indexing, string leaks, arena bounds, C FFI type rules, matmul dimension mismatch, generic name conflicts |
| 18 | [Appendix C — Comparison with C and Rust](18-appendix-c-comparison-with-c-and-rust.md) | side-by-side feature matrix for variables, integer types, strings, memory model, borrowing rules, generics, arrays, AI features, and migration guidance |

---

## Why Aero Exists

Most systems languages force a hard choice: either **ergonomics** (Python, JavaScript) but slow and interpreted, or **raw speed** (C, C++) but manual memory footguns and undefined behavior. Rust brought safety but with a steep ownership learning curve.

Aero takes a third path:

1. **No garbage collector** — no STW pauses, no runtime heap bookkeeping overhead
2. **Do it at compile time** — type inference, borrow checking, constant folding, LLVM O2 passes
3. **Don't reinvent low-level wheels** — delegate machine code generation, register allocation, and instruction scheduling to LLVM (20+ years of production optimization)

The result is a statically typed, compiled language where you write:

```aero
fn max<T>(a: T, b: T) -> T {
    if (a > b) { return a; }
    return b;
}
print(max(3, 7));   // 7
```

and get **zero-runtime-overhead monomorphized native code** — no virtual dispatch, no reflection, no GC.

---

## How to Use This Book

- **Type along as you read.** All example Aero code has been verified; typing it yourself builds muscle memory.
- **Finish every chapter's exercises** before advancing. Concepts in Chapters 9 (borrowing) and 10 (Arena) build cumulatively.
- **Windows is the primary target** for Aero 0.1; macOS / Linux users only need to swap the installation steps (Homebrew Rust, `brew install llvm@22`) — the syntax is identical.
- **Run the examples JIT-first** with `aero run file.aero` for quick iteration; use `aero build file.aero` to ship a portable `.exe`.

## Related Links

- 🔗 **[Aero Compiler Source (SereinCin/aero-lang)](https://github.com/SereinCin/aero-lang)** — Rust-based compiler workspace: `aero-lex`, `aero-parse`, `aero-hir`, `aero-ir`, `aero-pm`, `aero-cli`
- 🔗 **[Live Book on GitHub Pages](https://sereincin.github.io/aero-book/)** — rendered version with SEO-friendly HTML metadata, sitemap, and Schema.org Book structured data
