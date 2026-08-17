# Aero Language Guide

## Introduction

Aero is a systems programming language designed for the AI era. It combines the ergonomics of Python with the performance of C++, delivering native-code speed, fine-grained memory control, and first-class support for automatic differentiation, tensor operations, and GPU programming.

Key design goals:

- **Performance**: Compiled to machine code via LLVM, with zero-cost abstractions and no garbage collector.
- **AI-native**: Built-in `tensor`, `gpu`, and `arena` keywords, plus an automatic differentiation (`Grad`) system in the standard library.
- **Safety**: Ownership-based memory management with a borrow checker, eliminating use-after-free and data races at compile time.
- **Interoperability**: Seamless FFI with C via `extern "C"` blocks.

---

## Installation & Setup

### Compiler

Download the Aero compiler from the official release page. The compiler binary is named `aero` (or `aero.exe` on Windows). Add the directory containing the binary to your `PATH`.

Verify the installation by running `aero`; it prints the full list of commands and usage.

```bash
aero
```

### VS Code Extension

Install the official Aero extension from the VS Code marketplace for syntax highlighting, code completion, and inline diagnostics.

### Project Scaffolding

Create a new project with:

```bash
aero new my_project
cd my_project
```

This generates the following structure:

```
my_project/
  Aero.toml
  src/
    main.aero
```

---

## Hello World

```aero
print("Hello, World!\n");
```

Save the code to `hello.aero`, then compile and run:

```bash
aero run hello.aero
```

---

## Basic Syntax

### Variables

Use `let` to bind a name to a value. Bindings are mutable by default; there is no `mut` keyword.

```aero
let x: i64 = 42;   // annotated binding
let y: f64 = 3.14; // bindings are mutable by default
y = 2.71;
```

Type annotations are optional when the type can be inferred:

```aero
let z = 10; // inferred as i64
```

### Functions

Functions are declared with `fn`. A function that produces a value declares its return type with `-> T` and uses an explicit `return`; functions that return nothing simply omit the return type.

```aero
fn add(a: i64, b: i64) -> i64 {
    return a + b;
}

fn factorial(n: i64) -> i64 {
    if (n <= 1) {
        return 1;
    }
    return n * factorial(n - 1);
}
```

### Conditionals

```aero
let x = 10;
if (x > 5) {
    print("big\n");
} else {
    print("small\n");
}
```

### Loops

`while` and `loop` are available. `loop` runs indefinitely until a `break`.

```aero
let i = 0;
while (i < 5) {
    print(int_to_str(i));
    i = i + 1;
}
```

```aero
let i = 0;
loop {
    if (i >= 5) { break; }
    print(int_to_str(i));
    i = i + 1;
}
```

`for` iterates over array literals or vectors:

```aero
for (i in [0, 1, 2, 3, 4]) {
    print(int_to_str(i));
}
```

---

## Composite Types

### Struct

```aero
struct Point {
    x: f64,
    y: f64,
}

let p = Point { x: 1.0, y: 2.0 };
print(format("p.x = %f\n", p.x));
```

### Enum

```aero
enum Color {
    Red,
    Green,
    Blue,
}
```

### Pattern Matching

```aero
enum Color {
    Red,
    Green,
    Blue,
}

let color = Color::Red;

match (color) {
    Color::Red => {
        print("red\n");
    }
    Color::Green => {
        print("green\n");
    }
    Color::Blue => {
        print("blue\n");
    }
}
```

### Generics

```aero
fn identity<T>(x: T) -> T {
    return x;
}
```

### Traits & Impl

```aero
struct Point {
    x: f64,
    y: f64,
}

trait Drawable {
    fn draw(x: Point) -> str;
}

impl Drawable for Point {
    fn draw(x: Point) -> str {
        return "drawing point";
    }
}
```

---

## Ownership & Borrowing

Aero uses a move semantics model. Every value has exactly one owner at any time.

### Move

```aero
let s1 = String::from("hello");
let s2 = s1; // s1 is moved, cannot be used afterwards
```

### Borrowing

Values are passed by value (moved). The compiler enforces ownership so that a value moved into a function can no longer be used by its previous owner.

