# 14 Package Manager and Testing

## What is a Package

From Chapter 1 until now, you've been writing **single-file programs**. Real projects need to split files, organize directories, and run tests. Aero's package does exactly this.

A package is just a directory with a fixed structure:

```
mytool/
├── Aero.toml        # package manifest
└── src/
    └── main.aero    # main program
```

## Creating a Package

```bash
aero new mytool
```

This auto-generates:

- `Aero.toml`: package name, version, edition;
- `src/main.aero`: a skeleton that prints `Hello, Aero!`.

The generated `Aero.toml`:

```toml
[package]
name = "mytool"
version = "0.1.0"
edition = "2024"
```

## Run and Build

Inside the package directory:

```bash
aero run          # compile and run src/main.aero
aero build        # compile into a standalone executable
```

The output of `aero build` is in `target/aero/mytool.exe`. The whole build flow is the same as for a single file, just with the package concept added on top.

## Dependencies: Splitting Code into Libraries

When the project grows, you factor common code into a **library package** (only function definitions, no main), and the main package references it.

Suppose there's a `libs/mathlib` directory:

```toml
# libs/mathlib/Aero.toml
[package]
name = "mathlib"
version = "0.1.0"
```

```aero
// libs/mathlib/src/main.aero -- library package, provides functions
fn square(x: i64) -> i64 {
    return x * x;
}
```

The main package declares the dependency in `Aero.toml`:

```toml
[package]
name = "mytool"
version = "0.1.0"

[dependencies]
mathlib = { path = "../libs/mathlib" }
```

The main program's `src/main.aero` directly calls functions from the library:

```aero
print(square(9));   // 81
```

At compile time, Aero **merges the dependency library's code with the main program into a single source** and then compiles it. The rules:

- The library package provides functions, the main package calls them;
- The library package's top level **cannot have executable statements** (statements like `print(...)` will be rejected) -- libraries only provide definitions;
- Function names cannot be duplicated.

## Assertions: assert

```aero
assert(1 + 1 == 2);        // condition is true, nothing happens
// assert(1 + 1 == 3);     // fails: prints diagnostic info and terminates the program
```

`assert(condition)` prints a diagnostic and terminates the program with a non-zero exit code when the condition is false. `assert_eq(a, b)` is specialized for equality:

```aero
assert_eq(add(2, 3), 5);   // the result of add must be 5
```

An assertion failure looks like this (program exits with non-zero code):

```
[assert] assertion failed
```

## Test Framework: the test_ Prefix

Aero's testing convention: **any function whose name starts with `test_`** will be collected by `aero test` and run one by one.

```aero
// tests are written in main.aero
fn test_square() {
    assert_eq(square(4), 16);
    assert_eq(square(-3), 9);
}

fn test_add() {
    assert_eq(add(1, 2), 3);
}
```

Run:

```bash
aero test
```

Output:

```
PASS  test_square
PASS  test_add
2 tests: 2 passed, 0 failed
```

If any test fails (assertion fails), that function prints `FAIL`, and the final exit code is non-zero. CI relies on this to judge whether the build passes.

You can also test a single file:

```bash
aero test mytests.aero
```

Or put test files in the `tests/` directory (`.aero` files), and `aero test` will run all test files in that directory.

## A Complete Workflow

```bash
aero new calculator
cd calculator
# edit src/main.aero, write functions and test_ tests
aero test        # run tests first
aero run         # then run
aero build       # finally produce the exe
```

Suggested order for writing code: function → test → run → build.

## Exercises

1. Use `aero new` to create a package, modify `main.aero` to implement `is_leap_year(y) -> i64` (leap year check), write two `test_` functions testing 2024 (yes) and 2023 (no), run `aero test`.
2. Create a library package and a main package; the library provides `fact`, the main package calls it and prints `fact(6)`.
3. Put a line `print(1);` in the library package, run `aero build`, read the error message, and understand "library packages cannot have top-level statements".
