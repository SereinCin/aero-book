# 11 Tensor and Matrix

## Why Tensor

One of Aero's design goals is "native support for AI computation." The core of AI computation is matrix operations. Aero has built-in tensor types and matrix multiplication `matmul`, which distinguishes it from ordinary system languages.

A tensor can be thought of as a "multi-dimensional array with built-in mathematical operations." `tensor(2, 3)` is a matrix with 2 rows and 3 columns.

## Creation and Indexing

```aero
let a = tensor(2, 3);   // 2 rows, 3 columns, zero-initialized
print(a[0][0]);   // 0
print(a[1][2]);   // 0
```

All elements are initialized to 0. Use multi-dimensional indexing to read and write:

```aero
let a = tensor(2, 2);
a[0][1] = 5;
print(a[0][1]);   // 5
print(a[1][1]);   // 0, untouched
```

## One-Dimensional Tensor: Vector

```aero
let v = tensor(4);   // vector of length 4
v[2] = 7;
print(v[2]);   // 7
print(v[0]);   // 0
```

Indexing a one-dimensional tensor once yields a scalar.

## Sub-tensors

Indexing a multi-dimensional tensor **once** yields a "sub-tensor" (one dimension fewer), which can still be indexed further:

```aero
let a = tensor(2, 3);
a[1][2] = 9;
let row = a[1];        // get row 1, type is a 3-element one-dimensional tensor
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

Matrix multiplication requires **the number of columns in the left matrix equals the number of rows in the right matrix**. The result has rows equal to the left matrix's rows and columns equal to the right matrix's columns:

```aero
let a = tensor(1, 3);   // 1 row, 3 columns
let b = tensor(3, 2);   // 3 rows, 2 columns
let c = matmul(a, b);   // OK: 3 == 3
// c is 1 row, 2 columns
print(c[0][1]);
```

If dimensions don't match, **the compiler reports an error directly**:

```aero
let a = tensor(2, 3);
let b = tensor(2, 3);
// let c = matmul(a, b);   // Compile error: 2×3 and 2×3 cannot be multiplied (left columns 3 ≠ right rows 2)
```

A type mismatch between the two tensors will also cause an error.

## matmul Arguments Must Be Tensors

```aero
// let c = matmul(1, 2);   // Compile error: matmul arguments must be two-dimensional tensors
```

## What's Not Here Yet

In Aero 1.1.0, Tensor only has zero-initialization and matmul — no addition, transpose, dot product, etc. It is the first building block of "AI-native" support — subsequent versions will gradually add operators and connect to GPU.

## GPU Kernel: A Placeholder for the Future

Aero supports declaring GPU kernels:

```aero
extern "gpu" fn add_kernel(a: i64) {}
```

`extern "gpu"` declares a function that lives on the GPU. In 1.1.0 it is merely "valid syntax, doesn't execute, cannot be called from CPU code" — the real NVPTX backend is on the roadmap. Just know that it exists; you won't use it yet.

## Exercises

1. Create `tensor(3, 3)`, set all diagonal elements to 1 (identity matrix), and print `m[2][2]` to verify.
2. Write two matrices of size 2×3 and 3×2, perform `matmul`, and manually verify each element of the result.
3. Try `matmul` with two matrices whose dimensions don't match, and read the compile error.