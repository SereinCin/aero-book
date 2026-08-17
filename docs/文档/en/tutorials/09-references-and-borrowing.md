# 09 References and Borrowing

## Problem: Functions Can't Modify Outer Variables

Chapter 5 explained that function parameters are copied:

```aero
fn bump(n: i64) {
    n = n + 1;   // modifies the copy
}

let x = 10;
bump(x);
print(x);   // 10, unchanged
```

To make `bump` actually change `x` to 11, Aero uses **references**.

## References: Another Name for a Variable

```aero
fn bump(r: &mut i64) {
    *r = *r + 1;   // *r means "the variable that r points to"
}

let mut x = 10;   // Note: to be mutably borrowed, x must be declared as mut
bump(&mut x);
print(x);   // 11, actually changed
```

Breaking it down:

- `&mut x`: takes a mutable reference to x ("borrows the right to modify x").
- `&mut i64`: the parameter type is "mutable i64 reference".
- `*r`: dereference, reads or writes to the variable r points to.

References are Aero's only mechanism for "indirectly modifying an external variable." They are essentially addresses, but the compiler watches over them to prevent you from running into trouble.

## Read-Only References: Immutable Borrowing

```aero
fn peek(r: &i64) {
    print("%d\n", *r);
}

let x = 42;
peek(&x);   // 42
```

`&x` is an immutable borrow: you can read, but not write.

## Borrowing Rules (The Library Book Analogy)

Think of a variable as a book:

1. **You can lend out many "read-only copies" at the same time** — multiple `&x` references can coexist because everyone is just reading.
2. **"Mutable lending" is exclusive — only one person at a time** — while `&mut x` is borrowed, no other borrows are allowed.
3. **During a read-only borrow, you cannot lend out a mutable borrow** — if someone is reading, you can't have someone else rewriting at the same time.
4. **While a borrow is active, you cannot touch the original book** — while a reference is alive, the source variable itself cannot be reassigned (assignment counts as a write).

Violating any of these rules results in a compile-time error — the program won't even compile:

```aero
let x = 10;
let r1 = &x;
let r2 = &x;      // OK, multiple read-only borrows

// let r3 = &mut x;   // Error: read-only borrow already exists, cannot mutably borrow
// x = 20;            // Error: r1 is still alive, cannot write to x
print(*r1);
```

These rules eliminate some of the most insidious bugs — like "dangling pointers" and "data races" — at compile time.

## When Does a Borrow End: NLL

A borrow doesn't live "until the end of the variable's scope" — it lives until **its last use**. This rule is called NLL (Non-Lexical Lifetimes).

```aero
let x = 10;
let r = &x;
print(*r);   // r's last use is here
x = 20;      // OK! r is already done, now we can write to x
print(x);
```

Looks like it violates rule 4? No — after `print(*r)`, `r` has "returned the book," the borrow is over, and you can freely write to `x` afterward. **A borrow only lives until its last point of use** — this is the most practical feature of Aero's borrow checker.

## The Borrow Target Must Be a Variable

```aero
// let r = &(1 + 2);   // Error: borrow target must be a variable
```

A reference must point to a real, existing variable — it cannot point to a temporary expression (the temporary would be gone after the borrow).

## References to References? Not Yet

Aero 1.1.0's borrowing system is a streamlined version: it supports single-level `&T` / `&mut T`, but **does not** support references to references, storing references in arrays, returning references from functions, or other advanced patterns. Don't try to force it — the compiler will tell you what doesn't work. A full version of this system (closer to Rust's borrow checker) is on the roadmap; version 1.1.0 focuses on making the most common use cases work well.

## Exercises

1. Write `swap(a: &mut i64, b: &mut i64)` that swaps the values of two variables, and verify with `let x = 1; let y = 2; swap(&mut x, &mut y);`.
2. Write `increment_all` that adds 1 to every element of an array using references (hint: `let mut i = 0; while (i < 3) { arr[i] = arr[i] + 1; ... }` — you don't actually need references for this one; think about why).
3. Intentionally write code that "writes to the source variable during a borrow" and observe what the compile error looks like.
4. Verify NLL: can `let r = &x; print(*r); x = 100;` compile? What about `let r = &x; print(*r);` followed by `x = 100;`?