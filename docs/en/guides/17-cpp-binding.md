# 17 C++ Binding (bindgen)

Aero can compile not only to standalone executables, Python extensions, and Android/iOS shared libraries, but also be **called directly from C++ projects**. `aero build --cpp` generates two things at once:

- **A shared library** (`.dll` / `.so` / `.dylib`) containing the C ABI symbols of every `#[export]` function;
- **A C++ header** (`<name>.hpp`) with `extern "C"` declarations of those functions, including type mappings.

The C++ side just `#include`s the header and links the library to call Aero functions — no glue layer, no hand-written bridging. This resembles the `cxx` crate in the Rust ecosystem, but oriented toward "C++ calling Aero" (v1).

## Quick Start

```aero
// calc.aero
#[export]
fn add(a: i64, b: i64) -> i64 {
    return a + b;
}

#[export]
fn double(x: f64) -> f64 {
    return x * 2.0;
}

#[export]
fn is_even(n: i64) -> bool {
    if (n % 2 == 0) {
        return true;
    }
    return false;
}

extern "C" fn strlen(s: str) -> i64;

#[export]
fn str_len(s: str) -> i64 {
    return strlen(s);
}
```

Generate the bindings:

```
aero build --cpp calc.aero
```

This produces `calc.dll` (Windows) and `calc.hpp`:

```cpp
// calc.hpp (auto-generated)
#include <cstdint>

extern "C" {
int64_t add(int64_t a, int64_t b);
double double_(double x) asm("double");
bool is_even(int64_t n);
int64_t str_len(const char* s);
} // extern "C"
```

Use it directly from C++:

```cpp
// main.cpp
#include <cstdio>
#include <cassert>
#include "calc.hpp"

int main() {
    assert(add(40, 2) == 42);
    assert(double_(3.5) == 7.0);
    assert(is_even(10) == true);
    assert(str_len("hello") == 5);
    std::printf("ALL PASS\n");
}
```

Compile and link (MinGW can link the DLL directly):

```
g++ main.cpp -I. calc.dll -o main.exe
```

## Type Mapping (v1)

| Aero | C++ | Description |
| --- | --- | --- |
| `i64` | `int64_t` | signed 64-bit integer |
| `i32` | `int32_t` | signed 32-bit integer |
| `f64` | `double` | double-precision float |
| `bool` | `bool` | boolean |
| `str` | `const char*` | NUL-terminated C string |
| no return value | `void` | empty return |

v1 does not support `String`, `Vec`, or raw pointers `*T` across the boundary. Using these types makes the compiler report an error and list the supported range.

## C++ Keyword Conflicts

If an Aero function name happens to be a C++ keyword (for example `double`), the generated C++ identifier gets an underscore suffix (`double_`) and the real symbol name is marked with `asm("symbol")`, so linking is unaffected:

```cpp
double double_(double x) asm("double");   // written as double_, linked symbol is double
```

## Cross-Compilation

`--cpp` reuses the `--shared` cross-compilation pipeline:

```
# Android
aero build --cpp --target aarch64-linux-android --ndk <NDK-path> calc.aero

# iOS (macOS + Xcode)
aero build --cpp --target aarch64-apple-ios calc.aero
```

## Differences from the Python Binding

| | `--pyext` | `--cpp` |
| --- | --- | --- |
| Consumer | Python `import` | C++ `#include` + link |
| Export marker | `#[py_export]` | `#[export]` |
| Output | `<name>.pyd` + glue layer | `<name>.dll` + `<name>.hpp` |
| Extra runtime | CPython | none (pure C ABI) |

Functions marked `#[py_export]` are also `#[export]`, so C++ can call the raw symbols they export too.

## Exercises

1. Write a `#[export] fn fib(n: i64) -> i64`, generate bindings with `aero build --cpp`, and call `fib(10)` from C++.
2. Write a `#[export]` function returning `f64` and confirm the C++ side receives a `double`.
3. Deliberately name a function `class` or `new` and see how the generated header handles it.
