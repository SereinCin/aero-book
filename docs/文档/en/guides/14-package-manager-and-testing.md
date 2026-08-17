# 14 Package Manager and Testing

## What is a Package

From Chapter 1 until now, you've been writing **single-file programs**. Real projects need to split files, organize directories, and run tests. Aero's package system is what handles this.

A package is a directory with a fixed structure:

```
mytool/
├── Aero.toml        # Package descriptor file
└── src/
    └── main.aero    # Main program
```

## Creating a Package

```bash
aero new mytool
```

Automatically generated:

- `Aero.toml`: package name, version, edition;
- `src/main.aero`: a skeleton that prints `Hello, Aero!`.

Generated `Aero.toml`:

```toml
[package]
name = "mytool"
version = "0.1.0"
edition = "2024"
```

## Running and Building

Inside the package directory:

```bash
aero run          # Compile and run src/main.aero
aero build        # Compile into a standalone executable
```

`aero build` outputs to `target/aero/mytool.exe`. The build process is the same as a single file, just with the added concept of a package.

## Dependencies: Splitting Codebases

When a project grows, extract common code into a **library package** (function definitions only, no main), and reference it from the main package.

Suppose the `libs/mathlib` directory:

```toml
# libs/mathlib/Aero.toml
[package]
name = "mathlib"
version = "0.1.0"
```

```aero
// libs/mathlib/src/main.aero —— Library package, provides functions
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

The main program's `src/main.aero` directly calls the library's functions:

```aero
print(square(9));   // 81
```

During compilation, Aero **merges** the library code and the main program into a single source before compiling. Rules:

- Library packages provide functions, the main package calls them;
- Library packages **cannot have executable statements** at the top level (`print(...)` and similar statements will be rejected) — libraries only provide definitions;
- Function names must not conflict.

## Assertions: assert

```aero
assert(1 + 1 == 2);        // Condition is true, nothing happens
// assert(1 + 1 == 3);     // Fails: prints diagnostic info and terminates the program
```

When `assert(condition)` evaluates to false, it prints a diagnostic and terminates the program with a non-zero exit code. `assert_eq(a, b)` specifically checks equality:

```aero
assert_eq(add(2, 3), 5);   // The result of add must be 5
```

A failed assertion looks like this (program exits with non-zero code):

```
[assert] assertion failed
```

## Test Framework: test_ Prefix

Aero's testing convention: **functions whose names start with `test_`** are collected by `aero test` and run one by one.

```aero
// Tests written in main.aero
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

If any test fails (assertion failure), the corresponding function prints `FAIL`, and the final exit code is non-zero. CI pipelines rely on this to determine whether the build passes.

You can also test a single file:

```bash
aero test mytests.aero
```

Or place test files in the `tests/` directory (`.aero` files), and `aero test` will run all test files in that directory.

## A Complete Workflow

```bash
aero new calculator
cd calculator
# Edit src/main.aero, write functions and test_ tests
aero test        # Run tests first
aero run         # Then run
aero build       # Finally produce the exe
```

Suggested order for writing code: functions → tests → run → build.

## Exercises

1. Create a package with `aero new`, modify `main.aero` to implement `is_leap_year(y) -> i64` (leap year check), write two `test_` functions testing 2024 (yes) and 2023 (no), and run `aero test`.
2. Create a library package and a main package. The library provides `fact`, and the main package calls and prints `fact(6)`.
3. Put a `print(1);` statement in the library package, run `aero build`, read the error message, and understand that "library packages cannot have top-level statements."