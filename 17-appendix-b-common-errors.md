# 17 Appendix B: Common Errors and Solutions

This collects the errors beginners run into most often. Each entry gives: what the error looks like, why it happens, and how to fix it.

## Missing Semicolon

```
line 1 col 9 [parse] expected `;`, found ...
```

**Cause**: The statement didn't end with `;`. **Fix**: Go to the end of the previous line and add a semicolon. Aero's statements (`let`, assignment, `print`, `return`) all need semicolons.

## Type Mismatch (Implicit Narrowing)

```
[type] type mismatch: variable `y` expected `i32`, got `i64`
```

**Cause**: You're stuffing an `i64` value into an `i32` variable. Aero rejects implicit narrowing. **Fix**: Either use `i64` on both sides, or declare with `i64`. Integer literals can adapt to any integer type, but variables can't:

```aero
let x = 10000000000;      // i64
// let y: i32 = x;        // error
let y: i32 = 5;           // literal is fine
```

## Undefined Variable / Undefined Function

```
[name_resolution] undefined variable `x`
[name_resolution] undefined function `foo`
```

**Cause**: Misspelled the name, or the variable is used outside its block (scope issue). **Fix**: Check the spelling; confirm the variable is declared in the block where it's used (or an outer one).

## Duplicate Variable Declaration

```
[name_resolution] variable `x` is already declared in this scope
```

**Cause**: You `let`-ed a variable with the same name twice in the same scope. **Fix**: Use a different name, or move the declaration to an outer scope. Note that Aero doesn't allow shadowing.

## Borrow Conflict

```
[borrow_check] cannot mutably borrow: the variable is already borrowed
[borrow_check] cannot immutably borrow: the variable is already mutably borrowed
[borrow_check] variable cannot be assigned while borrowed
```

**Cause**: You violated the borrow rules from Chapter 9 -- mutable borrows aren't exclusive, writing to the source variable while borrowed. **Fix**: Make the conflicting borrows end earlier (using NLL: borrows automatically end after their last use), or rearrange the code order.

## if / while Condition Not Boolean

```
[type] `if condition` requires a boolean type, got `i64`
```

**Cause**: You wrote `if (x)`, Aero doesn't allow integers as conditions. **Fix**: Write `if (x != 0)`.

## Nested Function Definitions

```
[name_resolution] function definitions cannot be nested inside function bodies
```

**Cause**: You wrote `fn` inside a function body. **Fix**: Put all functions at the top level.

## Tuple Index Issues

```
[type] tuple index must be an integer constant within range
```

**Cause**: The tuple index used a variable, or exceeded the tuple's length. **Fix**: Tuple indices must be fixed constants. For dynamic indexing, use an array.

## Forgot to Free a String

This isn't a compile error, it's runtime memory slowly growing. **Symptom**: The longer the program runs, the higher the memory usage.

**Cause**: The results of `int_to_str`, `substr`, or runtime concatenation weren't `str_free`-d. **Fix**: Free them after use (see Chapter 8 "Who's Responsible for Freeing").

## Arena Out of Bounds

**Symptom**: The program suddenly terminates at runtime (`abort`).

**Cause**: `alloc` requested more memory than the `arena(N)` capacity. **Fix**: Increase `N`, or check the allocation logic.

## Library Package Has Top-Level Statements

```
[package_manager] library packages must not have top-level statements
```

**Cause**: The dependency library's `main.aero` contains executable statements like `print(...)`. **Fix**: Library packages should only contain function definitions.

## extern Function Type Not Allowed

```
[type] extern "C" parameter `x` type `bool` is not C ABI compatible
```

**Cause**: `bool` (or arrays, tuples) can't cross the C ABI. **Fix**: Use `i32`/`i64`/`str`/`*T` for parameters, and `i32`/`i64`/`*T`/void for returns.

## matmul Dimension Mismatch

```
[type] matmul dimension mismatch: 2x3 and 2x3 cannot be multiplied
```

**Cause**: The left matrix's column count ≠ the right matrix's row count. **Fix**: Check the shapes of both matrices (`tensor(rows, cols)`).

## Generic Parameter Name Conflict

```
[type] generic type parameter `i64` collides with a builtin type name
```

**Cause**: You used `i64`/`i32`/`bool`/`str` as a generic parameter name. **Fix**: Use names like `T`, `U`.

## Array Out of Bounds

The compiler may not catch this (constant cases with literal lengths are checked); runtime out-of-bounds is undefined behavior. **Fix**: Make sure yourself that the index is within `[0, len)`.

## Can't Remember What to Do

- Look at the first line of the error: line/column number + stage name.
- The stage name tells you where to look: `[parse]` for syntax, `[type]` for types, `[borrow_check]` for borrows, `[codegen]` is rare.
- The messages are in English, but the format is fixed. Look up the corresponding entry in this chapter.
