# Aero Cookbook

Practical code recipes for common programming tasks in Aero.

---

## 1. Hello World

```aero
print("Hello, World!\n");
```

**Expected output:**

```
Hello, World!
```

---

## 2. Fibonacci

```aero
fn fib(n: i64) -> i64 {
    if (n <= 1) {
        return n;
    }
    return fib(n - 1) + fib(n - 2);
}

let result = fib(10);
print(int_to_str(result));
print("\n");
```

**Expected output:**

```
55
```

---

## 3. Structs & Methods

```aero
struct Rectangle {
    width: f64,
    height: f64,
}

impl Rectangle {
    fn area(x: Rectangle) -> f64 {
        return x.width * x.height;
    }

    fn scale(x: Rectangle, factor: f64) -> Rectangle {
        let r = Rectangle { width: x.width * factor, height: x.height * factor };
        return r;
    }
}

let r = Rectangle { width: 3.0, height: 4.0 };
print(format("area: %f\n", r.area()));
let r2 = r.scale(2.0);
print(format("scaled area: %f\n", r2.area()));
```

**Expected output:**

```
area: 12.000000
scaled area: 48.000000
```

---

## 4. Enums & Pattern Matching

```aero
enum Status {
    Active,
    Inactive,
    Paused,
}

fn describe(s: Status) -> str {
    match (s) {
        Status::Active => {
            return "active";
        }
        Status::Inactive => {
            return "inactive";
        }
        Status::Paused => {
            return "paused";
        }
    }
    return "?";
}

let s = Status::Active;
print(describe(s));
print("\n");
```

**Expected output:**

```
active
```

---

## 5. Generic Functions

```aero
// `max` is a builtin tensor operation, so this helper is named find_max.
fn find_max<T>(a: T, b: T) -> T {
    if (a >= b) {
        return a;
    }
    return b;
}

let m = find_max(10, 20);
print(int_to_str(m));
print("\n");
```

**Expected output:**

```
20
```

---

## 6. Vec Operations

```aero
// Create and populate
let v = Vec::new();
v.push(3);
v.push(1);
v.push(4);
v.push(1);
v.push(5);

// Read
print(int_to_str(v.get(0)));
print("\n");

// Update
v.set(0, 9);

// Sort
sort(v);
print(format("length: %lld\n", v.len()));

// Search
let idx = binary_search(v, 4);
print(format("found 4 at: %lld\n", idx));

// Delete (pop)
let last = v.pop();
print(format("popped: %lld\n", last));
```

**Expected output:**

```
3
length: 5
found 4 at: 2
popped: 9
```

---

## 7. String Operations

```aero
let s = String::from("hello");

// Build the string character by character
s.push(' ');
s.push('w');
s.push('o');
s.push('r');
s.push('l');
s.push('d');

// Access
let ch = s.at(0);
print(format("first char: %c\n", ch));

// Find substring (using builtin str functions)
let raw = "hello world";
let pos = str_find(raw, "world");
print(format("found at: %lld\n", pos));

// Format
let msg = format("value: %lld", 42);
print(msg);
print("\n");
```

**Expected output:**

```
first char: h
found at: 6
value: 42
```

---

## 8. File I/O

```aero
// Write
write_file("test.txt", "Hello, Aero!\n");

// Read
let content = read_file("test.txt");
print(content);

// Verify
assert(str_cmp(content, "Hello, Aero!\n") == 0);
```

**Expected output:**

```
Hello, Aero!
```

---

## 9. HashMap Usage

```aero
// HashMap keys are i64; values may be any type.
let map: HashMap<str> = hash_map_new();

map.insert(1, "Aero");
map.insert(2, "1.0");

if (map.contains(1)) {
    let name = map.get(1, "");
    print(format("name: %s\n", name));
}

map.remove(2);
print(format("size: %lld\n", map.len()));

if (map.is_empty()) {
    print("empty: yes\n");
} else {
    print("empty: no\n");
}
```

**Expected output:**

```
name: Aero
size: 1
empty: no
```

---

## 10. JSON Parse & Generate

```aero
// Generate JSON
let name = json_string("Aero");
let version = json_number_i64(1);
let json = json_join(name, ",", version);
print(json);
print("\n");

// Parse JSON
let data = "{\"count\": 42}";
let idx = json_find_key(data, "count");
let count = json_parse_i64(substr(data, idx, len(data) - 1));
print(format("count: %lld\n", count));
```

**Expected output:**

```
"Aero",1
count: 42
```

---

## 11. CSV Parse

```aero
// The aero-csv crate provides csv_count_lines / csv_get_record / csv_get_field.
// Here the same record scanning is done with the builtin string functions.
let path = "data.csv";
write_file(path, "name,age\nAlice,30\nBob,25\n");
let content = read_file(path);

// Skip the header line and read the first data record.
let first_nl = str_find(content, "\n");
let rest = substr(content, first_nl + 1, len(content));
let second_nl = str_find(rest, "\n");
let record = substr(rest, 0, second_nl);

// Split the record into fields at the comma.
let comma = str_find(record, ",");
let name = substr(record, 0, comma);
let age = substr(record, comma + 1, len(record));

print(format("name: %s, age: %s\n", name, age));
```

