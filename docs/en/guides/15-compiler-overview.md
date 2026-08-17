# 15 Compiler Overview

This chapter is not about how to implement a compiler, but about clarifying one thing: what your Aero code actually goes through before it becomes an executable. After reading this, you'll have a concrete sense of "why Aero is fast" and "why some errors are caught at compile time."

## Pipeline

```
Source Code → Lexical Analysis → Syntax Analysis → HIR (Name Resolution + Type Inference + Borrow Check) → LLVM IR → Machine Code
```

## Step 1: Lexical Analysis (Lex)

Splits the string into "tokens". `let x = 1;` is split into `let`, `x`, `=`, `1`, `;`. This step doesn't care about syntax correctness — it only handles splitting. Strings, numbers, identifiers, operators, and parentheses are all tokens.

## Step 2: Syntax Analysis (Parse)

Assembles tokens into a tree (AST) according to the grammar. `1 + 2 * 3` becomes a tree where "multiplication is below addition," corresponding to operator precedence. The errors caught at this stage are "syntax errors" — such as missing semicolons or mismatched parentheses.

## Step 3: HIR and Type Inference

This is Aero's core. The AST is transformed into HIR (High-Level Intermediate Representation), and three things happen simultaneously:

1. **Name Resolution**: Determines which variable `x` refers to, which function `max` is — everything is matched up. This stage catches "undefined variables" and "duplicate function definitions."
2. **Type Inference**: You write `let x = 1 + 2;` without specifying the type, and the compiler infers it at compile time — it's `i64`. This is what the whitepaper means by "0.1 milliseconds to deduce a 64-bit integer type." Type mismatches (e.g., assigning an i64 to an i32) are caught here.
3. **Borrow Check**: The borrow rules from Chapter 9 are verified here. Borrow conflicts prevent compilation — the program can't be generated at all.

The stage names in error messages (`[Lexical]`, `[Syntax]`, `[Type]`, `[Borrow Check]`, `[Code Generation]`) indicate which step of the pipeline caught you.

## Step 4: LLVM IR

HIR is translated into LLVM's intermediate representation (IR). IR is a format "close to machine code but still with types and variables." Aero only writes up to this point — the rest is handed to LLVM. This is the "don't reinvent the wheel" principle: machine code generation, register allocation, and instruction scheduling are things LLVM has been doing for decades at a quality far beyond what you'd write yourself.

## Step 5: Optimization and Machine Code

LLVM runs optimizations on the IR (Aero uses O2-level passes), such as removing unused variables, optimizing loops, and folding constants. Then it generates machine code for your CPU.

This step answers the question from the preface: Why are Aero programs faster than Python? Because types are fixed at step 3, allowing LLVM to generate efficient machine code; Python's type checking and object management all happen at runtime, and every line of code pays that cost.

## JIT and AOT: Two Ways to Finish

After optimization, the IR has two paths:

- **JIT** (`aero run`): LLVM compiles the machine code directly in memory and executes it immediately. The advantage is speed; the disadvantage is that the program can't run without the compiler.
- **AOT** (`aero build`): LLVM writes the machine code into an object file (.obj), then invokes gcc to link the CRT startup code, libraries, and your program's machine code into a standalone .exe.

## The Full Journey of a Program

```
hello.aero
  → Lexical: split into print ( "Hello" \n ) ;
  → Syntax: AST: call print, argument is a string
  → HIR: print is a built-in function, argument type is str
  → Type Inference: string literal is str, OK
  → Borrow Check: no borrows, OK
  → LLVM IR: generate instructions to call printf
  → Optimization: fold constants
  → AOT: write .obj, gcc links into hello.exe
```

From source code to a runnable exe — that's the pipeline. Now you have a concrete picture of what "compiler" means.