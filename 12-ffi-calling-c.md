# 12 FFI: Calling C

## Aero Is Not an Island

The real world has a pile of existing C libraries: system APIs, graphics libraries, math libraries... Aero provides FFI (Foreign Function Interface) so you can directly call C functions without writing glue code.

## Declaring a C Function

```aero
extern "C" fn strlen(s: str) -> i64;
extern "C" fn abs(x: i32) -> i32;

print(strlen("hello"));   // 5
print(abs(-42));          // 42
```

Key points:

- `extern "C"` means "this is an external function with the C ABI";
- **No function body**, ends with a semicolon—it's just a declaration; the function body lives in some C library;
- The call syntax is the same as for ordinary functions.

`strlen` is a C standard library function (returns the string length). `abs` takes the absolute value.

## Symbol Aliases: Aero Name ≠ C Symbol Name

By default, the function name in Aero is the C symbol name looked up at link time. But some C function names differ from the Aero name you want to use; specify it with `= "symbol"`:

```aero
extern "C" fn string_len(s: str) -> i64 = "strlen";
extern "C" fn abs_c(x: i32) -> i32 = "abs";

print(string_len("hi"));   // 2
```

Here it's called `string_len` in Aero, but at link time the C symbol looked up is `strlen`. Without `= "..."`, the function name itself is used.

## Type Restrictions

C's ABI is quite old, and not every Aero type can pass through. What 0.1 allows:

| Position | Allowed types |
| --- | --- |
| Parameters | `i32`, `i64`, `str` (i.e., `char*`), `*T` pointers |
| Return | `i32`, `i64`, `*T` pointers, no return value |

**`bool` won't work**, `arrays/tuples` won't work:

```aero
// extern "C" fn bad(x: bool) -> i64;   // compile error: bool is not a C ABI-compatible type
```

## Strings and C's `char*` Are the Same Thing

Aero's `str` is, at the low level, C's `char*` (`i8*`). So:

```aero
extern "C" fn putchar(c: i32) -> i32;

putchar(65);   // outputs the character 'A' (ASCII 65)
```

When a `str` is passed into a C function, what the C function gets is a NUL-terminated string pointer. Conversely, a `char*` returned by a C function is a `str` in Aero.

## Linking System Libraries: The [link] Section

Having declared the functions, you also need the linker to wire up the library. The C standard library (strlen, abs) is available by default; **other libraries must be declared in `Aero.toml`**.

For example, to call the Windows API `GetTickCount` (returns the number of milliseconds since system boot), which lives in `kernel32.dll`:

```toml
[package]
name = "winpkg"
version = "0.1.0"

[link]
libs = ["kernel32"]   # links -lkernel32
```

```aero
extern "C" fn GetTickCount() -> i32;

let t = GetTickCount();
print("%d\n", t);   // e.g., 12560265
```

`[link].libs` is a list of library names (`-l<name>`), and `[link].lib_paths` is additional library search paths (`-L<path>`).

## Linking Your Own C Library

You can first compile a C static library with gcc, then call it from Aero. Suppose `mylib.c`:

```c
int aero_add(int a, int b) { return a + b; }
int aero_mul3(int a) { return a * 3; }
```

Compile into a library:

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

Note the library file name is `libmylib.a`, but in `libs` you write `mylib` (dropping the `lib` prefix and `.a` suffix)—this is a linker convention.

## The Difference Between JIT and AOT

- `aero run` (JIT): function symbols are looked up at runtime from the system's export table. Most of the C standard library works directly, but individual names may differ (e.g., some CRT functions are called `_snprintf` in the Windows export table).
- `aero build` (AOT): handed off to gcc for linking; symbol resolution is exactly like an ordinary C program, and the `[link]` section only truly takes effect here.

**Conclusion: for programs that need to call external libraries, building an exe with `aero build` is the most reliable.**

## Exercises

1. Declare and call `toupper(c: i32) -> i32` (C standard library, converts lowercase to uppercase), and convert `'a'` to `'A'` and print its ASCII code.
2. Declare `strcmp(a: str, b: str) -> i32`, compare two strings, print the result, and compare it with Aero's `str_cmp`.
3. Write a C file providing `int square(int)`, compile it into a static library, call it from Aero using the `[link]` section, and verify by building an exe with `aero build`.
4. Try declaring an extern function with a `bool` parameter, and look at the compiler error.
