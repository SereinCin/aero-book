# 04 Flow Control

## if / else

```aero
let x = 10;
if (x > 0) {
    print("positive\n");
} else {
    print("non-positive\n");
}
```

The condition goes inside parentheses and must be a `bool` expression. `else` can be omitted:

```aero
let x = 10;
if (x > 100) {
    print("big\n");
}
// Does nothing when x is not greater than 100
```

A number like `x` cannot be used directly as a condition. `if (x)` is legal in C (non-zero means true), but in Aero it is a compile error — you must write `if (x != 0)`. This is intentional: types are strict, no implicit conversions.

## if / else if Chains

```aero
let score = 85;
if (score >= 90) {
    print("A\n");
} else if (score >= 80) {
    print("B\n");
} else if (score >= 70) {
    print("C\n");
} else {
    print("D\n");
}
```

It searches from top to bottom for the first branch whose condition is true and executes it, skipping the rest.

## while Loops

```aero
let i = 0;
while (i < 5) {
    print("%d\n", i);
    i = i + 1;
}
// Outputs 0 1 2 3 4
```

Remember to update the condition variable inside the loop body, otherwise you get an infinite loop. Press `Ctrl+C` to interrupt a program stuck in an infinite loop.

The condition of `while`, just like `if`, must be a bool:

```aero
let i = 10;
while (i > 0) {
    i = i - 1;
}
print("done\n");
```

## Combining Them

```aero
// Print all numbers between 1 and 100 that are divisible by 3
let i = 1;
while (i <= 100) {
    if (i % 3 == 0) {
        print("%d\n", i);
    }
    i = i + 1;
}
```

`%` is the modulo (remainder) operator. `i % 3 == 0` means i is divisible by 3.

## What's Missing: break and continue

Aero 0.1's while **does not have** `break` or `continue`. To exit a loop early, you can only use condition control:

```aero
// Simulating break: find the first multiple of 7 greater than 100
let i = 1;
let found = 0;   // 0 = false, 1 = true, Aero 0.1 uses integers to simulate
while (i <= 1000 && found == 0) {
    if (i % 7 == 0) {
        print("%d\n", i);
        found = 1;
    }
    i = i + 1;
}
```

This is a known shortcoming of the current version and will be added later.

## Scope Reminder

The `{ }` of if and while are also blocks, and variables declared inside the block go out of scope when the block ends (covered in Chapter 2). Note the cost of declaring variables in a loop:

```aero
let total = 0;
let i = 0;
while (i < 10) {
    let square = i * i;   // Re-declared every iteration
    total = total + square;
    i = i + 1;
}
print(total);   // 285
```

`square` is recreated every iteration, used and discarded — that's fine. But if you declare a variable inside the loop that needs to "persist across iterations", it will be reset over and over — this is one of the most common traps for beginners:

```aero
let i = 0;
while (i < 5) {
    let sum = 0;          // Wrong! sum resets to 0 every iteration
    sum = sum + i;
    i = i + 1;
}
```

`sum` should be declared outside the loop.

## Exercises

1. Print all odd numbers between 1 and 20.
2. Use a while loop to compute the sum of 1 to 100, output 5050.
3. Write a program: start from 1, multiply by 2 each time, print, until it exceeds 1000. See how many numbers it prints.
4. Use if/else if to map a 0–100 score to a grade (A/B/C/D/E) and print it.
