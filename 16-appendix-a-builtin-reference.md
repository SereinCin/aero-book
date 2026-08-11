# 16 Appendix A: Builtin Function Reference

## Output

| Function | Description |
| --- | --- |
| `print(x)` | Print a value (integers printed directly, no newline) |
| `print("format string", a, b, ...)` | Print according to format string: `%d` for integers, `%s` for strings |

## Assertions and Testing

| Function | Description |
| --- | --- |
| `assert(cond)` | If the condition is false, print a diagnostic and terminate the program (non-zero exit code) |
| `assert_eq(a, b)` | Fail and terminate if the two values are not equal |

## Strings

| Function | Description |
| --- | --- |
| `len(s)` | String length (in bytes) |
| `s1 + s2` | Concatenation (literal concatenation is zero-cost; with variables it's a runtime allocation, result needs `str_free`) |
| `s1 == s2` / `!=` / `<` / `>` / `<=` / `>=` | Lexicographic comparison |
| `s[i]` | Numeric value of the i-th byte (`i64`) |
| `int_to_str(n)` | Integer → string (**runtime allocation, needs `str_free`**) |
| `str_to_int(s)` | String → integer (returns 0 on parse failure) |
| `substr(s, start, end)` | Take substring `[start, end)`, bounds auto-clamped (**needs `str_free`**) |
| `str_contains(hay, needle)` | Whether hay contains needle (`bool`) |
| `str_find(hay, needle)` | Position of first occurrence of needle, returns -1 if not found |
| `str_cmp(a, b)` | C-style comparison, returns negative / 0 / positive |
| `str_free(s)` | Free a runtime-allocated string |

## Numeric Computation

| Function | Description |
| --- | --- |
| `matmul(a, b)` | 2D tensor matrix multiplication (left columns = right rows, checked at compile time) |

## Files and Command Line

| Function | Description |
| --- | --- |
| `read_file(path)` | Read an entire file as a string, returns empty string on failure |
| `write_file(path, contents)` | Write a file, returns byte count, returns -1 on failure |
| `arg_count()` | Number of command-line arguments (excluding program name) |
| `arg(i)` | The i-th argument (`arg(1)` is the first argument), returns empty string if out of bounds |

## Operator Reference

| Category | Operators |
| --- | --- |
| Arithmetic | `+` `-` `*` `/` unary negation `-x` |
| Comparison | `<` `>` `<=` `>=` `==` `!=` |
| Logical | `&&` (and) `||` (or), both short-circuit |
| Reference | `&x` (immutable borrow) `&mut x` (mutable borrow) `*r` (dereference) |
| Index | `a[i]` read and write; tuple index must be a constant |
