# 07 Arrays and Tuples

## Arrays: A Sequence of Values of the Same Type

```aero
let arr = [1, 2, 3];
print(arr[0]);   // 1
print(arr[2]);   // 3
```

Arrays use square brackets `[1, 2, 3]`, elements are accessed via `arr[0]`, and indexing starts at 0.

The type of an array is `[element_type; length]`. The type of `arr` above is `[i64; 3]`. **Length is part of the type** — `[i64; 3]` and `[i64; 4]` are two different types and cannot be assigned to each other.

## Array Elements Must Be the Same Type

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

`[]` cannot infer the element type, so you must annotate:

```aero
let empty: [i64; 0] = [];
print(len_arr(empty));  // See exercise below
```

## A Pitfall of Arrays: No Bounds Checking

Aero 0.1's array access **does not perform bounds checking** (at compile time it only checks the constant case of literal lengths; runtime out-of-bounds access is undefined behavior). Don't write:

```aero
let arr = [1, 2, 3];
// print(arr[5]);   // Runtime out-of-bounds, unpredictable result, don't do this
```

You must ensure the index is within `0 <= i < length` yourself. This is a common style of systems-level languages — fast, but handing the responsibility to you.

## Tuples: Packing Different Types

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

Tuples are good for "temporarily packing a few related values", such as a function returning multiple results:

```aero
fn divmod(a: i64, b: i64) -> (i64, i64) {
    return (a / b, a % b);
}

let r = divmod(17, 5);
print(r[0]);   // 3
print(r[1]);   // 2
```

## Arrays vs Tuples, How to Choose

- Same-type elements, variable-count logic (looping through) → array.
- Packing a few values of different types → tuple.

Note that Aero 0.1's arrays **do not support changing length at runtime** (no push/pop), and you cannot use a variable as the array length (the `n` in `[i64; n]` must be a compile-time constant).

## Exercises

1. Write a function `sum3(arr: [i64; 3]) -> i64` that returns the sum of the three elements, verify with `[5, 6, 7]`.
2. Write a function that returns `(quotient, remainder)`, test it once with negative numbers and observe the result (think about whether `-17 / 5` is `-3` or `-4`).
3. Use a loop to accumulate the elements of `[1, 2, 3, 4, 5]` and print the total (hint: use a variable index to access `arr[i]` inside the loop).
