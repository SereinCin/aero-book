# 05 Functions

## Defining a Function

```aero
fn add(a: i64, b: i64) -> i64 {
    return a + b;
}

print(add(1, 2));   // 3
```

Breaking down the structure:

- The `fn` keyword declares a function;
- `add` is the function name;
- `(a: i64, b: i64)` is the parameter list, each parameter is `name: type`;
- `-> i64` is the return type;
- `{ ... }` is the function body;
- `return` returns the value.

## Functions Without a Return Value

The return type can be omitted; such a function "does something but produces no result":

```aero
fn greet(name: i64) {
    print("hello %d\n", name);
}

greet(7);   // hello 7
```

## Parameters Are Copied

Aero's function parameters are passed by value: modifying a parameter inside the function does not affect the caller's variable.

```aero
fn bump(n: i64) {
    n = n + 1;
    print("inside: %d\n", n);   // inside: 11
}

let x = 10;
bump(x);
print("outside: %d\n", x);      // outside: 10, unchanged
```

To modify the caller's variable, use references (Chapter 9) or have the function return the new value.

## Recursion

A function can call itself:

```aero
fn fact(n: i64) -> i64 {
    if (n <= 1) { return 1; }
    return n * fact(n - 1);
}

print(fact(5));   // 120
```

This is the most classic recursion example. A recursive function must have a "termination condition" (`n <= 1`), otherwise it recurses infinitely until the stack overflows.

## Forward Calls: Functions Can Be Defined Later

Aero scans in two passes at compile time: the first pass collects all function signatures, the second pass generates code. So functions **can call each other and can call functions defined later**:

```aero
fn is_even(n: i64) -> i64 {
    if (n == 0) { return 1; }
    return is_odd(n - 1);
}

fn is_odd(n: i64) -> i64 {
    if (n == 0) { return 0; }
    return is_even(n - 1);
}

print(is_even(10));   // 1 (true)
print(is_odd(7));     // 1 (true)
```

When `is_even` calls `is_odd`, `is_odd` hasn't been defined yet — that's fine, the compiler already knows its signature.

## Scope Rules

- The function body is an independent block: variables and parameters inside the function are invisible outside.
- Function parameters can be reassigned inside the function (the `n = n + 1` in `bump` above).
- **Function definitions cannot be nested**: you cannot write `fn` inside a function body. All functions are top-level definitions.

```aero
fn outer() {
    // fn inner() {}   // Error: function definitions are not allowed to be nested inside a function body
}
```

## Return Value Type Checking in Function Calls

A common confusion: why does it matter whether `return` has a value or not?

- If `-> i64` is declared, `return` inside the function must carry a value.
- If no return type is declared, `return` cannot carry a value.

```aero
fn f() -> i64 {
    return 1;   // Must carry a value
}

fn g() {
    return;     // Empty return, exits early
    // return 1;  // Error
}
```

## Exercises

1. Write a `max2(a, b) -> i64` that returns the larger number; then write `max3(a, b, c)` using `max2` twice.
2. Write a fibonacci function `fib(n) -> i64` (fib(0)=0, fib(1)=1, fib(n)=fib(n-1)+fib(n-2)), print fib(10).
3. Write a function `is_prime(n) -> i64` (returns 1 for prime, 0 for non-prime), print all primes between 2 and 30.
4. Verify: modifying a parameter inside a function does not change the outer variable.
