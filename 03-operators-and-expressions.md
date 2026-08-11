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

A few things to watch out for:

- **Integer division truncates directly**. `7 / 2` gives `3`, not `3.5`. Aero 0.1 only has integers—there's no floating-point type. This is a limitation of the current version.
- **Division by zero is undefined behavior**. Aero doesn't check it for you—don't write `x / 0`.
- **Precedence matches C**: `* /` is higher than `+ -`, and parentheses `( )` are the highest.

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

Six comparison operators: `<` `>` `<=` `>=` `==` `!=`. The result is always `bool`.

Both sides of a comparison must have the same type. `3 == 3.0` is a direct compile error (there's no floating point, and no implicit conversion exists either).

## Logical Operations: Short-Circuiting

```aero
let x = 5;
print(x > 0 && x < 10);   // true
print(x < 0 || x > 100);  // false
print(true && false);     // false
print(true || false);     // true
```

`&&` is AND, `||` is OR; the operands must be `bool`.

**Short-circuiting** means: when the left side of `&&` is `false`, the right side isn't evaluated at all; when the left side of `||` is `true`, the right side isn't evaluated. This matters because it means you can safely write:

```aero
let i = 10;
if (i != 0 && 100 / i > 5) { print("ok\n"); }
```

When `i != 0` is false, `100 / i` isn't actually computed—no division by zero.

## print's Format Strings

If the first argument to print is a string, it's treated as a C-style format string, and the following arguments fill in the format specifiers:

```aero
print("%d + %d = %d\n", 6, 7, 6 * 7);   // 6 + 7 = 42
print("value is %d\n", 100);            // value is 100
print("%s is %d years old\n", "Aero", 1);  // Aero is 1 years old
```

- `%d` fills in an integer
- `%s` fills in a string

You can also skip the format string and just print a single number:

```aero
print(42);   // outputs 42 (no newline)
```

## The Difference Between Expressions and Statements

An expression is "something that has a value": `1 + 2`, `x > 0`, `"abc"` are all expressions. A statement "does something": `print(x);`, `let y = 5;` are statements.

In Aero, `print(expression);` must always be followed by a semicolon. A missing semicolon is the most common compile error:

```
error: line 1 col 9 [parse] expected `;`, found ...
```

When you see this, first look for the missing semicolon on the previous line.

## Exercises

1. Use print and a format string to output `5 + 3 = 8`.
2. Write an expression that verifies the short-circuiting of `&&`: `false && (1 / 0 > 0)`, and confirm it can compile, run, and not crash.
3. Compute `(1 + 2) * (3 + 4) - 5`—first work out the answer in your head, then let the program verify it.
