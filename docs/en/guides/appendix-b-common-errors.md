# 17 Appendix B: Common Errors and Solutions

This section collects the most common errors beginners run into. Each entry shows: what the error looks like, why it happens, and how to fix it.

## Missing Semicolon

```
line 1 col 9 [Syntax] expected `;`, found ...
```

**Cause**: The statement is missing a `;` at the end. **Fix**: Add a semicolon at the end of the previous line. All Aero statements (`let`, assignment, `print`, `return`) require semicolons.

## Type Mismatch (Implicit Narrowing)

```
[Type] type mismatch: variable `y` expected `i32`, got `i64`
```

**Cause**: Assigning an `i64` value to an `i32` variable. Aero rejects implicit narrowing. **Fix**: Either use `i64` for both, or use `i64` in the declaration. Integer literals can adapt to any integer type, but variables cannot:

```aero
let x = 10000000000;      // i64
// let y: i32 = x;        // Error
let y: i32 = 5;           // Literal works
```

## Undefined Variable / Undefined Function

```
[Name Resolution] undefined variable `x`
[Name Resolution] undefined function `foo`
```

**Cause**: The name is misspelled, or the variable is used outside its block (scope issue). **Fix**: Check the spelling; make sure the variable declaration is in the block where it's used (or an outer scope).

## Duplicate Variable Declaration

```
[Name Resolution] variable `x` is already declared in this scope
```

**Cause**: Using `let` twice for the same variable name in the same scope. **Fix**: Use a different name, or move the declaration to an outer scope. Note that Aero does not allow shadowing.

## Borrow Conflict

```
[Borrow Check] cannot mutably borrow: the variable is already borrowed
[Borrow Check] cannot immutably borrow: the variable is already mutably borrowed
[Borrow Check] variable cannot be assigned while borrowed
```

**Cause**: Violating the borrow rules from Chapter 9 — mutable borrows are not exclusive, or writing to the source variable while it is borrowed. **Fix**: Let the conflicting borrow end earlier (NLL: borrows automatically end after their last use), or reorder the code.

## if / while Condition is Not Boolean

```
[Type] `if condition` requires a boolean type, got `i64`
```

**Cause**: Writing `if (x)` — Aero does not allow integers as conditions. **Fix**: Write `if (x != 0)`.

## Nested Function Definitions

```
[Name Resolution] function definitions cannot be nested inside function bodies
```

**Cause**: Writing `fn` inside a function body. **Fix**: Place all functions at the top level.

## Tuple Index Issues

```
[Type] tuple index must be an integer constant within range
```

**Cause**: Using a variable as a tuple index, or an index beyond the tuple's length. **Fix**: Tuple indices must be hard-coded constants. For dynamic indexing, use arrays.

## Forgetting to Free Strings

This is not a compile error, but memory gradually grows at runtime. **Symptom**: The longer the program runs, the higher the memory usage gets.

**Cause**: The results of `int_to_str`, `substr`, or runtime concatenation were not freed with `str_free`. **Fix**: Free after use (see Chapter 8, "Who is Responsible for Freeing").

## Arena Overflow

**Symptom**: The program suddenly terminates (`abort`) at runtime.

**Cause**: The memory requested by `alloc` exceeds the capacity of `arena(N)`. **Fix**: Increase `N`, or review the allocation logic.

## Library Package Has Top-Level Statements

```
[Package Manager] library packages must not have top-level statements
```

**Cause**: The dependency library's `main.aero` contains executable statements like `print(...)`. **Fix**: Library packages should only contain function definitions.

## extern Function Type Not Allowed

```
[Type] extern "C" parameter `x` type `bool` is not C ABI compatible
```

**Cause**: `bool` (or arrays, tuples) cannot cross the C ABI. **Fix**: Use `i32`/`i64`/`str`/`*T` for parameters, and `i32`/`i64`/`*T`/void for return types.

## matmul Dimension Mismatch

```
[Type] matmul dimension mismatch: 2x3 and 2x3 cannot be multiplied
```

**Cause**: The number of columns in the left matrix does not equal the number of rows in the right matrix. **Fix**: Check the shapes of both matrices (`tensor(rows, columns)`).

## Generic Parameter Name Conflict

```
[Type] generic type parameter `i64` collides with a builtin type name
```

**Cause**: Using `i64`/`i32`/`bool`/`str` as a generic parameter name. **Fix**: Use names like `T`, `U`, etc.

## Native Array Out of Bounds

The compiler may not always catch this (constant-length literals are checked at compile time); out-of-bounds access at runtime is undefined behavior. **Fix**: Ensure the index is within `[0, len)`.

> **Vec Dynamic Array**: `get(i)` returns 0 if out of bounds, `set(i, v)` silently ignores the write — it won't crash or corrupt memory.

## Division by Zero

Aero checks for division by zero; dividing by zero returns 0 and does not crash. However, relying on "division by zero returns 0" for logic is a bad habit — it's recommended to ensure the divisor is not zero yourself.

## What if You Can't Remember

- Look at the first line of the error: line number, column number, and stage name.
- The stage name tells you where to look: `[Syntax]` for syntax, `[Type]` for types, `[Borrow Check]` for borrows, `[Code Generation]` is rare.
- The messages are in English, but the format is fixed. Flip to the corresponding entry in this chapter for reference.