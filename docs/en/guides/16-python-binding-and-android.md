# 16 Python Binding and Android/iOS Cross-Compilation

Aero can compile not only to standalone executables but also to **shared libraries** callable from other languages and platforms:

- **Python binding**: export Aero functions as Python C extensions (`.pyd` / `.so`) that Python can `import` directly;
- **Android / iOS shared libraries**: cross-compile `lib*.so` / `lib*.dylib` for mobile apps to call via JNI / FFI.

Both share the same foundation: turning Aero functions into externally visible C ABI symbols (`#[export]`) and building a shared library (`aero build --shared`). This chapter starts from that foundation.

## 1. Exporting Functions: `#[export]`

The symbols of ordinary Aero functions are internal. To make a function callable from a shared library, add `#[export]`:

```aero
#[export]
fn add(a: i64, b: i64) -> i64 {
    return a + b;
}

#[export]
fn double(x: f64) -> f64 {
    return x * 2.0;
}
```

Constraints on `#[export]`:

- Cannot be a generic function (must be monomorphized);
- Cannot be combined with `extern "C"` (what is exported is the actual function definition);
- Parameters must be C-ABI compatible types: `i32` / `i64` / `f64` / `bool` / `str` / `String` / `*T`;
- Return types follow the same rule, plus no return value.

## 2. Shared Libraries: `aero build --shared`

Build a file with `#[export]` functions into a dynamic library:

```
aero build --shared examples/export_demo.aero
```

This produces `.dll` (Windows) / `.so` (Linux/Android) / `.dylib` (macOS) for the target platform. A top-level `main` statement is preserved but hidden — it is not the library's entry point.

## 3. Python Binding: `aero build --pyext`

`#[py_export]` builds on `#[export]` and additionally makes the compiler **automatically generate the CPython glue layer** — wrapper functions, method tables, module definitions, and the `PyInit_<module>` entry point are all generated for you. You don't write a single line of C.

```aero
// py_bind.aero
#[py_export]
fn add(a: i64, b: i64) -> i64 {
    return a + b;
}

#[py_export]
fn double(x: f64) -> f64 {
    return x * 2.0;
}

#[py_export]
fn greet(name: str) -> str {
    return name;
}
```

Compile:

```
aero build --pyext examples/py_bind/py_bind.aero
```

This generates `py_bind.pyd` (Windows; `.so` on other platforms). Import it directly from Python:

```python
import py_bind
py_bind.add(40, 2)      # 42
py_bind.double(3.5)     # 7.0
py_bind.greet("aero")   # 'aero'
```

### Module Name

By default the prefix of the source file name is used as the module name (`py_bind.aero` → module `py_bind`). To rename it:

```
aero build --pyext --py-module mymod examples/py_bind/py_bind.aero
```

The module name must match the generated file name (`mymod.pyd` ↔ `import mymod`).

### Locating Python

Compiling links against Python's import library. The compiler searches in order:

1. `--py-home <prefix>` (Python installation prefix);
2. The `PYTHON_HOME` environment variable;
3. Probing the `python` executable.

On Windows the official Python ships an MSVC import library, so the compiler automatically uses `objdump` + `dlltool` to produce a MinGW-compatible `libpython3xx.a`.

### Type Conversion Table

| Aero type | Python type | Argument parsing | Return construction |
| --- | --- | --- | --- |
| `i64` | `int` | `l` | `PyLong_FromLongLong` |
| `f64` | `float` | `d` | `PyFloat_FromDouble` |
| `bool` | `bool` | `p` | `PyBool_FromLong` |
| `str` | `str` (UTF-8) | `s` | `PyUnicode_FromString` |
| `String` | `bytes` | `y#` (copied into a buffer) | `PyBytes_FromStringAndSize` |
| no return value | `None` | — | `Py_None` |

`String` maps to Python's `bytes` — Aero's `String` is a `{ data, len, cap }` raw byte buffer (safe with embedded `\0`), naturally suited to binary data. Example:

```aero
#[py_export]
fn bytes_reverse(b: String) -> String {
    let out = String::new();
    let n = b.len();
    let mut i = n - 1;
    while (i >= 0) {
        let c = b.at(i);
        out.push(c);
        i = i - 1;
    }
    return out;
}
```

```python
import py_bind2
py_bind2.bytes_reverse(b"abc")           # b'cba'
py_bind2.bytes_reverse(b"\x00\x01\x02")  # b'\x02\x01\x00'
```

### Smoke Test

`examples/py_bind/` and `examples/py_bind2/` each ship a `test_py.py`; after compiling:

```
python test_py.py   # prints "py_bind smoke: ALL PASS"
```

## 4. Android Shared Libraries: `aero build --shared --target aarch64-linux-android`

Compile Aero core logic into an Android `lib*.so` that a host app calls via JNI / FFI.

