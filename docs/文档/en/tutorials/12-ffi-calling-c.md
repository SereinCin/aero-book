# 12 FFI: Calling C

## Aero Is Not an Island

The real world is full of existing C libraries: system APIs, graphics libraries, math libraries... Aero provides FFI (Foreign Function Interface), allowing you to call C functions directly without writing glue code.

## Declaring a C Function

```aero
extern "C" fn strlen(s: str) -> i64;
extern "C" fn abs(x: i32) -> i32;

print(strlen("hello"));   // 5
print(abs(-42));          // 42
```

Key points:

- `extern "C"` means "this is an external function using the C ABI";
- **No function body**, ends with a semicolon — it's just a declaration; the actual function lives in some C library;
- Call it just like a normal function.

`strlen` is a C standard library function (returns the length of a string). `abs` returns the absolute value.

## Symbol Aliasing: Aero Name ≠ C Symbol Name

By default, the function name in Aero is the C symbol name used during linking. But sometimes the C function name differs from the Aero name you want to use — use `= "symbol"` to specify:

```aero
extern "C" fn string_len(s: str) -> i64 = "strlen";
extern "C" fn abs_c(x: i32) -> i32 = "abs";

print(string_len("hi"));   // 2
```

Here, the Aero name is `string_len`, but the C symbol it looks for during linking is `strlen`. If you don't write `= "..."`, the function name itself is used.

## Type Restrictions

The C ABI is quite old, and not all Aero types can pass through it. Allowed in 1.1.0:

| Position | Allowed Types |
| --- | --- |
| Parameters | `i32`, `i64`, `str` (i.e., `char*`), `*T` pointer |
| Return | `i32`, `i64`, `*T` pointer, no return value |

**`bool` is not allowed**, `array/tuple` is not allowed:

```aero
// extern "C" fn bad(x: bool) -> i64;   // Compile error: bool is not a C ABI-compatible type
```

## Strings and C's `char*` Are the Same Thing

Aero's `str` is essentially C's `char*` (`i8*`) underneath. So:

```aero
extern "C" fn putchar(c: i32) -> i32;

putchar(65);   // outputs character 'A' (ASCII 65)
```

When `str` is passed to a C function, the C function receives a NUL-terminated string pointer. Conversely, when a C function returns `char*`, it becomes `str` in Aero.

## Linking System Libraries: The [link] Section

Once you've declared the functions, you need to tell the linker to connect the libraries. The C standard library (strlen, abs) is available by default; **other libraries must be declared in `Aero.toml`**.

For example, calling the Windows API `GetTickCount` (returns milliseconds since system startup), which lives in `kernel32.dll`:

```toml
[package]
name = "winpkg"
version = "1.1.0"

[link]
libs = ["kernel32"]   # link -lkernel32
```

```aero
extern "C" fn GetTickCount() -> i32;

let t = GetTickCount();
print("%d\n", t);   // e.g. 12560265
```

`[link].libs` is a list of library names (`-l<name>`), and `[link].lib_paths` is additional library search paths (`-L<path>`).

## Linking Your Own C Libraries

You can compile a C static library with gcc first, then call it from Aero. Suppose `mylib.c`:

```c
int aero_add(int a, int b) { return a + b; }
int aero_mul3(int a) { return a * 3; }
```

Compile it into a library:

```
gcc -c mylib.c -o mylib.o
ar rcs libmylib.a mylib.o
```

Then `Aero.toml`:

```toml
[link]
libs = ["mylib"]
lib_paths = ["."]
```

Aero code:

```aero
extern "C" fn aero_add(a: i32, b: i32) -> i32;
extern "C" fn aero_mul3(a: i32) -> i32;

print(aero_add(2, 3));    // 5
print(aero_mul3(4));      // 12
```

Note that the library file is named `libmylib.a`, and you write `mylib` in `libs` (dropping the `lib` prefix and `.a` suffix) — this is the linker convention.

## Difference Between JIT and AOT

- `aero run` (JIT): Function symbols are looked up from the system's export table at runtime. Most C standard library functions work directly, but some names may differ (e.g., certain CRT functions are called `_snprintf` in the Windows export table).
- `aero build` (AOT): Linking is handled by gcc, symbol resolution is exactly the same as a normal C program, and the `[link]` section takes effect here.

**Conclusion: For programs that need to call external libraries, using `aero build` to produce an exe is the most reliable approach.**

## Exercises

1. Declare and call `toupper(c: i32) -> i32` (C standard library, converts lowercase to uppercase), converting `'a'` to the ASCII code of `'A'` and printing it.
2. Declare `strcmp(a: str, b: str) -> i32`, compare two strings, print the result, then compare with Aero's `str_cmp`.
3. Write a C file providing `int square(int)`, compile it into a static library, use the `[link]` section to call it from Aero, and verify with `aero build`.
4. Try declaring an extern function with a `bool` parameter and see the compile error message.