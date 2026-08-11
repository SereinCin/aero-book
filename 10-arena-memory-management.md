# 10 Arena Memory Management

## Three Approaches to Memory Management

When a program needs to dynamically allocate memory, languages generally offer three approaches:

1. **Manual allocation/free** (C's malloc/free): flexible, but you must remember to free every time. Forget, and you leak; free twice, and you crash.
2. **Garbage collection** (Java/Python): automatic, but the runtime has "stop-to-collect-garbage" pauses, and memory usage is not controllable.
3. **Region allocation (Arena)**: asks the system for a large block of memory **once**, the program "draws from it sequentially," and when done, **returns the whole block**.

Aero takes the third path. This is the embodiment of its "three no's principle" (no GC, do what can be done at compile time) on the memory front: arenas have no runtime pauses and don't require per-item bookkeeping.

## Creating an Arena

```aero
let a = arena(64);
```

`arena(64)` creates a memory pool with a capacity of 64 bytes. This `a` is your "memory manager."

## Allocation: alloc

```aero
let a = arena(64);
let p = a.alloc(2);   // request 2 slots
p[0] = 7;
p[1] = 9;
print(p[0] + p[1]);   // 16
```

`a.alloc(n)` carves out n slots from the pool and returns an indexable pointer: use `p[0]`, `p[1]` directly. Here a "slot" is 8 bytes (i64), so `alloc(2)` actually consumes 16 bytes of pool capacity.

Multiple allocations are laid out side by side:

```aero
let a = arena(64);
let p = a.alloc(3);   // first block
let q = a.alloc(2);   // second block, right after it
p[0] = 1;
q[0] = 10;
print(p[0] + q[0]);   // 11
```

## What Happens When the Pool Is Full

`alloc` that exceeds the remaining capacity calls `abort()` to terminate the program directly. This is "better to kill than to let it slip"—an arena overrun is the programmer's fault, so it dies on the spot rather than silently corrupting memory.

```aero
let a = arena(8);
// let p = a.alloc(2);   // requests 16 bytes, pool only has 8, program terminates directly
```

## reset: Reusing

```aero
let a = arena(64);
let p = a.alloc(1);
p[0] = 5;

a.reset();            // offset reset to zero: the whole pool can be reallocated

let q = a.alloc(1);
q[0] = 8;
print(q[0]);          // 8
```

`a.reset()` moves the "used marker" back to zero. Note it **does not clear data**; it only allows new allocations to overwrite old positions. Don't use the old pointer `p` after reset (the memory it points to can now be overwritten by q).

Scenario: in a game, each frame needs to allocate a bunch of temporary objects; at frame end, `reset()`, and the next frame starts fresh. Zero fragmentation, zero free overhead.

## Automatic Release at Block End

An arena is bound to its scope. **When a brace block ends, the arena created inside is automatically reset**—you don't have to do anything:

```aero
if (true) {
    let a = arena(64);
    let p = a.alloc(10);
    p[0] = 1;
    print(p[0]);   // 1
}
// here a's memory has been returned wholesale, and a no longer exists
print(9);
```

This solves a big problem: the "whoever allocates frees" approach from the Chapter 8 strings becomes "return everything together at block end" in the arena world. This is also what the whitepaper calls "scope-based region allocator, achieving zero runtime overhead."

## Arenas Cannot Be Copied or Moved

```aero
let a = arena(64);
// let b = a;   // error: Arena cannot be copied or moved
// a = arena(32);  // error: Arena variable cannot be reassigned
```

Why? An arena is "the sole manager within the block"; copying one out means two managers manage the same memory, which is a mess. So it can only stay in the scope where it was born, living and dying with that scope.

## Arena vs malloc: When to Use Which

- Need to allocate **in batches** and free together → arena, fast and convenient.
- Need a **single object to outlive the scope** (e.g., a function returns a heap-allocated string to the caller) → use the string library's malloc approach (`int_to_str` returns + caller `str_free`).

Aero 0.1 doesn't yet have the ability to "pass arena pointers to functions and return them," so memory that must survive across functions can only go through the malloc family for now. The two coexist in 0.1, each with its own use.

## Exercises

1. Write a loop: each iteration allocates 1 slot from an arena and writes the iteration number; after the loop ends (block ends), confirm the arena is auto-released (using `a` outside the loop will be a compile error).
2. Trigger an arena overflow: capacity 16, `alloc(3)`, observe the program's termination behavior.
3. Use an arena to simulate "packing 5 students' scores and summing them."
4. Think: why can the `arena(64)` literal only appear in a `let` initializer (it can't be stuffed into an array or used as a function argument)? Hint: think about who manages its lifetime.
