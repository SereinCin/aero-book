# 01 Environment and First Program

## What You Need to Install

The Aero compiler itself is written in Rust, code generation goes through LLVM, and AOT linking uses MinGW's gcc. So you need three things:

| Component | Purpose | Version Requirement |
| --- | --- | --- |
| Rust toolchain (cargo/rustc) | Compiling the Aero compiler itself | 1.97+ |
| LLVM 22 | Generating machine code | 22.x (llvm-sys 221) |
| MinGW UCRT64 gcc | Linking object files into an .exe | Any recent version |

If you only want to use `aero run` (running programs via JIT), gcc is optional; but to produce a standalone executable with `aero build`, gcc is required.

## Installation Steps (Windows)

**1. Install Rust**

Go to https://rustup.rs to download rustup-init.exe, accept all the defaults. After installation, open a new terminal and verify:

```
cargo --version
rustc --version
```

**2. Install LLVM**

Aero uses LLVM 22. After installation, note its install directory, for example `D:\Scripts\LLVM\clang+llvm-22.1.8-x86_64-pc-windows-msvc`. Then set the environment variable:

```
set LLVM_SYS_221_PREFIX=D:\Scripts\LLVM\clang+llvm-22.1.8-x86_64-pc-windows-msvc
```

This environment variable is for Rust's llvm-sys library to find LLVM. The 221 in the name corresponds to LLVM 22.1—don't change it.

**3. Install gcc**

Install MSYS2, then in the MSYS2 terminal:

```
pacman -S mingw-w64-ucrt-x86_64-gcc
```

Add `C:\msys64\ucrt64\bin` to your PATH.

## Building the Aero Compiler

Once you have the Aero source directory, run the build script that comes with the project:

```
scripts\build.bat
```

The script compiles the entire workspace and finally produces the compiler binary at `target\debug\aero.exe`. The build takes a few minutes—let it finish.

Verify:

```
target\debug\aero.exe
```

With no arguments it prints the usage:

```
Usage:
  aero run <file.aero | package dir>   compile and run
  aero build [file.aero | dir]         compile to a standalone executable (AOT)
  aero new <name>                      create a new package skeleton
  aero test [file.aero]                run tests (default: all in tests/)
```

## Your First Program

Create a file `hello.aero`:

```aero
// First Aero program
print("Hello, Aero!\n");
```

Run it:

```
aero run hello.aero
```

Terminal output:

```
Hello, Aero!
```

Congratulations, your first Aero program is running.

## What This print Actually Does

`print` is Aero's built-in function; it ultimately calls the C standard library's `printf`. So:

- `print("Hello, Aero!\n")` outputs a line of text; `\n` is the newline character;
- without `\n` in the string, there's no newline;
- you can use format strings just like in C, e.g. `print("%d\n", 42)` (covered in detail in Chapter 3).

Try a few:

```aero
print(1 + 1);            // outputs 2
print(2 + 3 * 4);        // outputs 14 (multiplication/division take precedence)
print("two lines\nof text\n");   // outputs two lines
```

## Compiling to a Standalone Executable

`aero run` is the JIT approach: after compiling, it executes directly in memory without producing a file on disk. The upside is speed; the downside is you always need the compiler along for the ride.

To get a standalone, distributable .exe, use:

```
aero build hello.aero
```

This generates `hello.exe` next to `hello.aero`. Double-click it or run it in the terminal:

```
hello.exe
```

The output is the same: `Hello, Aero!`. This exe doesn't depend on the Aero compiler, nor on LLVM's dynamic libraries—it's a complete native program. Copy it to another Windows machine, and as long as the target machine has a normal UCRT runtime, it'll run.

## What Error Messages Look Like

Aero's compile errors are in English, with a consistent format:

```
error: line 3 col 5 [parse] expected `;`, found `}`
```

Left to right: the row and column of the error, the stage where it occurred (lexical/syntactic/type/borrow/code generation), and the specific message. When you see an error, first check the row and column, then the message, then jump to that location in the code.

## Exercises

1. Modify `hello.aero` so it outputs three lines: your name, today's date, and a slogan.
2. Use `aero build` to compile, rename the generated exe, copy it to another directory and run it, confirming it doesn't depend on the original directory.
3. Deliberately delete a semicolon and run it again. Read the error message and see whether the row/column pointer is accurate.
