# 08 Strings

## String Literals

```aero
let name = "Aero";
print("hello\n");   // \n is a newline
```

The string type is called `str`, and literals use double quotes. Escape characters:

| Notation | Meaning |
| --- | --- |
| `\n` | Newline |
| `\t` | Tab |
| `\"` | Double quote |
| `\\` | Backslash |

```aero
print("say \"hi\"\n");   // say "hi"
print("a\tb\n");         // a, then a tab, then b
```

## Concatenation: `+`

```aero
print("%s\n", "foo" + "bar");   // foobar
print("%s\n", "a" + "b" + "c"); // abc
```

`+` can join strings together. Two details:

- Literal + literal (`"foo" + "bar"`) is merged into a single constant **at compile time**, with zero runtime cost.
- As long as one side is a variable, it's a **runtime concatenation**: new memory is allocated, and you must `str_free` it when done (see "Who Is Responsible for Freeing" below).

```aero
let s = "x";
print("%s\n", s + "y");   // xy (runtime concatenation, result must be str_free'd)
```

## Comparison

```aero
let a = "abc";
let b = "abd";
print(a == "abc");   // true
print(a != b);       // true
print(a < b);        // true, by lexicographic order
print(a > b);        // false
print(a <= "abc");   // true
print(a >= b);       // false
```

All six comparison operators are supported for strings, and the comparison is **lexicographic** (byte by byte). This is a distinguishing feature of Aero's string system — in many languages strings only have `==` and `!=`.

## Length and Indexing

```aero
let s = "hello";
print(len(s));   // 5
print(s[0]);     // 104, the byte value of 'h'
print(s[1]);     // 101, 'e'
```

- `len(s)` returns the number of characters (bytes).
- `s[i]` returns the i-th byte, of type `i64` (Aero 0.1 has no single-character type). The ASCII code of `'h'` is 104, so `s[0]` is 104.

If you want to print a character itself, you can't use `%c` — Aero 0.1's format strings only support `%d` and `%s`. To convert a byte back to a string you can use `substr`:

```aero
let s = "hello";
print("%s\n", substr(s, 1, 2));   // e
```

## Substrings: substr

```aero
let s = "hello world";
print("%s\n", substr(s, 0, 5));    // hello
print("%s\n", substr(s, 6, 11));   // world
print("%s\n", substr(s, 3, 3));    // (empty string, end <= start)
print("%s\n", substr(s, 0, 100));  // hello world (out-of-bounds auto-truncated)
```

`substr(s, start, end)` takes the portion from `start` to `end` (not including `end`). The bounds are automatically clamped to the string length, and `end <= start` yields an empty string. The return value is newly allocated memory; `str_free` it when done.

## Converting Between Numbers and Strings

```aero
let n = int_to_str(42);
print("%s\n", n);   // 42
str_free(n);

let m = str_to_int("123");
print("%d\n", m);   // 123

print("%d\n", str_to_int("abc"));   // 0, returns 0 on parse failure
```

- `int_to_str(n)`: integer → string.
- `str_to_int(s)`: string → integer (backed by C's strtoll, returns 0 on parse failure).

## Search and Comparison

```aero
let s = "hello world";

print("%d\n", str_contains(s, "world"));   // 1 (true)
print("%d\n", str_contains(s, "xyz"));     // 0 (false)

print("%d\n", str_find(s, "world"));       // 6, the position
print("%d\n", str_find(s, "xyz"));         // -1, not found

print("%d\n", str_cmp("abc", "abd"));      // negative (abc is smaller)
print("%d\n", str_cmp("abc", "abc"));      // 0
```

- `str_contains(haystack, needle)`: boolean, whether haystack contains needle.
- `str_find(haystack, needle)`: the position of the first occurrence of needle, or -1 if not found.
- `str_cmp(a, b)`: C-style comparison, returns negative / 0 / positive.

## Who Is Responsible for Freeing (Important)

There are two sources of string memory:

1. **Literals** (`"hello"`, results of compile-time concatenation): stored in the program's read-only data section, **never need to be freed**, and **cannot be freed**.
2. **Runtime-allocated results** (`int_to_str`, `substr`, runtime `+` concatenation): new memory allocated with `malloc`, **must be `str_free`d when done**.

```aero
// Correct example
let a = int_to_str(100);
print("%s\n", a);
str_free(a);          // Free it

// Wrong example: not freeing in a loop, memory keeps growing
let i = 0;
while (i < 1000) {
    let t = int_to_str(i);   // Allocated each time
    print("%s\n", t);
    // str_free(t);          // Forgot to free → memory leak
    i = i + 1;
}
```

The rule is simple: **only literals can be ignored; for strings returned by function calls, unless you can prove it's a literal, `str_free` it when done**.

This set of "whoever allocates, frees" rules will get a more elegant answer in Aero in Chapter 10 (Arena).

## Exercises

1. Print `len("Aero")` and verify it's 4.
2. Split `"Hello, Aero!"` into `"Hello"` and `"Aero"` using `substr` and print them separately.
3. Use `str_to_int` to convert `"2026"` to a number, add 1, then convert it back to a string with `int_to_str` and print.
4. Decide whether the result of `"a" + "b"` needs `str_free`, then write code to verify your judgment.
