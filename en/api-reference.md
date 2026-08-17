# Aero Language API Reference

## 1. Keyword Reference

| Keyword | Category | Description |
|---|---|---|
| `let` | Control flow | Bind a variable; immutable by default |
| `mut` | Declarations | Mark a binding or reference as mutable |
| `if` / `else` | Control flow | Conditional branching |
| `while` | Control flow | Condition-based loop |
| `loop` | Control flow | Infinite loop, exited via `break` |
| `for` | Control flow | Iteration over ranges or collections |
| `in` | Control flow | Used in `for` loops |
| `return` | Control flow | Return a value from a function |
| `break` | Control flow | Exit a loop early |
| `continue` | Control flow | Skip to next loop iteration |
| `match` | Control flow | Pattern matching |
| `as` | Control flow | Type coercion |
| `fn` | Declarations | Function declaration |
| `struct` | Declarations | Composite data type |
| `union` | Declarations | Overlapping field storage |
| `enum` | Declarations | Tagged union |
| `trait` | Declarations | Interface definition |
| `impl` | Declarations | Implement a trait for a type |
| `type` | Declarations | Type alias |
| `dyn` | Declarations | Dynamic dispatch trait object |
| `const` | Declarations | Compile-time constant |
| `mod` | Declarations | Module declaration |
| `use` | Declarations | Import names into scope |
| `pub` | Declarations | Visibility modifier |
| `crate` | Declarations | Root module reference |
| `extern` | Declarations | External function or block declaration |
| `arena` | Declarations | Arena allocation region |
| `tensor` | Declarations | Multi-dimensional array type |
| `gpu` | Declarations | GPU kernel declaration |

---

## 2. Type System

### Primitive Types

| Type | Description | Size |
|---|---|---|
| `i32` | 32-bit signed integer | 4 bytes |
| `i64` | 64-bit signed integer | 8 bytes |
| `f32` | 32-bit floating point | 4 bytes |
| `f64` | 64-bit floating point | 8 bytes |
| `char` | Single character | 4 bytes |
| `bool` | Boolean (`true` / `false`) | 1 byte |
| `str` | C-style string (`i8*`) | pointer-sized |
| `void` | No value | 0 bytes |

### Literals

| Literal | Pattern | Default Type |
|---|---|---|
| Integer | `[0-9]+` | `i64` |
| Float | `[0-9]+.[0-9]+([eE][+-]?[0-9]+)?` | `f64` |
| Char | `'a'`, `'\n'` | `char` |
| String | `"hello\n"` | `str` (C-style `i8*`) |
| Boolean | `true`, `false` | `bool` |

### Compound Types

- **Struct**: Named product type with fields (`struct Point { x: f64, y: f64 }`)
- **Enum**: Tagged union (`enum Option<T> { Some(T), None }`)
- **Union**: Overlapping storage (`union Value { i: i64, f: f64 }`)
- **Array/Tensor**: Multi-dimensional data (`tensor[f32, 3, 224, 224]`)
- **Function pointer**: `fn(i64, i64) i64`
- **Reference**: `&T` (immutable), `&mut T` (mutable)
- **Trait object**: `dyn TraitName`

### Generics

```aero
fn identity<T>(x: T) -> T {
    return x;
}

struct Pair<A, B> {
    first: A,
    second: B,
}
```

---

## 3. Operator Table (Precedence High to Low)

| Precedence | Operators | Associativity | Description |
|---|---|---|---|
| 1 | `.` `::` `()` `[]` | Left | Member access, path, call, index |
| 2 | `-` `&` `*` `?` | Right | Unary negate, ref, deref (no unary `!`; use `x == false`) |
| 3 | `as` | Left | Type coercion |
| 4 | `*` `/` `%` | Left | Multiplicative |
| 5 | `+` `-` | Left | Additive |
| 6 | `<<` `>>` | Left | Shift (added in 1.1.0) |
| 7 | `>` `>=` `<` `<=` | Left | Comparison |
| 8 | `==` `!=` | Left | Equality |
| 9 | `&` | Left | Bitwise AND (added in 1.1.0) |
| 10 | `^` | Left | Bitwise XOR (added in 1.1.0) |
| 11 | `|` | Left | Bitwise OR (added in 1.1.0) |
| 12 | `&&` | Left | Logical AND |
| 13 | `||` | Left | Logical OR |
| 14 | `=` `=>` `->` | Right | Assignment, match arm, return type |

