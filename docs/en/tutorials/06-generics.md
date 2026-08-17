# 06 Generics

## Problem: Same Logic, Multiple Types

Suppose you want to write a function that "returns the larger of two arguments." Integers work, but comparing strings also makes sense, and `i32` would work too. Copy the same logic for every type?

```aero
fn max_i64(a: i64, b: i64) -> i64 {
    if (a > b) { return a; }
    return b;
}
```

Then you'd also need `max_i32`, `max_str`... tedious.

Generics are for exactly this: **write once, the compiler generates multiple versions as needed**.

## Syntax: `<T>`

```aero
fn max<T>(a: T, b: T) -> T {
    if (a > b) { return a; }
    return b;
}

print(max(3, 7));    // 7
print(max(10, 5));   // 10
```

`<T>` declares a type parameter. Every `T` in the function signature means "the same type": parameters `a` and `b` must be the same type, and the return value is also that type.

When calling, **you don't need to write the type parameter** — the compiler infers it from the arguments: in `max(3, 7)`, both are integers, so `T` is integer.

## Why `a > b` Is Valid

`max<T>` uses `a > b`. The generic parameter `T` isn't determined yet, so how can it be compared?

Aero's rule: **Generic parameters are loosely accepted within the signature before resolution, and are checked against the concrete type at instantiation**. `max(3, 7)` instantiates `T = i64`, and `i64 > i64` is valid, so it compiles. If you call it with a type that doesn't support `>` (like `bool`), it will error at instantiation time.

## One Function, Multiple Type Parameters

```aero
fn make_pair<A, B>(a: A, b: B) -> (A, B) {
    return (a, b);
}

let p = make_pair(100, true);
print(p[0]);   // 100
print(p[1]);   // true
```

`A` and `B` are two independent type parameters; they can be different.

## Generic Array Parameters

```aero
fn first<T>(arr: [T; 3]) -> T {
    return arr[0];
}

let nums = [1, 2, 3];
print(first(nums));   // 1
```

Note that the array length is fixed as `[T; 3]` — Aero 1.1.0 generics do not support "arrays of arbitrary length"; the length is part of the type (more on this in Chapter 7).

## Generic Function Calling a Generic Function

```aero
fn identity<T>(x: T) -> T {
    return x;
}

fn wrap<T>(x: T) -> T {
    return identity(x);   // generic calling another generic
}

print(wrap(42));   // 42
```

The compiler expands layer by layer: `wrap(i64)` internally calls `identity(i64)`, generating two copies of code.

## Monomorphization: Cost and Benefit

Aero's generics are **monomorphized**: for each unique set of concrete type arguments, a separate copy of machine code is generated.

- Benefit: No runtime overhead — `max(i64)` is as fast as a hand-written `max_i64`, and type safety is guaranteed at compile time.
- Cost: If `max(3, 7)` and `max(1, 2)` use different types, two copies of code are generated, increasing program size. Repeated calls with the same type produce only one copy (the compiler deduplicates).

## A Limitation

Generic parameter names cannot conflict with built-in type names:

```aero
// fn bad<i64>(x: i64) -> i64 { return x; }   // Error: generic parameter name conflicts with built-in type name
```

Names like `i64`, `i32`, `bool`, `str` are reserved by the type system and cannot be used as generic parameter names. Single-letter names like `T`, `U`, `A`, `B` are the simplest to use.

## Exercises

1. Write an `identity<T>` and call it with an integer and a string respectively.
2. Write a generic function `second<T>(t: (T, T)) -> T` that returns the second element of a tuple, and verify it by calling it.
3. Use `max<T>` to compare two strings (`"abc"` and `"abd"`) — see which is larger.
4. Think: Why doesn't Aero provide `max` as a built-in generic function? (Hint: Can every type be compared?)