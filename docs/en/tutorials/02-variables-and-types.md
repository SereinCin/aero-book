# 02 Variables and Types

## Declaring a Variable

Aero uses `let` to declare variables:

```aero
let x = 1;
print(x);   // outputs 1
```

No need to write the type — the compiler infers it. The `x` above is inferred as a 64-bit integer.

Aero has four basic types:

| Type | Meaning | Literal Example |
| --- | --- | --- |
| `i64` | 64-bit signed integer | `42`, `-7` |
| `i32` | 32-bit signed integer | integer, determined by annotation |
| `bool` | Boolean | `true`, `false` |
| `str` | String | `"hello"` |

## Type Inference Rules

Aero's type inference has an important detail: **integer literals are "stretchable"**.

```aero
let x = 1;          // x is i64
let y: i32 = 1;     // valid! literal 1 can adapt to i32
let z: i32 = x;     // error! x is already i64, assigning to i32 is implicit narrowing
```

The rule is: **integer literals** (like `1`, `2 + 3`) can be annotated as either `i32` or `i64` — they follow whatever type annotates them. But if a variable already has a fixed type (e.g., `x` is `i64`), assigning it to an `i32` variable is rejected by the compiler — it won't silently truncate.

This is intentional. Implicit narrowing is one of the most common sources of bugs in C/C++: a large number gets silently truncated to a smaller type, causing completely wrong behavior with no compilation error. Aero catches this at compile time.

Remember: **once a type is determined, it will never be narrowed**.

## Explicit Annotation

Use a colon to specify the type explicitly:

```aero
let n: i64 = 10000000000;   // billions, must be i64
let small: i32 = 5;
let flag: bool = true;
let name: str = "Aero";
```

When is explicit annotation needed?

- When the literal exceeds the default inference range (integer literals default to i64, but being explicit is safer);
- For empty arrays, empty strings, and other cases where the element type can't be inferred (covered in Chapters 7 and 8);
- When code readability demands it.

## Variables Can Be Reassigned

After `let` declaration, variables are not constants — they can be changed:

```aero
let x = 1;
x = x + 1;
print(x);   // outputs 2
```

Use `=` to reassign. The type of the new value must match the original type; narrowing is not allowed:

```aero
let x: i32 = 1;
x = 2;          // OK
// x = 100000000000;   // compile error: value exceeds i32 range (implicit narrowing)
```

## Block Scope

`{ }` curly braces form a block. Variables declared inside a block are no longer valid after the block ends:

```aero
let a = 1;
if (true) {
    let b = 2;
    print(a + b);   // outputs 3, a is accessible from outside
}
// print(b);        // error: b does not exist here
```

Variables from the outer scope are accessible inside, but variables declared inside are not accessible outside. This is a common rule in all C-like languages, and Aero follows the same convention.

Additionally: **redeclaring a variable with the same name in the same scope is not allowed**.

```aero
let x = 1;
// let x = 2;   // error: variable `x` redeclared in the current scope
```

## A Detail: No `const` Keyword

Aero 1.1.0 does not have `const` — all variables can be reassigned. Making a value "read-only" relies on convention, not compiler enforcement. This will be improved in future versions.

## Exercises

1. Declare an `i32` variable, an `i64` variable, and a `bool` variable, assign each a value, and `print` them individually.
2. Write code that deliberately assigns an `i64` variable to an `i32` variable, and observe the compilation error message.
3. Declare a variable inside an `if` block, try to `print` it outside the block, observe the error, then fix it.