---

## 4. Builtin Function Reference

### I/O

| Signature | Returns | Description |
|---|---|---|
| `print(s: str)` | `void` | Print string to stdout |
| `read_file(path: str) -> str` | `str` | Read entire file as string |
| `write_file(path: str, data: str) -> i64` | `i64` | Write string to file |

### Assertions

| Signature | Returns | Description |
|---|---|---|
| `assert(cond: bool)` | `void` | Panic if condition is false |
| `assert_eq(a: i64, b: i64)` | `void` | Panic if values differ |

### String

| Signature | Returns | Description |
|---|---|---|
| `len(s: str) -> i64` | `i64` | Length of string |
| `int_to_str(n: i64) -> str` | `str` | Integer to string |
| `str_to_int(s: str) -> i64` | `i64` | String to integer |
| `str_contains(s: str, sub: str) -> bool` | `bool` | Check if substring exists |
| `str_find(s: str, sub: str) -> i64` | `i64` | Find substring index (-1 if not found) |
| `str_cmp(a: str, b: str) -> i64` | `i64` | Lexicographic comparison |
| `str_free(s: str) -> void` | `void` | Free allocated string |
| `substr(s: str, start: i64, end: i64) -> str` | `str` | Extract substring |
| `format(fmt: str, ...) -> str` | `str` | Format string with arguments |

### Hashing

| Signature | Returns | Description |
|---|---|---|
| `hash_i64(n: i64) -> i64` | `i64` | Hash an integer |
| `str_hash(s: str) -> i64` | `i64` | Hash a string |

### Math

| Signature | Returns | Description |
|---|---|---|
| `matmul(a: tensor, b: tensor) -> tensor` | `tensor` | Matrix multiplication |
| `sum(v: tensor) -> f64` | `f64` | Sum of tensor elements |
| `tensor_add(a: tensor, b: tensor) -> tensor` | `tensor` | Element-wise tensor addition |
| `blas_dot(a: tensor, b: tensor) -> f64` | `f64` | BLAS dot product |

### Misc

| Signature | Returns | Description |
|---|---|---|
| `arg_count() -> i64` | `i64` | Number of CLI arguments |
| `arg(n: i64) -> str` | `str` | Get CLI argument by index |

---

## 5. Standard Library API

### Option\<T\>

```aero
// `Option<T>` is auto-injected from the standard library:
let a: Option<i64> = Option::Some(5);
let b: Option<i64> = Option::None;
print(int_to_str(a.unwrap_or(-1)));
print("\n");
print(int_to_str(b.unwrap_or(-1)));
print("\n");
```

| Method | Signature | Description |
|---|---|---|
| `is_some` | `fn is_some(o: Option<T>) -> bool` | True if `Some` |
| `is_none` | `fn is_none(o: Option<T>) -> bool` | True if `None` |
| `unwrap_or` | `fn unwrap_or(o: Option<T>, def: T) -> T` | Unwrap or return default |

### Result\<T, E\>

```aero
// `Result<T, E>` is auto-injected from the standard library:
let ok: Result<i64, str> = Result::Ok(7);
let err: Result<i64, str> = Result::Err("boom");
print(format("is_ok: %d, is_err: %d\n", ok.is_ok(), err.is_err()));
print(format("unwrap: %lld\n", ok.unwrap_or(-1)));
```

| Method | Signature | Description |
|---|---|---|
| `is_ok` | `fn is_ok(r: Result<T, E>) -> bool` | True if `Ok` |
| `is_err` | `fn is_err(r: Result<T, E>) -> bool` | True if `Err` |
| `unwrap_or` | `fn unwrap_or(r: Result<T, E>, def: T) -> T` | Unwrap Ok or return default |
| `unwrap_err_or` | `fn unwrap_err_or(r: Result<T, E>, def: E) -> E` | Unwrap Err or return default |