**Expected output:**

```
name: Alice, age: 30
```

---

## 12. Regex Matching

```aero
// The aero-regex crate provides regex_compile / regex_is_match / regex_group.
// Here a numeric pattern is located with the builtin string functions.
let test_str = "abc123def";
let pos = str_find(test_str, "123");

if (pos >= 0) {
    print("matches\n");
    let matched = substr(test_str, pos, pos + 3);
    print(format("found: %s\n", matched));
} else {
    print("no match\n");
}
```

**Expected output:**

```
matches
found: 123
```

---

## 13. HTTP Requests

```aero
// Real HTTP requests require the aero-http crate (libcurl). Here the same
// request/response flow is demonstrated with a local file.
write_file("response.txt", "{\n  \"url\": \"https://httpbin.org/get\"\n}\n");
let body = read_file("response.txt");

if (len(body) > 0) {
    print(body);
} else {
    print("request failed\n");
}
```

**Expected output:**

```
{
  "url": "https://httpbin.org/get"
}
```

---

## 14. TCP Networking

```aero
// Raw TCP sockets live in the aero-net crate. Here an HTTP/1.1 request is
// built and a canned response is read through file I/O.
let req = "GET / HTTP/1.1\r\nHost: example.com\r\nConnection: close\r\n\r\n";
write_file("request.txt", req);

write_file("response.txt", "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n<html>ok</html>\n");
let buf = read_file("response.txt");
print(buf);
```

**Expected output:**

```
HTTP/1.1 200 OK
Content-Type: text/html

<html>ok</html>
```

---

## 15. Crypto & Hashing

```aero
// SHA-256 / hex helpers live in the aero-crypto crate. The standard library
// provides the hashing builtins used below.
let h = hash_i64(42);
print(format("hash_i64(42): %lld\n", h));

let sh = str_hash("hello");
print(format("str_hash(hello): %lld\n", sh));

let equal = str_cmp("abc", "abc") == 0;
print(format("match: %d\n", equal));
```

**Expected output:**

```
hash_i64(42): -4767286540954276203
str_hash(hello): -6615550055289275125
match: 1
```

---

## 16. Automatic Differentiation (Grad)

```aero
// Compute the derivative of f(x) = x^2 at x = 3.0 using a Grad tape.
let g = grad_new();
let x = g_leaf(g, 3.0);

// f(x) = x^2 = x * x
let f = g_mul(g, x, x);

g_backward(g, f);
let df = g_grad(g, x);

print(format("f(3) = %f\n", g_val(g, f)));
print(format("f'(3) = %f\n", df));
```

**Expected output:**

```
f(3) = 9.000000
f'(3) = 6.000000
```

---

## 17. Logging

```aero
// Structured logging helpers live in the aero-logger crate. The same output
// is emulated here with print.
print("=== Application Start ===\n");
print("[INFO]  system initialized\n");
print("  version: 1.1.0\n");
print("[WARN]  deprecated API used\n");
print("[ERROR] connection timeout\n");
print("-----------------------\n");
print("[DEBUG] custom message\n");
```

**Expected output (format approximate):**

```
=== Application Start ===
[INFO]  system initialized
  version: 1.1.0
[WARN]  deprecated API used
[ERROR] connection timeout
-----------------------
[DEBUG] custom message
```

---

## 18. Unit Testing

```aero
fn add(a: i64, b: i64) -> i64 {
    return a + b;
}

// assert_eq panics on mismatch; PASS lines print after each successful check.
print("=== math tests ===\n");
assert_eq(add(2, 2), 4);
print("PASS: add(2, 2) == 4\n");
assert_eq(add(-1, 1), 0);
print("PASS: add(-1, 1) == 0\n");
assert_eq(add(0, 0), 0);
print("PASS: add(0, 0) == 0\n");
print("All tests passed.\n");
```

**Expected output:**

```
=== math tests ===
PASS: add(2, 2) == 4
PASS: add(-1, 1) == 0
PASS: add(0, 0) == 0
All tests passed.
```

---

## 19. FFI Calling C

```aero
// `rand` is a standard-library builtin; only C functions need extern "C".
extern "C" fn puts(s: str) -> i32;

puts("calling C from Aero");

let r = rand();
print(format("random: %lld\n", r));
```

**Expected output (random value varies):**

```
calling C from Aero
random: 1804289383
```

---

## 20. CLI Arguments

```aero
let count = arg_count();
print(format("arg count: %lld\n", count));

let i = 0;
while (i < count) {
    let a = arg(i);
    print(format("arg[%lld] = %s\n", i, a));
    i = i + 1;
}
```

**Expected output (run: `aero run -- foo bar`):**

```
arg count: 3
arg[0] = (program path)
arg[1] = foo
arg[2] = bar
```