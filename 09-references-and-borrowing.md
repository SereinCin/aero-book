# 09 References and Borrowing

## The Problem: Functions Can't Change External Variables

Chapter 5 explained that function parameters are copied:

```aero
fn bump(n: i64) {
    n = n + 1;   // modifying the copy
}

let x = 10;
bump(x);
print(x);   // 10, unchanged
```

To make `bump` actually change `x` to 11, Aero's solution is **references**.

## References: Another Name for a Variable

```aero
fn bump(r: &mut i64) {
    *r = *r + 1;   // *r means "the variable r points to"
}

let mut x = 10;   // note: to be mutably borrowed, x must be declared mut
bump(&mut x);
print(x);   // 11, actually changed
```

Breaking it down:

- `&mut x`: takes a mutable reference to x ("borrows x's modification right").
- `&mut i64`: the parameter type is "mutable i64 reference".
- `*r`: dereference, reads/writes the variable r points to.

References are the only mechanism in Aero that can "indirectly modify external variables." It's essentially an address, but the compiler watches it to prevent you from borrowing incorrectly.

## Read References: Immutable Borrows

```aero
fn peek(r: &i64) {
    print("%d\n", *r);
}

let x = 42;
peek(&x);   // 42
```

`&x` is an immutable borrow: you can only read, not write.

## Borrowing Rules (The Book-Borrowing Analogy)

Think of a variable as a book:

1. **You can lend out many "read-only copies" at once**—multiple `&x` can coexist, everyone reads but no one writes.
2. **"Writable lending" to only one person at a time**—while `&mut x` is borrowed, no other borrows are allowed.
3. **During read-only borrows, you can't lend writable**—while someone is reading, you can't let someone modify at the same time.
4. **Can't touch the book while it's borrowed**—while the borrow is alive, the source variable itself can't be reassigned (assignment counts as writing).

Violate any of these, and the compiler reports an error outright—the program won't even compile:

```aero
let x = 10;
let r1 = &x;
let r2 = &x;      // OK, multiple read-only borrows

// let r3 = &mut x;   // error: already has a read-only borrow, can't mutably borrow
// x = 20;            // error: r1 is still alive, can't write x
print(*r1);
```

This rule eliminates the most insidious bugs like "dangling pointers" and "data races" at compile time.

## When Does a Borrow End: NLL

A borrow doesn't "live until the variable's scope ends," but lives until its **last use**. This rule is called NLL (Non-Lexical Lifetimes).

```aero
let x = 10;
let r = &x;
print(*r);   // r's last use is here
x = 20;      // OK! r has ended, now we can write x
print(x);
```

Does this look like it violates rule 4? No—after `print(*r)`, `r` "returns the book," the borrow ends, and you can write freely afterward. **A borrow only lives until the place where it's last used**—this is the most practical feature of Aero's borrow checker.

## The Borrow Target Must Be a Variable

```aero
// let r = &(1 + 2);   // error: borrow target must be a variable
```

A reference must point to a real, existing variable, not a temporary expression (temporary values would be gone once lent out).

## Reference to a Reference? Don't Think About It Yet

Aero 0.1's borrow system is a streamlined version: it supports single-level `&T` / `&mut T`, but **does not support** nesting references, storing references in arrays, or using references as return values. Don't force it—the compiler will tell you what doesn't work. The complete version of this system (closer to Rust's borrow checking) is on the roadmap; 0.1 first ensures the most common scenarios work well.

## Exercises

1. Write `swap(a: &mut i64, b: &mut i64)` that swaps the values of two variables; verify with `let x = 1; let y = 2; swap(&mut x, &mut y);`.
2. Write `increment_all` that uses references to add 1 to each element of an array (hint: `let mut i = 0; while (i < 3) { arr[i] = arr[i] + 1; ... }`—this one works without references too, think about why).
3. Deliberately write code that "writes to the source variable during a borrow," and see what the compiler error looks like.
4. Verify NLL: does `let r = &x; print(*r); x = 100;` compile? What about adding `x = 100;` after `let r = &x; print(*r);`?