### Traits

```aero
// The standard library pre-declares Copy, Clone, Drop, Iterator,
// IntoIterator, Add, Eq and Ord. User traits use the same syntax:

struct Circle {
    r: f64,
}

trait Area {
    fn area(x: Circle) -> f64;
}

impl Area for Circle {
    fn area(x: Circle) -> f64 {
        return x.r * x.r * 3.14;
    }
}
```

### Vec\<T\>

| Function | Signature | Description |
|---|---|---|
| `new` | `fn new() -> Vec<T>` | Create empty vector |
| `with_cap` | `fn with_cap(cap: i64) -> Vec<T>` | Create with pre-allocated capacity |
| `push` | `fn push(&mut self, val: T) -> void` | Append element |
| `pop` | `fn pop(&mut self) -> T` | Remove and return last element |
| `get` | `fn get(&self, i: i64) -> T` | Get element by index |
| `set` | `fn set(&mut self, i: i64, val: T) -> void` | Set element by index |
| `len` | `fn len(&self) -> i64` | Number of elements |
| `is_empty` | `fn is_empty(&self) -> bool` | True if empty |
| `free` | `fn free(&mut self) -> void` | Free backing memory |

Indexing: `v[i]` (get), `v[i] = x` (set).

### String

| Function | Signature | Description |
|---|---|---|
| `new` | `fn new() -> String` | Create empty string |
| `from` | `fn from(s: str) -> String` | Create from C string |
| `len` | `fn len(&self) -> i64` | Character count |
| `at` | `fn at(&self, i: i64) -> char` | Character at index |
| `push` | `fn push(&mut self, c: char) -> void` | Append character |
| `pop` | `fn pop(&mut self) -> char` | Remove last character |
| `free` | `fn free(&mut self) -> void` | Free backing memory |

Indexing: `s[i]` (get character).

### HashMap\<K, V\>

Constructed via `hash_map_new<V>()`. Keys are `i64`; values may be any type `V`.

| Function | Signature | Description |
|---|---|---|
| `insert` | `fn insert(&mut self, k: i64, v: V)` | Insert key-value pair |
| `get` | `fn get(&mut self, k: i64, def: V) -> V` | Get value by key, or `def` when absent |
| `contains` | `fn contains(&mut self, k: i64) -> bool` | Check key existence |
| `remove` | `fn remove(&mut self, k: i64) -> bool` | Remove key-value pair; returns whether present |
| `len` | `fn len(&self) -> i64` | Number of entries |
| `is_empty` | `fn is_empty(&self) -> bool` | True if empty |

### BTreeMap\<K, V\>

Constructed via `btree_map_new()`. Keys and values are `i64`.

| Function | Signature | Description |
|---|---|---|
| `insert` | `fn insert(&mut self, k: i64, v: i64)` | Insert key-value pair |
| `get` | `fn get(&mut self, k: i64, def: i64) -> i64` | Get value by key, or `def` when absent |
| `contains` | `fn contains(&mut self, k: i64) -> bool` | Check key existence |
| `remove` | `fn remove(&mut self, k: i64) -> bool` | Remove key-value pair; returns whether present |
| `len` | `fn len(&self) -> i64` | Number of entries |
| `is_empty` | `fn is_empty(&self) -> bool` | True if empty |
| `keys` | `fn keys(&mut self) -> Vec<i64>` | Return all keys in order |

### BTreeSet

Constructed via `btree_set_new()`. Elements are `i64`.

| Function | Signature | Description |
|---|---|---|
| `insert` | `fn insert(&mut self, v: i64)` | Insert value |
| `contains` | `fn contains(&mut self, v: i64) -> bool` | Check membership |
| `remove` | `fn remove(&mut self, v: i64) -> bool` | Remove value |
| `len` | `fn len(&self) -> i64` | Number of elements |
| `is_empty` | `fn is_empty(&self) -> bool` | True if empty |
| `to_vec` | `fn to_vec(&mut self) -> Vec<i64>` | Convert to sorted vector |

