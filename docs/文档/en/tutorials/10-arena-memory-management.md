# 10 Arena Memory Management

## Three Approaches to Memory Management

When a program needs to dynamically allocate memory, languages generally offer three approaches:

1. **Manual allocation/freeing** (C's malloc/free): Flexible, but you must remember to free every time. Forget and it leaks, free twice and it crashes.
2. **Garbage Collection** (Java/Python): Automatic, but at runtime there are "stop-the-world" pauses, and memory usage is unpredictable.
3. **Region-based allocation (Arena)**: Request a large block of memory from the system **at once**, the program "allocates sequentially" within it, and **returns the entire block** when done.

Aero takes the third path. This is how its "three-no principle" (no GC, do at compile time what can be done at compile time) manifests in memory: arenas have no runtime pauses and require no individual bookkeeping.

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

`a.alloc(n)` carves out `n` slots from the pool and returns an indexable pointer: `p[0]`, `p[1]` can be used directly. Each "slot" is 8 bytes (i64), so `alloc(2)` actually consumes 16 bytes of pool capacity.

Multiple allocations line up sequentially:

```aero
let a = arena(64);
let p = a.alloc(3);   // first block
let q = a.alloc(2);   // second block, follows after
p[0] = 1;
q[0] = 10;
print(p[0] + q[0]);   // 11
```

## What Happens When the Pool is Full

If `alloc` exceeds the remaining capacity, it calls `abort()` and terminates the program immediately. This is "better safe than sorry" — an arena overflow is the programmer's fault, and it fails loudly rather than silently corrupting memory.

```aero
let a = arena(8);
// let p = a.alloc(2);   // needs 16 bytes, pool only has 8, program terminates immediately
```

## reset: Reuse

```aero
let a = arena(64);
let p = a.alloc(1);
p[0] = 5;

a.reset();            // offset resets to zero: the entire pool can be reallocated

let q = a.alloc(1);
q[0] = 8;
print(q[0]);          // 8
```

`a.reset()` resets the "used marker" back to zero. Note that it does **not clear data** — it merely allows new allocations to overwrite old positions. The old pointer `p` should not be used after reset (the memory it points to can already be overwritten by `q`).

Use case: In a game, you allocate a bunch of temporary objects each frame, call `reset()` at the end of the frame, and start fresh the next frame. Zero fragmentation, zero deallocation overhead.

## Automatic Release at Block End

An arena is tied to its scope. **When the block ends, any arena created inside it is automatically reset** — you don't need to do anything:

```aero
if (true) {
    let a = arena(64);
    let p = a.alloc(10);
    p[0] = 1;
    print(p[0]);   // 1
}
// Here, a's memory has been fully returned, and a no longer exists
print(9);
```

This solves a major problem: the "who allocates, who frees" approach from Chapter 8's string functions becomes "return everything at block end" in the arena world. This is what the whitepaper calls "scope-based region allocator with zero runtime overhead."

## Arena is Not Copyable, Not Movable

```aero
let a = arena(64);
// let b = a;   // Error: Arena cannot be copied or moved
// a = arena(32);  // Error: Arena variable cannot be reassigned
```

Why? An arena is the "sole manager" of its block. Copying it would mean two managers overseeing the same memory, which would be chaos. So it can only live in the scope where it was born, following the scope's lifecycle.

## Arena vs malloc: When to Use Which

- Need to **allocate in batches** and free together → arena, fast and simple.
- Need a **single object to outlive its scope** (e.g., a function returning a heap-allocated string to the caller) → use the string library's malloc approach (`int_to_str` returns + caller calls `str_free`).

Aero 0.1 does not yet have the ability to pass an arena pointer out of a function as a return value, so cross-function memory can only go through the malloc family for now. Both systems coexist in 0.1, each with their own purpose.

## Exercises

1. Write a loop: each iteration allocates 1 slot from an arena and writes the iteration number, then confirm that the arena is automatically freed after the loop ends (using `a` outside the loop should cause a compile error).
2. Cause an arena overflow: capacity 16, `alloc(3)`, observe the program termination behavior.
3. Use an arena to simulate "packing 5 student scores and summing them."
4. Think: Why can the literal `arena(64)` only appear in a `let` initializer (it cannot be placed in an array or passed as a function argument)? Hint: Think about who manages its lifetime.