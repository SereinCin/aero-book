# 02 Variables and Types

## Declaring a Variable

Aero uses `let` to declare variables:

```aero
let x = 1;
print(x);   // outputs 1
```

No need to write the type—the compiler infers it. In the line above, `x` is inferred as a 64-bit integer.

Aero has four basic types:

| Type | Meaning | Literal Examples |
| --- | --- | --- |
| `i64` | 64-bit signed integer | `42`, `-7` |
| `i32` | 32-bit signed integer | integer, determined by annotation |
| `bool` | Boolean | `true`, `false` |
| `str` | String | `"hello"` |

## Rules of Type Inference

There's an important detail in Aero's type inference: **integer literals have a "flexible" type**.

```aero
let x = 1;          // x is i64
let y: i32 = 1;     // legal! the literal 1 can fit i32
let z: i32 = x;     // error! x is already i64, fitting it into i32 is implicit narrowing
```

The rule is: **integer literals** (like `1`, `2 + 3`) can be annotated as either `i32` or `i64`—whichever annotation is given to them, they follow it. But if a variable already has a fixed type (e.g., `x` is `i64`), and you then assign it to an `i32` variable, the compiler refuses—it won't silently truncate for you.

This rule is deliberate. Implicit narrowing is one of the most common sources of bugs in C/C++: a large number gets silently truncated to a smaller type, the program behaves completely wrong yet compiles without error. Aero chooses to catch this at compile time.

Remember one sentence: **once a type is determined, it will never narrow again**.

## Explicit Annotations

When you want to spell out the type, use a colon:

```aero
let n: i64 = 10000000000;   // billions, must be i64
let small: i32 = 5;
let flag: bool = true;
let name: str = "Aero";
```

When do you need explicit annotations?

- When a literal exceeds the default inference range (though integer literals default to i64, spelling it out is safer);
- For empty arrays, empty strings, and other scenarios where the element type can't be inferred (covered in Chapters 7 and 8);
- When code readability calls for it.

## Variables Can Be Reassigned

After `let` declaration, the variable isn't a constant—you can change it:

```aero
let x = 1;
x = x + 1;
print(x);   // outputs 2
```

Use `=` to reassign. The type of the new value must match the original type; narrowing isn't allowed:

```aero
let x: i32 = 1;
x = 2;          // OK
// x = 100000000000;   // compile error: value exceeds i32 range (implicit narrowing)
```

## Block Scope

`{ }` curly braces form a block. Variables declared inside a block become invalid once the block ends:

```aero
let a = 1;
if (true) {
    let b = 2;
    print(a + b);   // outputs 3, a from the outside is also usable
}
// print(b);        // error: b doesn't exist here
```

Variables from outside are usable inside; variables from inside aren't usable outside. This is a common rule across all C-like languages, and Aero is no different.

Additionally: **declaring a variable with the same name in the same scope is not allowed**.

```aero
let x = 1;
// let x = 2;   // error: variable `x` redeclared in the current scope
```

## A Detail: No `const` Keyword

Aero 0.1 has no `const`—all variables can be reassigned. If you want a value to be "read-only," you rely on convention, not compiler enforcement. This will be improved in future versions.

## Exercises

1. Declare an `i32` variable, an `i64` variable, and a `bool` variable, assign a value to each, and `print` them separately.
2. Write a piece of code that deliberately assigns an `i64` variable to an `i32` variable, and observe the compile error message.
3. Declare a variable inside an `if` block, then use `print` to print it outside the block, observe the error, and then fix it.