### LinkedList\<T\>

Constructed via `linked_list_new<T>()`.

| Function | Signature | Description |
|---|---|---|
| `len` | `fn len(&self) -> i64` | Number of elements |
| `is_empty` | `fn is_empty(&self) -> bool` | True if empty |
| `front` | `fn front(&mut self) -> Option<T>` | First element, or `None` when empty |
| `back` | `fn back(&mut self) -> Option<T>` | Last element, or `None` when empty |
| `push_front` | `fn push_front(&mut self, v: T)` | Prepend |
| `push_back` | `fn push_back(&mut self, v: T)` | Append |
| `pop_front` | `fn pop_front(&mut self) -> Option<T>` | Remove and return first |
| `pop_back` | `fn pop_back(&mut self) -> Option<T>` | Remove and return last |
| `get` | `fn get(&mut self, i: i64, def: T) -> T` | Element at index, or `def` when out of range |
| `set` | `fn set(&mut self, i: i64, v: T) -> bool` | Set at index; returns whether in range |
| `remove` | `fn remove(&mut self, i: i64) -> bool` | Remove at index; returns whether in range |
| `reverse` | `fn reverse(&mut self)` | Reverse in place |
| `clear` | `fn clear(&mut self)` | Remove all elements |

### Sorting

| Function | Signature | Description |
|---|---|---|
| `sort` | `fn sort(v: &mut Vec<i64>)` | Sort in place |
| `binary_search` | `fn binary_search(v: &mut Vec<i64>, target: i64) -> i64` | Binary search, returns index or -1 |
| `reverse` | `fn reverse(v: &mut Vec<i64>)` | Reverse in place |

### Functional

```aero
// `FnPred`, `FnTrans` and `FnRed` are compiler-known function pointer types:
//   FnPred  = fn(i64) -> bool
//   FnTrans = fn(i64) -> i64
//   FnRed   = fn(i64, i64) -> i64

fn is_even(x: i64) -> bool {
    if (x % 2 == 0) {
        return true;
    }
    return false;
}

fn double(x: i64) -> i64 {
    return x * 2;
}

fn add_all(a: i64, b: i64) -> i64 {
    return a + b;
}

let v = Vec::new();
let i = 0;
while (i < 6) {
    v.push(i);
    i = i + 1;
}

let evens = _filter_impl(v, is_even);
let doubled = _map_impl(evens, double);
let total = _reduce_impl(doubled, add_all, 0);
print(format("total: %lld\n", total));
```

### Path Operations

| Function | Signature | Description |
|---|---|---|
| `path_join` | `fn path_join(a: str, b: str) -> String` | Join path segments |
| `path_basename` | `fn path_basename(p: str) -> String` | Last component |
| `path_dirname` | `fn path_dirname(p: str) -> String` | Parent directory |
| `path_extension` | `fn path_extension(p: str) -> String` | File extension |

### JSON Helpers

| Function | Signature | Description |
|---|---|---|
| `json_escape` | `fn json_escape(s: str) -> String` | Escape string for JSON |
| `json_string` | `fn json_string(s: str) -> String` | Create JSON string value |
| `json_number_i64` | `fn json_number_i64(n: i64) -> String` | Create JSON i64 number |
| `json_number_f64` | `fn json_number_f64(n: f64) -> String` | Create JSON f64 number |
| `json_bool` | `fn json_bool(b: bool) -> String` | Create JSON boolean |
| `json_null` | `fn json_null() -> String` | Create JSON null |
| `json_join` | `fn json_join(a: String, sep: str, b: String) -> String` | Join two JSON fragments with a separator |
| `json_parse_i64` | `fn json_parse_i64(s: str) -> i64` | Parse JSON integer |
| `json_parse_bool` | `fn json_parse_bool(s: str) -> bool` | Parse JSON boolean |
| `json_unescape` | `fn json_unescape(s: str) -> String` | Unescape JSON string |
| `json_find_key` | `fn json_find_key(s: str, key: str) -> i64` | Locate a key in a JSON object; index after `"key":` or -1 |

