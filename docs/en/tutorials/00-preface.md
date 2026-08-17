# 00 Preface

## What is Aero

Aero is a systems-level programming language. Let's break that down:

- **Systems-level**: It compiles to machine code, has no virtual machine, and no garbage collector at runtime. The programs you write run fast, memory is controllable, and you can interact directly with the operating system and C libraries.
- **Programming language**: It has its own syntax, type system, and compiler. You write code in Aero, and the compiler turns it into an executable file.

Aero's goal is: as convenient to write as Python, as fast to run as C++. This isn't just a slogan — it's the result of deliberate design trade-offs. Aero pushes as much work as possible to compile time — type inference, memory checks, and even partial code expansion — leaving only what is truly necessary at runtime.

Three design principles (Aero's "Three No's"):

1. **No garbage collection**. GC can pause your program mid-execution to clean up memory — such pauses are unacceptable for systems-level programs. Aero manages memory differently (see Chapter 10, Arena).
2. **Anything that can be done at compile time, must not be left to runtime**. Type inference, memory layout — all resolved at compile time.
3. **No reinventing the wheel**. Aero does not write its own machine code generator; it passes its intermediate representation to LLVM, which generates high-quality native code.

## Who This Book Is For

- You've written some code before, in any language — Python, JavaScript, C, Java, etc.
- You want to understand "how to write programs without a garbage collector."
- You want to write a small tool that compiles into a standalone .exe.

You don't need to know C++ or understand compiler theory. Chapter 15 will briefly introduce Aero's compilation pipeline — that's just to give you a mental model, not a prerequisite.

## How to Read This Book

Read in order. The first eight chapters are the foundation (variables, flow control, functions, arrays, strings). Chapter 9 (Borrowing) and Chapter 10 (Arena) are the soul of Aero — worth spending extra time on. From Chapter 11 onward, read on demand — Chapter 11 if you're doing numerical computing, Chapter 12 if you're calling C libraries, Chapter 13 if you're building command-line tools.

Each chapter follows the same structure: concepts first, then runnable code, then gotchas, and finally exercises. You're best off typing the code yourself — making a mistake once is more memorable than reading ten times.

## On Speed

Many people ask, the first time they run Aero: why is the compiled program so much faster than Python?

The answer is not mysterious. Python interprets every line at runtime, dynamically looks up types, and manages objects with reference counting. Aero locks down types at compile time, lays out memory in advance, and generates native machine code. In Chapter 15 you'll see the whole pipeline. Just remember this: **Aero shifts complexity from runtime to compile time** — and the work done at compile time doesn't need to be paid for again when the program runs.

## A Convention

Comments at the top of code blocks in this book (`// ...`) are explanatory only and do not affect execution. You can delete the comments and the program will behave the same.

Now, set up your environment and run your first Aero program.