```
aero build --shared --target aarch64-linux-android --ndk <NDK-path> examples/export_demo.aero
```

This produces `libexport_demo.so` (Android convention: shared libraries must be named `lib<name>.so`).

### Locating the NDK

The compiler searches for the NDK in order:

1. `--ndk <path>`;
2. The `ANDROID_NDK_HOME` environment variable;
3. Common install locations (`%LOCALAPPDATA%\Android\Sdk\ndk`, `~/Android/Sdk/ndk`, etc.).

The NDK root must contain `toolchains/llvm/prebuilt/<host-platform>/`; the compiler links with its `clang` + `sysroot`. If multiple versions are installed under `ndk/`, the newest is picked automatically.

### Supported Target Triples

| Triple | ABI | NDK `--target` |
| --- | --- | --- |
| `aarch64-linux-android` | arm64-v8a | `aarch64-linux-android21` |
| `armv7-linux-androideabi` | armeabi-v7a | `armv7a-linux-androideabi21` |
| `x86_64-linux-android` | x86_64 | `x86_64-linux-android21` |
| `i686-linux-android` | x86 | `i686-linux-android21` |

The API level is fixed at 21 (Android 5.0+); the compiler only calls into bionic libc, so there is no version gate.

### Verifying Exported Symbols

With an NDK you can build the `.so` locally; without one, use CI (a GitHub Actions Ubuntu runner with the NDK installed) as a smoke test. Inspect the exported symbols with `llvm-nm` / `readelf`:

```
llvm-nm -D --defined-only libexport_demo.so   # should show T add / T double
readelf -h libexport_demo.so                  # Machine: AArch64
```

## 5. iOS Shared Libraries: `aero build --shared --target aarch64-apple-ios`

Compile Aero core logic into an iOS `lib*.dylib` that a host app calls via FFI / Swift-ObjC bridging.

```
aero build --shared --target aarch64-apple-ios examples/export_demo.aero
```

This produces `libexport_demo.dylib`.

### Hard Prerequisite

The iOS toolchain (Xcode + `xcrun`) **only runs on macOS**, so a Windows dev machine cannot verify it locally. The command works in two scenarios:

- **macOS dev machine** (with Xcode installed): run it directly;
- **CI**: a GitHub Actions `macos-latest` runner ships Xcode; the repo provides a smoke pipeline in [`ios.yml`](https://github.com/SereinCin/aero-lang/blob/main/.github/workflows/ios.yml).

Running on a non-macOS machine reports "cannot locate Xcode toolchain" and exits non-zero.

### SDK and Architecture

The compiler locates the system libraries with `xcrun --sdk <sdk> --show-sdk-path` and the compiler with `xcrun --sdk <sdk> -f clang`. The SDK is chosen automatically from the triple:

| Triple | Platform | SDK | `-arch` |
| --- | --- | --- | --- |
| `aarch64-apple-ios` | device (arm64) | `iphoneos` | `arm64` |
| `aarch64-apple-ios-sim` | Apple Silicon simulator | `iphonesimulator` | `arm64` |
| `x86_64-apple-ios` | Intel simulator | `iphonesimulator` | `x86_64` |

The deployment version is fixed at iOS 13 (safe for both device and simulator).

### Mach-O Symbol Names

Apple's Mach-O format prefixes all global symbols with `_`. The `_snprintf` the compiler generates internally for the Windows CRT is automatically renamed to `snprintf` during cross-compilation, so after linking the symbol is exactly `_snprintf` exported by libSystem — no hand-written symbol aliases needed.

### Verifying Exported Symbols

```
nm -gU libexport_demo.dylib    # should show T _add / T _double
lipo -info libexport_demo.dylib  # Architecture: arm64
```

## Summary

- `#[export]` turns Aero functions into C ABI symbols; `#[py_export]` additionally auto-generates the Python glue;
- `aero build --shared` builds shared libraries, `--pyext` builds Python extensions, and `--target <android-triple>` / `--target <ios-triple>` cross-compiles;
- The Python binding currently locks the full CPython ABI (recompile after upgrading Python);
- Android requires the NDK (`--ndk` or `ANDROID_NDK_HOME`);
- iOS requires macOS + Xcode; on Windows it can only be verified via CI (a macOS runner).

## Exercises

1. Write a `#[py_export] fn fib(n: i64) -> i64`, build it into a `.pyd`, then `import` it and call `fib(10)`.
2. Write a function that takes a `String` (bytes) and returns an `i64` checksum; verify it with `b"hello"`.
3. With an NDK, compile `export_demo.aero` into an `aarch64-linux-android` `.so` and confirm the `add` symbol is visible with `readelf`.
4. With macOS + Xcode, compile `export_demo.aero` into an `aarch64-apple-ios` `.dylib` and confirm `_add` is visible with `nm -gU`.
