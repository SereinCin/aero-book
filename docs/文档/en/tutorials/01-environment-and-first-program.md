# 01 Environment and First Program

## What to Install

Aero's compiler itself is written in Rust, with code generation through LLVM, and AOT linking using MinGW's gcc. So you need three things:

| Component | Purpose | Version Requirement |
| --- | --- | --- |
| Rust toolchain (cargo/rustc) | Compile the Aero compiler itself | 1.97+ |
| LLVM 22 | Generate machine code | 22.x (llvm-sys 221) |
| MinGW UCRT64 gcc | Link object files into .exe | Any reasonably recent version |

If you only want to use `aero run` (JIT mode to run programs), gcc isn't required; but if you want to use `aero build` to produce a standalone executable, gcc is necessary.

## Installation Steps (Windows)

**1. Install Rust**

Go to https://rustup.rs to download rustup-init.exe, follow the defaults. After installation, open a new terminal and verify:

```
cargo --version
rustc --version
```

**2. Install LLVM**

Aero uses LLVM 22. After installation, note its installation directory, e.g. `D:\Scripts\LLVM\clang+llvm-22.1.8-x86_64-pc-windows-msvc`. Then set the environment variable:

```
set LLVM_SYS_221_PREFIX=D:\Scripts\LLVM\clang+llvm-22.1.8-x86_64-pc-windows-msvc
```

This environment variable is used by Rust's llvm-sys library to locate LLVM. The `221` in the name corresponds to LLVM 22.1 — don't change it.

**3. Install gcc**

Install MSYS2, then in the MSYS2 terminal:

```
pacman -S mingw-w64-ucrt-x86_64-gcc
```

Add `C:\msys64\ucrt64\bin` to your PATH.

## Build the Aero Compiler

Once you have the Aero source directory, run the project's build script:

```
scripts\build.bat
```

The script will compile the entire workspace and produce the compiler binary at `target\debug\aero.exe`. Building takes a few minutes, let it finish.

Verify:

```
target\debug\aero.exe
```

Without arguments, it prints usage:

```
Usage:
  aero run <file.aero | package dir>   compile and run
  aero build [file.aero | dir]         compile to a standalone executable (AOT)
  aero new <name>                      create a new package skeleton
  aero test [file.aero]                run tests (default: all in tests/)
```

For convenience, consider adding the `target\debug` directory to your system PATH environment variable so you can run `aero` commands from any directory:

```
setx PATH "%PATH%;project-directory\target\debug"
```

## First Program

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

Congratulations, your first Aero program is up and running.

## What Does This `print` Actually Do

`print` is a built-in function in Aero, which ultimately calls C standard library's `printf`. So:

- `print("Hello, Aero!\n")` outputs a line of text, `\n` is the newline character;
- Without `\n` in the string, no newline is printed;
- You can use format strings like in C, e.g. `print("%d\n", 42)` (detailed in Chapter 3).

Try writing a few:

```aero
print(1 + 1);            // outputs 2
print(2 + 3 * 4);        // outputs 14 (multiplication/division first)
print("two lines\nof text\n");   // outputs two lines
```

## Compile to a Standalone Executable

`aero run` uses JIT mode: it compiles and executes directly in memory, without writing a file to disk. The advantage is speed; the downside is needing the compiler every time.

To get a standalone distributable .exe, use:

```
aero build hello.aero
```

This generates `hello.exe` next to `hello.aero`. Double-click it or run it in a terminal:

```
hello.exe
```

The output is the same: `Hello, Aero!`. This exe does not depend on the Aero compiler or LLVM dynamic libraries — it's a complete native program. Copy it to another Windows machine, and as long as that machine has a standard UCRT runtime, it will run.

## What Error Messages Look Like

Aero's compilation errors are in English, with a uniform format:

```
error: line 3 col 5 [syntax analysis] expected `;`, found `}`
```

From left to right: the line and column of the error, the error stage (lexical/syntax/type/borrow/code generation), and the specific message. When you see an error, first check the line and column, then read the message, and finally go to the code at that location.

## Exercises

1. Modify `hello.aero` to output three lines: your name, today's date, and a slogan.
2. Compile with `aero build`, rename the generated exe, copy it to another directory, and confirm it doesn't depend on the original directory.
3. Deliberately delete a semicolon and run again, read the error message, and check whether the line/column numbers are accurate.