### Automatic Differentiation (Grad)

| Function | Signature | Description |
|---|---|---|
| `grad_new` | `fn grad_new() -> Grad` | Create an empty tape |
| `g_leaf` | `fn g_leaf(g: &mut Grad, v: f64) -> i64` | Record a leaf node (variable or constant) |
| `g_add` | `fn g_add(g: &mut Grad, a: i64, b: i64) -> i64` | Add two nodes |
| `g_mul` | `fn g_mul(g: &mut Grad, a: i64, b: i64) -> i64` | Multiply two nodes |
| `g_neg` | `fn g_neg(g: &mut Grad, a: i64) -> i64` | Negate |
| `g_inv` | `fn g_inv(g: &mut Grad, a: i64) -> i64` | Invert (1/x) |
| `g_sub` | `fn g_sub(g: &mut Grad, a: i64, b: i64) -> i64` | Subtract |
| `g_div` | `fn g_div(g: &mut Grad, a: i64, b: i64) -> i64` | Divide |
| `g_backward` | `fn g_backward(g: &mut Grad, root: i64)` | Backpropagate gradients |
| `g_grad` | `fn g_grad(g: &mut Grad, node: i64) -> f64` | Retrieve computed gradient |
| `g_val` | `fn g_val(g: &mut Grad, node: i64) -> f64` | Retrieve node value |

---

## 6. Official Core Crate API

### aero-json

```aero
struct JsonValue {
    data: str,
}

fn parse_json(s: str) -> JsonValue {
    return JsonValue { data: s };
}

fn json_get_int(v: &JsonValue, key: str) -> i64 {
    return 0;
}

fn json_get_str(v: &JsonValue, key: str) -> str {
    return "";
}

fn json_number(v: &JsonValue) -> f64 {
    return 0.0;
}
```

### aero-csv

```aero
fn csv_count_lines(path: str) -> i64 {
    return 0;
}

fn csv_get_record(path: str, n: i64) -> str {
    return "";
}

fn csv_count_fields(record: str) -> i64 {
    return 0;
}

fn csv_get_field(record: str, n: i64) -> str {
    return "";
}
```

### aero-logger

```aero
enum LogLevel {
    Debug,
    Info,
    Warn,
    Error,
}

fn log_error(msg: str) {}
fn log_warn(msg: str) {}
fn log_info(msg: str) {}
fn log_debug(msg: str) {}
fn log(msg: str, level: LogLevel) {}
fn log_separator() {}
fn log_kv(key: str, val: str) {}
fn log_section(title: str) {}
```

### aero-test

```aero
fn assert_true(cond: bool) {}
fn assert_eq_i64(a: i64, b: i64) {}
fn assert_eq_i32(a: i32, b: i32) {}
fn assert_eq_str(a: str, b: str) {}
fn test_group(name: str) {}
fn test_summary() {}
```

### aero-collections

```aero
// Stack
struct Stack<T> {
    data: Vec<T>,
}

struct Queue<T> {
    data: Vec<T>,
}

fn stack_push<T>(s: &mut Stack<T>, v: T) {}
fn stack_pop<T>(s: &mut Stack<T>) -> T {
    return s.data.pop();
}
fn stack_peek<T>(s: &mut Stack<T>) -> T {
    return s.data.get(s.data.len() - 1);
}
fn stack_is_empty<T>(s: &mut Stack<T>) -> bool {
    return s.data.is_empty();
}
fn stack_len<T>(s: &mut Stack<T>) -> i64 {
    return s.data.len();
}

// Queue
fn queue_enqueue<T>(q: &mut Queue<T>, v: T) {}
fn queue_dequeue<T>(q: &mut Queue<T>) -> T {
    return q.data.pop();
}
fn queue_is_empty<T>(q: &mut Queue<T>) -> bool {
    return q.data.is_empty();
}
fn queue_len<T>(q: &mut Queue<T>) -> i64 {
    return q.data.len();
}
```

