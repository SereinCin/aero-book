# 13 File I/O and Command Line Arguments

## Reading a File

```aero
let content = read_file("data.txt");
print("%s\n", content);
```

`read_file(path)` reads the entire file and returns it as a string. **If the file does not exist or reading fails, it returns an empty string** (not an error, just an empty string — be aware of this distinction).

An empty file also returns an empty string — so an "empty string" could mean "the file is empty" or "reading failed." To distinguish between the two, check if the file exists first, or use a return value convention.

## Writing a File

```aero
let n = write_file("out.txt", "hello world");
print("%d\n", n);   // 11, number of bytes written
```

`write_file(path, contents)` writes the contents to a file and returns the number of bytes written; returns `-1` on failure.

## Mini Tool: File Copy

```aero
// copy.aero: copy arg1 to arg2
let src = read_file(arg(1));
if (len(src) == 0) {
    print("cannot read %s\n", arg(1));
    // There is no way to return from the main program in 1.1.0, so execution continues
}
let n = write_file(arg(2), src);
print("wrote %d bytes\n", n);
```

Note: `arg(1)`, `arg(2)` are command-line arguments, covered in the next section.

## Command Line Arguments

An executable produced by `aero build` can accept arguments at runtime:

```
myapp.exe input.txt output.txt
```

Use two built-in functions to access arguments:

```aero
let n = arg_count();   // number of arguments
print("arg count: %d\n", n);

print("%s\n", arg(0));   // program path itself
print("%s\n", arg(1));   // first argument
print("%s\n", arg(2));   // second argument
```

Rules:

- `arg_count()`: total number of arguments (**including** the program name itself, equivalent to C's `argc`).
- `arg(i)`: the i-th argument (starting from 0), **returns an empty string if out of bounds**.
- `arg(0)` is the program's own path; the actual "first argument" is `arg(1)`.

## Complete Example: Simple Grep

Write a small tool that searches for a substring in a file:

```aero
// find.aero: find <file> <substring>, outputs matching line numbers
extern "C" fn atoi(s: str) -> i32 = "atoi";

let path = arg(1);
let needle = arg(2);
let content = read_file(path);

if (str_contains(content, needle)) {
    print("found in %s\n", path);
} else {
    print("not found\n");
}
```

Compile:

```
aero build find.aero
```

Run:

```
find.exe data.txt aero
```

`atoi` is a C standard library function that converts a string to an integer; it's not used here, just shown as an example of declaring functions. It's good practice to check the argument count with `arg_count`:

```aero
if (arg_count() < 2) {
    print("usage: find <file> <substring>\n");
}
```

## No Arguments Under JIT

When running `aero run file.aero`, `arg_count()` returns 0, and `arg(i)` returns an empty string — the JIT mode does not pass command-line arguments. Command-line arguments are only meaningful in a standalone executable produced by `aero build`.

## Exercises

1. Write a program that reads `data.txt` and prints its character count (using `len`).
2. Write a program that accepts two command-line arguments (two numeric strings), converts them to integers, adds them, and prints the result.
3. Write a program that reads a file and counts how many lines it contains (hint: count the number of `\n` characters, using `str_find` in a loop).
4. Use `write_file` to generate a 100-line file (build a string in a loop), then verify the line count using the program from exercise 3.