# 11 Tensors and Matrices

## Why Tensors

One of Aero's positions is "natively supporting AI computation." The core of AI computation is matrix operations. Aero has a built-in tensor (Tensor) type and matrix multiplication `matmul`—this is what sets it apart from ordinary systems languages.

A Tensor can be understood as a "multi-dimensional array that natively supports math operations." `tensor(2, 3)` is a 2-row, 3-column matrix.

## Creation and Read/Write

```aero
let a = tensor(2, 3);   // 2 rows, 3 columns, zero-initialized
print(a[0][0]);   // 0
print(a[1][2]);   // 0
```

All elements are initialized to 0. Read/write with multi-dimensional indexing:

```aero
let a = tensor(2, 2);
a[0][1] = 5;
print(a[0][1]);   // 5
print(a[1][1]);   // 0, untouched
```

## One-Dimensional Tensors: Vectors

```aero
let v = tensor(4);   // vector of length 4
v[2] = 7;
print(v[2]);   // 7
print(v[0]);   // 0
```

Indexing a one-dimensional tensor once yields a scalar.

## Subtensors

Indexing a multi-dimensional tensor **once** yields a "subtensor" (one fewer dimension), which you can keep indexing:

```aero
let a = tensor(2, 3);
a[1][2] = 9;
let row = a[1];        // take row 1, type is a 3-element one-dimensional tensor
print(row[2]);         // 9
```

## Matrix Multiplication: matmul

```aero
let a = tensor(2, 2);
let b = tensor(2, 2);
a[0][0] = 1; a[0][1] = 2;
a[1][0] = 3; a[1][1] = 4;
b[0][0] = 5; b[0][1] = 6;
b[1][0] = 7; b[1][1] = 8;

let c = matmul(a, b);
print(c[0][0]);   // 19 = 1*5 + 2*7
print(c[0][1]);   // 22 = 1*6 + 2*8
print(c[1][1]);   // 50 = 3*6 + 4*8
```

## Dimension Rules

Matrix multiplication requires **the number of columns of the left matrix = the number of rows of the right matrix**. The result's row count = the left matrix's row count, column count = the right matrix's column count:

```aero
let a = tensor(1, 3);   // 1 row, 3 columns
let b = tensor(3, 2);   // 3 rows, 2 columns
let c = matmul(a, b);   // OK: 3 == 3
// c is 1 row, 2 columns
print(c[0][1]);
```

If the dimensions don't match, **the compiler reports an error at compile time**:

```aero
let a = tensor(2, 3);
let b = tensor(2, 3);
// let c = matmul(a, b);   // compile error: 2×3 and 2×3 cannot be multiplied (left cols 3 ≠ right rows 2)
```

If the element types of the two tensors don't match, it also reports an error.

## matmul's Arguments Must Be Tensors

```aero
// let c = matmul(1, 2);   // compile error: matmul arguments must be two-dimensional tensors
```

## What's Not There Yet

Aero 0.1's Tensor only has zero-initialization and matmul; no addition, transposition, dot product, etc. It's the first brick of "AI-native"—later versions will gradually add operators and connect to the GPU.

## GPU Kernels: A Hook for the Future

Aero supports declaring GPU kernels:

```aero
extern "gpu" fn add_kernel(a: i64) {}
```

`extern "gpu"` declares a function on the GPU. In 0.1 it's just "declaration is legal, doesn't execute, can't be called from CPU code"; the real NVPTX backend is on the roadmap. Just know this thing exists; you won't need it for now.

## Exercises

1. Create `tensor(3, 3)`, set all diagonal elements to 1 (identity matrix), and print `m[2][2]` to verify.
2. Write a 2×3 and a 3×2 matrix, do `matmul`, and hand-verify each element of the result.
3. Try `matmul` on two matrices with mismatched dimensions, and read the compiler error.
