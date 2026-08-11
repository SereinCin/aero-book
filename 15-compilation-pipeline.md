# 15 Compilation Pipeline Overview

This chapter doesn't teach you how to implement a compiler. It only explains clearly: what exactly happens to the Aero code you write before it becomes an executable file. After reading this, you'll have a concrete sense of "why Aero is fast" and "why some errors are reported at compile time".

## The Pipeline

```
source code → lexical analysis → syntax analysis → HIR (name resolution + type inference + borrow checking) → LLVM IR → machine code
```

## Step 1: Lexical Analysis (Lex)

Chop the string into "words" (tokens). `let x = 1;` becomes `let`, `x`, `=`, `1`, `;`. This step doesn't care whether the syntax is correct, it only chops things up. Strings, numbers, identifiers, operators, and parentheses are all tokens.

## Step 2: Syntax Analysis (Parse)

Assemble tokens into a tree (AST) according to the grammar. `1 + 2 * 3` becomes a tree where "multiplication is below addition", corresponding to operator precedence. The errors this stage catches are "syntax errors" -- like a missing semicolon or mismatched parentheses.

## Step 3: HIR and Type Inference

This is Aero's core. The AST is transformed into HIR (High-level Intermediate Representation), while three things happen:

1. **Name resolution**: which variable does `x` refer to, which function is `max` -- everything is pinned down. This stage catches "undefined variable" and "duplicate function definition".
2. **Type inference**: you write `let x = 1 + 2;` without a type, and the compiler infers the type at compile time -- it's `i64`. This is what the whitepaper calls "deriving a 64-bit integer type in 0.1 milliseconds". Type mismatches (like stuffing an i64 into an i32) are caught here.
3. **Borrow checking**: the borrow rules from Chapter 9 are verified here. Borrow conflicts don't compile, the program can't even be produced.

The stage names in error messages (`[lex]`, `[parse]`, `[type]`, `[borrow_check]`, `[codegen]`) tell you which step of the pipeline you got stopped at.

## Step 4: LLVM IR

HIR is translated into LLVM's intermediate representation (IR). IR is a format that's "close to machine code but still carries types and variables". Aero itself only goes up to this step, then hands off to LLVM -- this is the "don't reinvent the low-level wheels" principle: machine code generation, register allocation, and instruction scheduling have been done by LLVM for decades, with quality far beyond what you'd write yourself.

## Step 5: Optimization and Machine Code

LLVM runs optimizations on the IR (Aero uses O2-level passes), such as removing unused variables, optimizing loops, and precomputing constants. Then it generates machine code for your CPU.

This step answers the question from the preface: why is Aero faster than Python? Because types are pinned down at step 3, LLVM can generate efficient machine code; Python's type checking and object management all happen at runtime, where every line of code pays a tax.

## JIT and AOT: Two Ways to Finish

The optimized IR has two paths:

- **JIT** (`aero run`): LLVM compiles the machine code directly in memory and executes it immediately. The upside is speed, the downside is the program can't run without the compiler.
- **AOT** (`aero build`): LLVM writes the machine code into an object file (.obj), then calls gcc to link the CRT startup code, libraries, and your program's machine code into a standalone .exe.

## A Program's Complete Journey

```
hello.aero
  → lex: chop into print ( "Hello" \n ) ;
  → parse: AST: call print, argument is a string
  → HIR: print is a builtin function, argument type str
  → type inference: string literal is str, OK
  → borrow check: no borrows, OK
  → LLVM IR: generate instruction to call printf
  → optimization: fold constants
  → AOT: emit .obj, gcc links into hello.exe
```

From source code to a runnable exe, it's just this one line. Now you have a mental picture of the word "compiler".