### aero-time

```aero
fn format_timestamp() -> str {
    return "";
}

fn format_duration(secs: f64) -> str {
    return "";
}

fn format_bytes(bytes: i64) -> str {
    return "";
}

fn timestamp() -> i64 {
    return 0;
}
```

### aero-io

```aero
// `read_file(path)` and `file_exists(path)` are compiler builtins.
fn file_extension(path: str) -> str {
    return "";
}

fn basename(path: str) -> str {
    return "";
}
```

### aero-toml

```aero
enum TomlValue {
    Str,
    Num,
    Bool,
    List,
}

fn toml_parse_line(line: str) -> TomlValue {
    return TomlValue::Str;
}

fn toml_is_section(line: str) -> bool {
    return false;
}

fn toml_is_comment(line: str) -> bool {
    return false;
}
```

### aero-http

```aero
struct HttpResponse {
    data: str,
    ok: bool,
}

fn http_get(url: str) -> HttpResponse {
    return HttpResponse { data: "", ok: false };
}

fn http_post(url: str, data: str) -> HttpResponse {
    return HttpResponse { data: "", ok: false };
}

fn http_ok(res: &HttpResponse) -> bool {
    return res.ok;
}

fn http_body(res: &HttpResponse) -> str {
    return res.data;
}
```

### aero-crypto

```aero
fn hex_encode(data: str) -> str {
    return "";
}

fn sha256(data: str) -> str {
    return "";
}

fn sha256_hash(data: str) -> str {
    return "";
}

fn xor_obfuscate(data: str, key: str) -> str {
    return "";
}

fn constant_time_eq(a: str, b: str) -> bool {
    return false;
}
```

### aero-regex

```aero
struct RegexMatch {
    data: str,
}

struct Regex {}

fn regex_compile(pattern: str) -> Regex {
    return Regex {};
}

fn regex_is_match(re: &Regex, s: str) -> bool {
    return false;
}

fn regex_find(re: &Regex, s: str) -> RegexMatch {
    return RegexMatch { data: "" };
}

fn regex_free(re: &mut Regex) {}

fn regex_test(re: &Regex, s: str) -> bool {
    return false;
}

fn regex_group(m: &RegexMatch, n: i64) -> str {
    return "";
}
```

### aero-net

```aero
struct TcpSocket {}

fn net_init() {}
fn net_cleanup() {}
fn tcp_create() -> TcpSocket {
    return TcpSocket {};
}
fn tcp_connect(s: &TcpSocket, addr: str, port: i32) {}
fn tcp_send(s: &TcpSocket, data: str) -> i64 {
    return 0;
}
fn tcp_recv(s: &TcpSocket, buffer: &mut str, len: i64) -> i64 {
    return 0;
}
fn tcp_close(s: &mut TcpSocket) {}
```

---

## 7. Compiler CLI Reference

| Command | Description |
|---|---|
| Command | Description |
|---|---|
| `aero run <file.aero \| package dir>` | Compile and run (`-O0..-O3` to pick an opt level) |
| `aero build [file.aero \| dir]` | Compile to a standalone exe (AOT; `-O0..-O3`) |
| `aero build <file> --target <triple>` | Cross-compile (e.g. `x86_64-unknown-linux-gnu`) |
| `aero new <name>` | Scaffold a new project |
| `aero test [file.aero]` | Run tests |
| `aero fmt <file.aero>` | Format source (`--check` only checks; `--width`/`--indent`) |
| `aero bench <file.aero>` | Run benchmarks |
| `aero clippy <file.aero>` | Static analysis (100+ rules) |
| `aero cov <file.aero>` | Coverage report |
| `aero --lsp \| aero lsp` | Language server (JSON-RPC 2.0) |
| `aero lock [dir]` | Resolve deps, write Aero.lock |
| `aero publish <dir>` | Publish a library package to the registry |
| `aero ls` | List packages/versions in the registry |
| `aero registry` | Print the registry location |