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

The condition is written in parentheses and must be a `bool` expression. The `else` clause can be omitted:

```aero
let x = 10;
if (x > 100) {
    print("big\n");
}
// x is not greater than 100, so nothing happens
```

A numeric value like `x` cannot be used directly as a condition. `if (x)` is valid in C (non-zero is truthy), but in Aero it's a compile error — you must write `if (x != 0)`. This is intentional: strict typing, no implicit conversions.

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

It scans from top to bottom, executing the first branch whose condition is true, and skipping the rest.

## while Loops

```aero
let i = 0;
while (i < 5) {
    print("%d\n", i);
    i = i + 1;
}
// outputs 0 1 2 3 4
```

Remember to update the condition variable inside the loop body, otherwise you'll get an infinite loop. Press `Ctrl+C` to interrupt an infinite loop.

The while condition, like if, must be a bool:

```aero
let i = 10;
while (i > 0) {
    i = i - 1;
}
print("done\n");
```

## Combined Usage

```aero
// Print all numbers from 1 to 100 that are divisible by 3
let i = 1;
while (i <= 100) {
    if (i % 3 == 0) {
        print("%d\n", i);
    }
    i = i + 1;
}
```

`%` is the modulo (remainder) operator. `i % 3 == 0` means i is divisible by 3.

## break and continue

`break` ends the whole loop immediately; `continue` skips the rest of the current round and moves straight on to the next one:

```aero
// break: stop at the first multiple of 7 within 100
let i = 1;
while (i <= 100) {
    if (i % 7 == 0) {
        print("%d\n", i);   // 7
        break;
    }
    i = i + 1;
}
```

```aero
// continue: skip multiples of 3 and add up the rest
let i = 0;
let sum = 0;
while (i < 10) {
    i = i + 1;
    if (i % 3 == 0) { continue; }
    sum = sum + i;   // 1+2+4+5+7+8+10 = 37
}
print("%d\n", sum);
```

Both may only appear inside a loop body; using either one outside a loop is a compile error.

## Scope Reminder

The `{ }` in if and while also form blocks — variables declared inside a block become invalid when the block ends (as covered in Chapter 2). Note the cost of declaring variables inside a loop:

```aero
let total = 0;
let i = 0;
while (i < 10) {
    let square = i * i;   // redeclared each iteration
    total = total + square;
    i = i + 1;
}
print(total);   // 285
```

`square` is created fresh each iteration and discarded — no problem. But if you declare a variable inside a loop that you intend to "persist across iterations," it will be reset every time — this is one of the most common pitfalls for beginners:

```aero
let i = 0;
while (i < 5) {
    let sum = 0;          // wrong! sum is reset to 0 each iteration
    sum = sum + i;
    i = i + 1;
}
```

`sum` should be declared outside the loop.

## Exercises

1. Print all odd numbers from 1 to 20.
2. Use a while loop to compute the sum of 1 to 100, output 5050.
3. Write a program: start from 1, multiply by 2 each iteration, print, until exceeding 1000. See how many numbers it prints.
4. Use if/else if to map a percentage grade to a letter grade (A/B/C/D/E), and print it out.