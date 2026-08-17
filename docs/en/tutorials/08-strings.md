# 08 Strings

## String Literals

```aero
let name = "Aero";
print("hello\n");   // \n is a newline
```

The string type is called `str`, and literals use double quotes. Escape sequences:

| Sequence | Meaning |
| --- | --- |
| `\n` | Newline |
| `\t` | Tab |
| `\"` | Double quote |
| `\\` | Backslash |

```aero
print("say \"hi\"\n");   // say "hi"
print("a\tb\n");         // a then tab then b
```

## Concatenation: `+`

```aero
print("%s\n", "foo" + "bar");   // foobar
print("%s\n", "a" + "b" + "c"); // abc
```

`+` concatenates strings. Two details:

- Literal + literal (`"foo" + "bar"`) is merged into a single constant at **compile time** — zero runtime overhead.
- If at least one side is a variable, it's a **runtime concatenation**: new memory is allocated, and you must `str_free` it when done (see "Who Is Responsible for Freeing" below).

```aero
let s = "x";
print("%s\n", s + "y");   // xy (runtime concatenation, result needs str_free)
```

## Comparison

```aero
let a = "abc";
let b = "abd";
print(a == "abc");   // true
print(a != b);       // true
print(a < b);        // true, lexicographic order
print(a > b);        // false
print(a <= "abc");   // true
print(a >= b);       // false
```

All six comparison operators work with strings, comparing **lexicographically** (byte by byte). This is a distinctive feature of Aero's string system — many languages only support `==` and `!=` for strings.

## Length and Indexing

```aero
let s = "hello";
print(len(s));   // 5
print(s[0]);     // 104, the byte value of 'h'
print(s[1]);     // 101, 'e'
```

- `len(s)` returns the number of characters (bytes).
- `s[i]` returns the i-th byte, with type `i64` (Aero 1.1.0 does not have a single-character type). The ASCII code for `'h'` is 104, so `s[0]` is 104.

If you want to print a specific character, `%c` won't work — Aero 1.1.0's format string only supports `%d` and `%s`. To convert a byte back to a string, use `substr`:

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
print("%s\n", substr(s, 0, 100));  // hello world (out-of-bounds is automatically clamped)
```

`substr(s, start, end)` extracts the portion from `start` to `end` (exclusive of `end`). Bounds are automatically clamped to the string length; if `end <= start`, an empty string is returned. The return value is newly allocated memory — call `str_free` when done.

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
- `str_to_int(s)`: string → integer (under the hood it uses C's `strtoll`; returns 0 on parse failure).

## Searching and Comparing

```aero
let s = "hello world";

print("%d\n", str_contains(s, "world"));   // 1 (true)
print("%d\n", str_contains(s, "xyz"));     // 0 (false)

print("%d\n", str_find(s, "world"));       // 6, position
print("%d\n", str_find(s, "xyz"));         // -1, not found

print("%d\n", str_cmp("abc", "abd"));      // negative ("abc" is smaller)
print("%d\n", str_cmp("abc", "abc"));      // 0
```

- `str_contains(haystack, needle)`: boolean, whether haystack contains needle.
- `str_find(haystack, needle)`: the first occurrence position of needle, or -1 if not found.
- `str_cmp(a, b)`: C-style comparison, returns a negative/zero/positive number.

## Who Is Responsible for Freeing (Important)

String memory comes from two sources:

1. **Literals** (`"hello"`, compile-time concatenation results): stored in the program's read-only data section — **never need to be freed**, and **cannot be freed**.
2. **Runtime-allocated results** (`int_to_str`, `substr`, runtime `+` concatenation): newly allocated memory via `malloc` — **must be freed with `str_free`** when done.

```aero
// Correct example
let a = int_to_str(100);
print("%s\n", a);
str_free(a);          // free

// Wrong example: not freeing in a loop — memory keeps growing
let i = 0;
while (i < 1000) {
    let t = int_to_str(i);   // allocates each time
    print("%s\n", t);
    // str_free(t);          // forgot to free → memory leak
    i = i + 1;
}
```

The rule is simple: **Only literals need no management; for strings returned by function calls, unless you can prove they are literals, call `str_free` when done.**

This "who allocates, who frees" discipline leads to a more elegant solution in Chapter 10 (Arena).

## Exercises

1. Print `len("Aero")` and verify it is 4.
2. Split `"Hello, Aero!"` into `"Hello"` and `"Aero"` using `substr` and print each separately.
3. Use `str_to_int` to convert `"2026"` to a number, add 1, then convert it back to a string with `int_to_str` and print it.
4. Determine whether the result of `"a" + "b"` needs `str_free`, then write code to verify your judgment.