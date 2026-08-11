# 06 Generics

## The Problem: Same Logic, Multiple Types

Suppose you want to write a function that "returns the larger of two parameters". Integers work, but comparing strings is also reasonable, and `i32` works too. Copy the code for each type?

```aero
fn max_i64(a: i64, b: i64) -> i64 {
    if (a > b) { return a; }
    return b;
}
```

And then you also need `max_i32`, `max_str`... annoying.

That's exactly what generics are for: **write once, the compiler generates multiple versions on demand**.

## Syntax: `<T>`

```aero
fn max<T>(a: T, b: T) -> T {
    if (a > b) { return a; }
    return b;
}

print(max(3, 7));    // 7
print(max(10, 5));   // 10
```

`<T>` declares a type parameter. All `T`s in the function signature mean "the same type": parameters `a` and `b` must be the same type, and the return value is also that type.

When calling, you **don't need to write the type parameter** — the compiler infers it from the arguments: in `max(3, 7)` both arguments are integers, so `T` is integer.

## Why `a > b` Is Legal

`max<T>` writes `a > b`. The generic parameter `T` is not yet determined, so how can it compare?

Aero's rule: **generic parameters pass loosely when undetermined within the signature, and are checked against the concrete type at instantiation**. `max(3, 7)` instantiates `T = i64`, and at that point `i64 > i64` is legal, so it compiles. If you call it with a type that doesn't support `>` (e.g. `bool`), an error is reported at instantiation.

## One Function, Multiple Type Parameters

```aero
fn make_pair<A, B>(a: A, b: B) -> (A, B) {
    return (a, b);
}

let p = make_pair(100, true);
print(p[0]);   // 100
print(p[1]);   // true
```

`A` and `B` are two independent type parameters and can be different.

## Generic Array Parameters

```aero
fn first<T>(arr: [T; 3]) -> T {
    return arr[0];
}

let nums = [1, 2, 3];
print(first(nums));   // 1
```

Note that the array length here is fixed as `[T; 3]` — Aero 0.1's generics do not support "arrays of arbitrary length"; the length is part of the type (detailed in Chapter 7).

## Generic Functions Calling Generic Functions

```aero
fn identity<T>(x: T) -> T {
    return x;
}

fn wrap<T>(x: T) -> T {
    return identity(x);   // Calling a generic inside a generic
}

print(wrap(42));   // 42
```

The compiler expands layer by layer: `wrap(i64)` internally calls `identity(i64)`, generating two copies of code.

## Monomorphization: Costs and Benefits

Aero's generics are **monomorphized**: every time a specific set of type parameters is encountered, an independent copy of machine code is generated.

- Benefit: no runtime overhead. `max(i64)` is just as fast as a hand-written `max_i64`, and type safety is guaranteed at compile time.
- Cost: if `max(3, 7)` and `max(1, 2)` have different types, two copies of code are generated, increasing the program size. Repeated calls with the same type generate only one copy (the compiler deduplicates).

## A Restriction

Generic parameter names cannot conflict with built-in type names:

```aero
// fn bad<i64>(x: i64) -> i64 { return x; }   // Error: generic parameter name conflicts with a built-in type name
```

Names like `i64`, `i32`, `bool`, `str` are taken by the type system and cannot be used as generic parameter names. Using single letters like `T`, `U`, `A`, `B` is the easiest approach.

## Exercises

1. Write an `identity<T>` and call it with an integer and a string respectively.
2. Write a generic function `second<T>(t: (T, T)) -> T` that returns the second element of the tuple, and verify it by calling.
3. Use `max<T>` to compare two strings (`"abc"` and `"abd"`) and see which is larger.
4. Think: why doesn't Aero provide `max` as a built-in generic function? (Hint: can every type be compared by size?)