```aero
fn length(s: str) -> i64 {
    return len(s);
}

fn greet(name: str) {
    print("hi ");
    print(name);
    print("\n");
}

let n = length("hello");
greet("aero");
```

### Ownership Rules

1. Each value in Aero has exactly one owner.
2. When the owner goes out of scope, the value is freed.
3. You may have either one mutable reference or any number of immutable references, but not both.
4. References must always be valid.

---

## Standard Library Overview

The standard library (`std.aero`) is auto-injected into every compilation unit.

| Type / Module | Description |
|---|---|
| `Option<T>` | Optional value: `Some(T)` or `None` |
| `Result<T, E>` | Fallible result: `Ok(T)` or `Err(E)` |
| `Vec<T>` | Dynamic array with indexing |
| `String` | UTF-8 string type |
| `HashMap<V>` | Hash table with `i64` keys |
| `BTreeMap` | Ordered map with `i64` keys |
| `BTreeSet` | Ordered set |
| `LinkedList<T>` | Doubly-linked list |
| `sort`, `binary_search`, `reverse` | Sorting utilities |
| `_filter_impl`, `_map_impl`, `_reduce_impl` | Functional combinators over `Vec<i64>` |
| `Path` | Path manipulation |
| `JSON` | JSON serialization helpers |
| `Grad` | Automatic differentiation |

---

## Project Structure

### Aero.toml

Every Aero project has a manifest file:

```toml
[package]
name = "my_project"
version = "0.1.0"

[link]
libs = ["m"]
```

The `[link]` section specifies native libraries to link against.

### Modules

Use `mod` to declare a module and `::` to access its items. Inline modules are written with braces.

```aero
mod math {
    fn add(a: i64, b: i64) -> i64 {
        return a + b;
    }
}

let r = math::add(1, 2);
print(int_to_str(r));
print("\n");
```

---

## Build & Test

| Command | Description |
|---|---|
| `aero new <name>` | Create a new project |
| `aero build` | Compile the project |
| `aero run` | Build and execute |
| `aero test` | Run tests |
| `aero fmt` | Format source code |
| `aero clippy` | Lint source code |
| `aero cov` | Code coverage |
| `aero bench` | Run benchmarks |
| `aero lsp` | Start the language server |

---

## FFI

Aero can call C functions by declaring them with `extern "C" fn`.

```aero
extern "C" fn puts(s: str) -> i32;

puts("hello from C");
let r = rand();
print(format("random: %lld\n", r));
```

Link against native libraries in `Aero.toml`:

```toml
[link]
libs = ["m", "curl"]
```

---

## Builtin Function Quick Reference

| Function | Description |
|---|---|
| `print(s)` | Print a string to stdout |
| `assert(cond)` | Panic if `cond` is false |
| `assert_eq(a, b)` | Panic if `a != b` |
| `len(x)` | Return length of a collection |
| `int_to_str(n)` | Convert integer to string |
| `str_to_int(s)` | Parse string to integer |
| `str_contains(s, sub)` | Check substring |
| `str_find(s, sub)` | Find substring index |
| `str_cmp(a, b)` | Compare strings |
| `read_file(path)` | Read file contents |
| `write_file(path, data)` | Write to file |
| `format(fmt, ...)` | Format string |
| `hash_i64(n)` | Hash an integer |
| `matmul(a, b)` | Matrix multiplication |
| `tensor_add(a, b)` | Tensor addition |

---

## Official Core Crates Overview

| Crate | Description |
|---|---|
| `aero-json` | JSON parsing and value access |
| `aero-csv` | CSV record scanning |
| `aero-logger` | Structured logging with levels |
| `aero-test` | Unit testing utilities |
| `aero-collections` | Stack and Queue data structures |
| `aero-time` | Timestamp and formatting |
| `aero-io` | File I/O helpers |
| `aero-toml` | TOML line parsing |
| `aero-http` | HTTP client (libcurl wrapper) |
| `aero-crypto` | Hashing and crypto utilities |
| `aero-regex` | Regular expression matching |
| `aero-net` | TCP socket networking (WS2_32) |