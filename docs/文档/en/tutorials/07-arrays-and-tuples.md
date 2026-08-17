# 07 Arrays and Tuples

## Arrays: A Sequence of Values of the Same Type

```aero
let arr = [1, 2, 3];
print(arr[0]);   // 1
print(arr[2]);   // 3
```

Arrays use square brackets `[1, 2, 3]`, elements are accessed with `arr[0]`, and indexing starts at 0.

The type of an array is `[element type; length]`. The `arr` above has type `[i64; 3]`. **The length is part of the type** — `[i64; 3]` and `[i64; 4]` are two different types and cannot be assigned to each other.

## Array Elements Must Be of the Same Type

```aero
// let bad = [1, true];   // Error: array element types are inconsistent
```

## Modifying Elements

```aero
let arr = [1, 2, 3];
arr[0] = 100;
print(arr[0]);   // 100
```

## Direct Indexing of Array Literals

```aero
print([10, 20, 30][1]);   // 20
```

## Empty Arrays

`[]` cannot infer the element type, so a type annotation is required:

```aero
let empty: [i64; 0] = [];
print(len_arr(empty));  // see exercise below
```

## Array Pitfall: No Bounds Checking

Aero 1.1.0's **native arrays** (`[T; N]`) **do not perform bounds checking** on access (the compiler only checks constant-length literals at compile time; out-of-bounds access at runtime is undefined behavior). Don't write:

```aero
let arr = [1, 2, 3];
// print(arr[5]);   // Runtime out-of-bounds, unpredictable result — don't do this
```

You must ensure the index is within `0 <= i < length`. This is common in systems-level languages — fast, but the responsibility is yours.

> **Dynamic arrays (Vec) have different bounds behavior**: `Vec`'s `get(i)` and `set(i, v)` methods include bounds checking. On out-of-bounds access, `get` returns 0 and `set` is silently ignored — no crash. See Chapter 19 "Vec and Collection Types" for details.

## Tuples: Packing Different Types Together

```aero
let t = (10, true);
print(t[0]);   // 10
print(t[1]);   // true
```

Tuples use parentheses `(a, b)`, and elements can be of different types. The type is written as `(i64, bool)`.

Tuple indices **must be integer constants**, and **bounds are checked at compile time**:

```aero
let t = (10, true);
print(t[0]);   // 10
// print(t[2]);   // Compile error: tuple index out of bounds
// print(t[i]);   // Compile error: tuple index must be an integer constant
```

Tuples are suitable for "temporarily packing a few related values together," such as returning multiple results from a function:

```aero
fn divmod(a: i64, b: i64) -> (i64, i64) {
    return (a / b, a % b);
}

let r = divmod(17, 5);
print(r[0]);   // 3
print(r[1]);   // 2
```

## Arrays vs Tuples: How to Choose

- Same type of elements, logically variable-length (iterating with loops) → Array.
- A small number of values of different types packed together → Tuple.

Note that Aero 1.1.0 arrays **do not support changing length at runtime** (no push/pop), and you cannot use a variable for the array length (the `n` in `[i64; n]` must be a compile-time constant).

## Exercises

1. Write a function `sum3(arr: [i64; 3]) -> i64` that returns the sum of three elements, and verify it with `[5, 6, 7]`.
2. Write a function that returns `(quotient, remainder)`, test it with a negative number, and observe the result (think about whether `-17 / 5` is `-3` or `-4`).
3. Use a loop to sum the elements of `[1, 2, 3, 4, 5]` and print the total (hint: use a variable index to access `arr[i]` in the loop).