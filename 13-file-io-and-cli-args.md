# 13 File IO and Command-Line Arguments

## Reading Files

```aero
let content = read_file("data.txt");
print("%s\n", content);
```

`read_file(path)` reads the entire file and returns it as a string. **When the file doesn't exist or reading fails, it returns an empty string** (not an error—it returns an empty string; note the distinction).

An empty file also reads as an empty string—so "empty string" might mean "the file is empty" or "the read failed." To tell them apart, check whether the file exists first, or design a convention around return values.

## Writing Files

```aero
let n = write_file("out.txt", "hello world");
print("%d\n", n);   // 11, the number of bytes written
```

`write_file(path, contents)` writes the contents to the file and returns the number of bytes written; returns `-1` on failure.

## A Small Tool: Copying a File

```aero
// copy.aero: copy argument 1 to argument 2
let src = read_file(arg(1));
if (len(src) == 0) {
    print("cannot read %s\n", arg(1));
    // there's no return statement to exit the main program here; in 0.1 we just keep going
}
let n = write_file(arg(2), src);
print("wrote %d bytes\n", n);
```

Note: `arg(1)` and `arg(2)` are command-line arguments, covered in the next section.

## Command-Line Arguments

The executable produced by `aero build` can take arguments at runtime:

```
myapp.exe input.txt output.txt
```

Inside the program, use two built-in functions to get the arguments:

```aero
let n = arg_count();   // the number of arguments
print("arg count: %d\n", n);

print("%s\n", arg(0));   // the program's own path
print("%s\n", arg(1));   // the first argument
print("%s\n", arg(2));   // the second argument
```

Rules:

- `arg_count()`: the number of arguments (**excluding** the program name itself).
- `arg(i)`: the i-th argument (starting from 0); **returns an empty string when out of bounds**.
- `arg(0)` is the program's own path; the real "first argument" is `arg(1)`.

## A Complete Example: A Simple grep

Write a small tool that "finds a substring in a file":

```aero
// find.aero: find <file> <substring>, outputs the line numbers of matches
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

`atoi` is the C standard library's string-to-integer function; we don't actually need it here, it just demonstrates how to declare. Using `arg_count` to check the number of arguments is a good habit:

```aero
if (arg_count() < 2) {
    print("usage: find <file> <substring>\n");
}
```

## No Arguments Under JIT

When you run `aero run file.aero`, `arg_count()` is 0 and `arg(i)` returns an empty string—JIT mode doesn't pass command-line arguments. Command-line arguments only make sense in a standalone exe from `aero build`.

## Exercises

1. Write a program: read `data.txt` and print its character count (using `len`).
2. Write a program: take two command-line arguments (strings of two numbers), convert them to integers, add them, and print the result.
3. Write a program: read a file and count how many lines it has (hint: count the number of `\n`; you can use `str_find` in a loop).
4. Use `write_file` to generate a 100-line file (concatenate strings in a loop), then use the program from exercise 3 to verify the line count.
