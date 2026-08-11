# 00 Preface

## What Is Aero

Aero is a systems-level programming language. Let's break that down:

- **Systems-level**: It compiles to machine code, with no virtual machine and no garbage collector at runtime. Programs run fast, memory is controllable, and they can talk directly to the operating system and C libraries.
- **Programming language**: It has its own syntax, type system, and compiler. You write code in Aero, and the compiler turns the code into an executable.

Aero's goal: as effortless to write as Python, as fast to run as C++. This isn't a slogan—it's the result of deliberate design trade-offs. Aero pushes as much work as possible to compile time—type inference, memory checks, even some code expansion—leaving only what's truly necessary at runtime.

Three design bottom lines (Aero's "three no's"):

1. **No garbage collection**. GC stops your program midway through execution to sweep memory, and such pauses are unacceptable for systems-level programs. Aero manages memory in other ways (see Chapter 10 on Arena).
2. **Whatever can be done at compile time, never leave it to runtime**. Type inference and memory layout are all resolved during the compilation stage.
3. **Don't reinvent the wheel**. Aero doesn't write its own machine code generator; it hands its intermediate representation to LLVM, letting LLVM generate high-quality native code.

## Who This Book Is For

- You've written a little code in any language—Python, JavaScript, C, Java, all fine.
- You want to know "how to write programs in a language without garbage collection."
- You want to write a small tool that compiles into a standalone .exe.

You don't need to know C++, and you don't need to understand compiler theory. Chapter 15 briefly covers Aero's compilation pipeline, just to give you a mental model—it's not a barrier.

## How to Read This Book

Read it in order. The first eight chapters are the foundation (variables, control flow, functions, arrays, strings). Chapter 9 (borrowing) and Chapter 10 (Arena) are the soul of Aero and deserve extra time. Chapter 11 onward is on-demand reading—read Chapter 11 for numerical computing, Chapter 12 for calling C libraries, Chapter 13 for building command-line tools.

Every chapter follows the same structure: explain a concept first, then provide runnable code, then cover the "pitfalls," and finally give exercises. You're best off typing the code yourself—making a mistake once sticks better than reading it ten times.

## On "Fast"

Many people ask the first time they run Aero: why is the compiled program so much faster than Python?

The answer is no mystery. Python interprets each line of code at runtime, looks up types dynamically, and manages objects with reference counting. Aero nails down types at compile time, fixes memory layouts, and generates native machine code. Chapter 15 walks you through this whole pipeline. Remember the takeaway: **Aero moves complexity from runtime to compile time**, and the work done at compile time doesn't need to be paid for again at runtime.

## A Convention

All comments at the top of code blocks in this book (`// ...`) are just explanations and don't affect execution. You can delete the comments and the program's behavior won't change.

Now, set up the environment and run your first Aero program.
