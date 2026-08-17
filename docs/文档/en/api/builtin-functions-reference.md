# 16 Appendix A: Built-in Functions Quick Reference

## Output

| Function | Description |
| --- | --- |
| `print(x)` | Print a value (integers printed directly, no newline) |
| `print("format", a, b, ...)` | Print with format string: `%d` for integers, `%s` for strings |

## Assertions and Testing

| Function | Description |
| --- | --- |
| `assert(cond)` | If condition is false, print diagnostics and terminate the program (non-zero exit code) |
| `assert_eq(a, b)` | Fail and terminate if the two values are not equal |

## Strings

| Function | Description |
| --- | --- |
| `len(s)` | String length (number of bytes) |
| `s1 + s2` | Concatenation (zero-cost for literals; runtime allocation when variables are involved, result requires `str_free`) |
| `s1 == s2` / `!=` / `<` / `>` / `<=` / `>=` | Lexicographic comparison |
| `s[i]` | Numeric value of the i-th byte (`i64`) |
| `int_to_str(n)` | Integer → string (**runtime allocation, requires `str_free`**) |
| `str_to_int(s)` | String → integer (returns 0 on parse failure) |
| `substr(s, start, end)` | Extract substring `[start, end)`, boundaries are automatically clamped (**requires `str_free`**) |
| `str_contains(hay, needle)` | Whether hay contains needle (`bool`) |
| `str_find(hay, needle)` | Position of the first occurrence of needle, returns -1 if not found |
| `str_cmp(a, b)` | C-style comparison, returns negative / 0 / positive |
| `str_free(s)` | Free a runtime-allocated string |

## Numeric Computation

| Function | Description |
| --- | --- |
| `matmul(a, b)` | Two-dimensional tensor matrix multiplication (left columns = right rows, checked at compile time) |

## File and Command Line

| Function | Description |
| --- | --- |
| `read_file(path)` | Read entire file as a string, returns empty string on failure |
| `write_file(path, contents)` | Write to file, returns number of bytes, returns -1 on failure |
| `arg_count()` | Total number of command-line arguments (including program name, i.e., C's `argc`) |
| `arg(i)` | The i-th argument (`arg(1)` is the first argument), returns empty string if out of bounds |

## Operator Quick Reference

| Category | Operators |
| --- | --- |
| Arithmetic | `+` `-` `*` `/` unary minus `-x` |
| Comparison | `<` `>` `<=` `>=` `==` `!=` |
| Logical | `&&` (and) `||` (or), both short-circuit |
| Reference | `&x` (immutable borrow) `&mut x` (mutable borrow) `*r` (dereference) |
| Indexing | `a[i]` read/write; tuple indices must be constants |

## Vec Dynamic Array

| Function | Description |
| --- | --- |
| `Vec::new()` | Create an empty Vec |
| `Vec::with_cap(n)` | Create a Vec with pre-allocated capacity of n |
| `v.push(x)` | Append an element to the end |
| `v.pop()` | Pop the last element, returns 0 for empty Vec |
| `v.len()` | Return the number of elements |
| `v.is_empty()` | Check if empty |
| `v.get(i)` | Read the i-th element (returns 0 if out of bounds) |
| `v.set(i, x)` | Write the i-th element (ignores write if out of bounds) |

## Collection Types

| Function | Description |
| --- | --- |
| `hash_map_new()` | Create a HashMap |
| `m.insert(k, v)` | Insert a key-value pair |
| `m.get(k, def)` | Get the value for a key, returns `def` if not present |
| `m.contains(k)` | Check if the key exists |
| `m.remove(k)` | Remove a key-value pair |
| `m.len()` | Return the number of entries |
| `btree_map_new()` | Create a BTreeMap (ordered) |
| `bm.keys()` | Get the list of keys (`Vec`) |
| `btree_set_new()` | Create a BTreeSet |
| `bs.contains(x)` | Check if an element is present |
| `linked_list_new()` | Create a LinkedList |
| `ll.push_front(x)` | Insert at the front |
| `ll.push_back(x)` | Insert at the back |
| `ll.front()` | Return the front element (`Option<T>`, returns `None` if empty) |
| `ll.back()` | Return the back element (`Option<T>`, returns `None` if empty) |

## Algorithms

| Function | Description |
| --- | --- |
| `sort(v)` | Sort a Vec in-place (ascending) |
| `reverse(v)` | Reverse a Vec in-place |
| `binary_search(v, x)` | Binary search, returns the index; returns -1 if not found |