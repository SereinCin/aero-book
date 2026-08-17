# 03 Operators and Expressions

## Arithmetic Operations

```aero
print(1 + 2);    // 3
print(10 - 3);   // 7
print(7 * 6);    // 42
print(8 / 2);    // 4
print(-5 + 10);  // 5
```

Operators: `+` addition, `-` subtraction, `*` multiplication, `/` division, `-x` unary negation.

A few things to note:

- **Integer division truncates**. `7 / 2` evaluates to `3`, not `3.5`. Aero 1.1.0 only has integers, no floating-point types. This is a limitation of the current version.
- **Division by zero does not crash**. Aero checks at runtime whether the divisor is 0; if it is, it returns 0. However, relying on "division by zero returns 0" for logic is generally bad practice — it's better to ensure the divisor is not zero yourself.
- **Precedence is the same as C**: `* /` bind tighter than `+ -`, and parentheses `( )` have the highest precedence.

```aero
print(2 + 3 * 4);    // 14, not 20
print((2 + 3) * 4);  // 20
```

## Comparison Operations

```aero
print(3 < 5);     // true
print(3 > 5);     // false
print(3 <= 3);    // true
print(3 >= 4);    // false
print(3 == 3);    // true
print(3 != 4);    // true
```

Six comparison operators: `<` `>` `<=` `>=` `==` `!=`. All results are `bool`.

Both sides of a comparison must have the same type. `3 == 3.0` would be a compile-time error (no floating-point type, and no implicit conversion exists).

## Logical Operations: Short-Circuiting

```aero
let x = 5;
print(x > 0 && x < 10);   // true
print(x < 0 || x > 100);  // false
print(true && false);     // false
print(true || false);     // true
```

`&&` AND, `||` OR — operands must be `bool`.

**Short-circuiting** means: when `&&`'s left side is `false`, the right side is never evaluated; when `||`'s left side is `true`, the right side is never evaluated. This is important because it allows you to safely write:

```aero
let i = 10;
if (i != 0 && 100 / i > 5) { print("ok\n"); }
```

When `i != 0` is false, `100 / i` is never actually computed, so no division by zero occurs.

## print Format Strings

If the first argument to `print` is a string, it is treated as a C-style format string, and subsequent arguments fill in the format specifiers:

```aero
print("%d + %d = %d\n", 6, 7, 6 * 7);   // 6 + 7 = 42
print("value is %d\n", 100);            // value is 100
print("%s is %d years old\n", "Aero", 1);  // Aero is 1 years old
```

- `%d` fills in an integer
- `%s` fills in a string

You can also print a single number without a format string:

```aero
print(42);   // outputs 42 (no newline)
```

## The Difference Between Expressions and Statements

An expression is something that "has a value": `1 + 2`, `x > 0`, `"abc"` are all expressions. A statement is something that "does something": `print(x);`, `let y = 5;` are statements.

In Aero, `print(expression);` must always end with a semicolon. Missing a semicolon is the most common compilation error:

```
error: line 1 col 9 [syntax analysis] expected `;`, found ...
```

When you see this, check the previous line for a missing semicolon.

## Exercises

1. Use `print` with a format string to output `5 + 3 = 8`.
2. Write an expression that verifies `&&` short-circuiting: `false && (1 / 0 > 0)`, confirm it compiles, runs, and doesn't crash.
3. Calculate `(1 + 2) * (3 + 4) - 5`, compute the answer mentally first, then verify with the program.