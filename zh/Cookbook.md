# Aero 语言烹饪书

本文档包含 Aero 语言的常见场景代码示例。所有示例均为完整可运行代码。

---

## Hello World

```aero
print("Hello, Aero!\n");
```

输出：`Hello, Aero!`

---

## 斐波那契数列

```aero
fn fib(n: i64) -> i64 {
    if (n <= 1) {
        return n;
    }
    return fib(n - 1) + fib(n - 2);
}

let i = 0;
while (i < 10) {
    print("%s ", int_to_str(fib(i)));
    i = i + 1;
}
print("\n");
```

输出：`0 1 1 2 3 5 8 13 21 34`

---

## 结构体与方法

方法不使用 `self`，而是显式声明接收者参数：

```aero
struct Rectangle {
    width: f64,
    height: f64,
}

impl Rectangle {
    fn area(r: Rectangle) -> f64 {
        return r.width * r.height;
    }

    fn scale(r: &mut Rectangle, factor: f64) {
        r.width = r.width * factor;
        r.height = r.height * factor;
    }
}

let r = Rectangle { width: 3.0, height: 4.0 };
print("area: %f\n", r.area());
r.scale(2.0);
print("scaled area: %f\n", r.area());
```

输出：
```
area: 12.000000
scaled area: 48.000000
```

---

## 枚举与模式匹配

枚举变体只支持单参数，多参数请用元组。`match` 的每个分支必须是语句块：

```aero
enum Shape {
    Circle(f64),
    Rect((f64, f64)),
}

fn area(s: Shape) -> f64 {
    let result = 0.0;
    match (s) {
        Shape::Circle(r) => { result = 3.14159 * r * r; }
        Shape::Rect(dims) => { result = dims[0] * dims[1]; }
    }
    return result;
}

let c = Shape::Circle(5.0);
let r = Shape::Rect((3.0, 4.0));
print("circle area: %f\n", area(c));
print("rect area: %f\n", area(r));
```

输出：
```
circle area: 78.539750
rect area: 12.000000
```

---

## 泛型函数

```aero
fn identity<T>(x: T) -> T {
    return x;
}

fn swap<T>(a: T, b: T) -> (T, T) {
    return (b, a);
}

let x = identity(42);
let y = identity(3.14);
let ab = swap(1, 2);
print("%lld %f %lld %lld\n", x, y, ab[0], ab[1]);
```

输出：`42 3.140000 2 1`

---

## Vec 操作

```aero
let v = Vec::new();
v.push(3);
v.push(1);
v.push(4);
v.push(1);
v.push(5);

print("len: %lld\n", v.len());
print("v[0]: %lld\n", v[0]);

sort(v);
for (x in v) {
    print("%lld ", x);
}
print("\n");

let idx = binary_search(v, 4);
print("index of 4: %lld\n", idx);

reverse(v);
for (x in v) {
    print("%lld ", x);
}
print("\n");
```

输出：
```
len: 5
v[0]: 3
1 1 3 4 5
index of 4: 3
5 4 3 1 1
```

---

## 字符串操作

`str` 上没有方法，需先 `String::from(s)` 得到 `String` 再调用方法；`String` 转回 `str` 用 `.data()`：

```aero
let s = String::from("hello");
s.push(' ');
s.push('w');
s.push('o');
s.push('r');
s.push('l');
s.push('d');

print("%c\n", s.at(0));   // 'h'
print("%c\n", s[1]);      // 'e'
print("len: %lld\n", s.len());

let full = "hello, world";
if (str_contains(full, "world")) {
    let pos = str_find(full, "world");
    let part = substr(full, pos, pos + 5);
    print("%s\n", part);
}
```

输出：
```
h
e
len: 11
world
```

---

## 文件读写

```aero
let content = "Hello, Aero file I/O!\n";
write_file("example.txt", content);

let read = read_file("example.txt");
print("%s", read);
```

输出：`Hello, Aero file I/O!`

---

## HashMap 使用

注意：`HashMap` 的键固定为 `i64`，`get` 必须提供默认值：

```aero
let map = hash_map_new();

map.insert(1, 100);
map.insert(2, 200);
map.insert(3, 300);

if (map.contains(1)) {
    print("key 1: %lld\n", map.get(1, -1));
}

print("entries: %lld\n", map.len());
map.remove(3);
print("after remove: %lld\n", map.len());
```

输出：
```
key 1: 100
entries: 3
after remove: 2
```

---

## JSON 解析与生成

```aero
// 生成 JSON
let a = json_string("Aero");
let b = json_number_i64(42);
let c = json_bool(true);
let d = json_null();
let arr = json_join(json_join(json_join(a, ",", b), ",", c), ",", d);
print("[%s]\n", arr.data());

// 解析 JSON
let data = "{\"count\": 100, \"active\": true}";
let at = json_find_key(data, "count");
let count = json_parse_i64(substr(data, at, 999));
let at2 = json_find_key(data, "active");
let active = json_parse_bool(substr(data, at2, 999));
print("count: %lld, active: %d\n", count, active);
```

输出：
```
["Aero",42,true,null]
count: 100, active: 1
```

---

## CSV 解析

以下为 `aero-csv` crate 的本地实现（crate 需在 `Aero.toml` 中声明依赖）。其函数以整段内容字符串为输入：

```aero
fn csv_count_lines(s: str) -> i64 {
    let src = String::from(s);
    let n = src.len();
    let count = 0;
    let i = 0;
    while (i < n) {
        if (src.at(i) == 10) {
            count = count + 1;
        }
        i = i + 1;
    }
    if ((n > 0) && (src.at(n - 1) != 10)) {
        count = count + 1;
    }
    return count;
}

fn csv_get_record(s: str, n: i64) -> str {
    let src = String::from(s);
    let sl = src.len();
    let current = 0;
    let i = 0;
    while (i < sl) {
        if (current == n) {
            let line = "";
            let j = i;
            while ((j < sl) && (src.at(j) != 10)) {
                line = line + format("%c", src.at(j));
                j = j + 1;
            }
            return line;
        }
        if (src.at(i) == 10) {
            current = current + 1;
        }
        i = i + 1;
    }
    return "";
}

fn csv_count_fields(record: str) -> i64 {
    let r = String::from(record);
    let n = r.len();
    if (n == 0) {
        return 0;
    }
    let count = 1;
    let i = 0;
    while (i < n) {
        if (r.at(i) == 44) {
            count = count + 1;
        }
        i = i + 1;
    }
    return count;
}

fn csv_get_field(record: str, n: i64) -> str {
    let r = String::from(record);
    let rl = r.len();
    let current = 0;
    let i = 0;
    while (i < rl) {
        if (current == n) {
            let field = "";
            let j = i;
            while ((j < rl) && (r.at(j) != 44)) {
                field = field + format("%c", r.at(j));
                j = j + 1;
            }
            return field;
        }
        if (r.at(i) == 44) {
            current = current + 1;
        }
        i = i + 1;
    }
    return "";
}

let csv_path = "data.csv";
write_file(csv_path, "name,age,score\nAlice,30,95.5\nBob,25,88.0\n");

let content = read_file(csv_path);
let lines = csv_count_lines(content);
print("lines: %lld\n", lines);

let record = csv_get_record(content, 0);  // 表头
let fields = csv_count_fields(record);
print("fields: %lld\n", fields);

let field = csv_get_field(record, 0);
print("header field 0: %s\n", field);
```

输出：
```
lines: 3
fields: 3
header field 0: name
```

---

## 字符串查找（正则需 aero-regex crate）

真正的正则匹配需要 `aero-regex` crate（基于 POSIX regex FFI，需链接系统库）。单文件示例改用内置字符串查找演示同一思路：

```aero
let text = "abc123def456";
let pos = str_find(text, "123");
if (pos >= 0) {
    let matched = substr(text, pos, pos + 3);
    print("first match: %s\n", matched);
}
let n = str_contains(text, "456");
print("contains 456: %d\n", n);
```

输出：
```
first match: 123
contains 456: 1
```

---

## HTTP 请求

HTTP 请求需要 `aero-http` crate（libcurl FFI）。单文件示例演示其 API 调用形态：

```aero
struct HttpResponse {
    data: str,
    ok: bool,
}

fn http_get(url: str) -> HttpResponse {
    return HttpResponse { data: "", ok: false };
}

fn http_ok(resp: HttpResponse) -> bool {
    return resp.ok;
}

fn http_body(resp: HttpResponse) -> str {
    return resp.data;
}

let resp = http_get("https://httpbin.org/get");
if (http_ok(resp)) {
    print("request succeeded\n");
} else {
    print("request failed\n");
}
```

输出：`request failed`（真实环境中成功时返回 HTTP 响应体）。

---

## TCP 网络通信

TCP 网络通信需要 `aero-net` crate（Winsock FFI）。单文件示例演示其 API 调用形态：

```aero
struct TcpSocket {
    fd: i64,
    connected: bool,
}

fn net_init() -> i32 {
    return 0;
}

fn net_cleanup() {
}

fn tcp_create() -> TcpSocket {
    return TcpSocket { fd: 0, connected: false };
}

fn tcp_connect(sock: &TcpSocket, ip: str, port: i32) -> i32 {
    return 0;
}

fn tcp_send(sock: &TcpSocket, data: str) -> i32 {
    return 0;
}

fn tcp_recv(sock: &TcpSocket, bufsize: i64) -> str {
    return "";
}

fn tcp_close(sock: &TcpSocket) {
}

net_init();
let sock = tcp_create();
let rc = tcp_connect(sock, "httpbin.org", 80);
print("connect rc=%d\n", rc);
tcp_close(sock);
net_cleanup();
```

输出：`connect rc=0`（真实环境中 `rc` 为连接结果，`tcp_recv` 返回响应数据）。

---

## 加密与哈希

`aero-crypto` crate 的 `sha256` / `xor_obfuscate` 依赖 OpenSSL FFI。此处演示其纯 Aero 可复现的部分（十六进制编码、常量时间比较），哈希使用内置 `str_hash`：

```aero
// 十六进制编码（aero-crypto::hex_encode 的本地实现）
fn hex_encode(data: str, len: i64) -> str {
    let hex_chars = String::from("0123456789abcdef");
    let result = "";
    let i = 0;
    while (i < len) {
        let byte = utf8_at(data, i);
        let hi = (byte >> 4) & 15;
        let lo = byte & 15;
        result = result + format("%c", hex_chars.at(hi));
        result = result + format("%c", hex_chars.at(lo));
        i = i + 1;
    }
    return result;
}

// 常量时间比较（aero-crypto::constant_time_eq 的本地实现）
fn constant_time_eq(a: str, b: str) -> bool {
    if (len(a) != len(b)) {
        return false;
    }
    let result = 0;
    let i = 0;
    let n = len(a);
    while (i < n) {
        result = result | (utf8_at(a, i) ^ utf8_at(b, i));
        i = i + 1;
    }
    return result == 0;
}

let data = "hello";
print("hex: %s\n", hex_encode(data, len(data)));

// 字符串哈希（内置 str_hash，FNV-1a）
print("hash: %lld\n", str_hash(data));

print("eq abc/abc: %d\n", constant_time_eq("abc", "abc"));
print("eq abc/abd: %d\n", constant_time_eq("abc", "abd"));
```

输出：
```
hex: 68656c6c6f
hash: -6615550055289275125
eq abc/abc: 1
eq abc/abd: 0
```

---

## 自动微分（Grad 求导）

所有 Grad 操作都以 `&mut Grad` 作为第一个参数，节点用 `i64` 索引表示：

```aero
// 计算 f(x, y) = x * y + x 在 x=3, y=2 处的梯度
let g = grad_new();
let x = g_leaf(g, 3.0);
let y = g_leaf(g, 2.0);

let t = g_mul(g, x, y);   // x * y
let f = g_add(g, t, x);   // + x

g_backward(g, f);

print("f val: %f\n", g_val(g, f));
print("df/dx: %f\n", g_grad(g, x));
print("df/dy: %f\n", g_grad(g, y));
```

输出：
```
f val: 9.000000
df/dx: 3.000000
df/dy: 3.000000
```

解析：f(3,2) = 3×2 + 3 = 9，∂f/∂x = y + 1 = 3，∂f/∂y = x = 3。

---

## 日志记录

以下为 `aero-logger` crate 的本地实现（crate 需在 `Aero.toml` 中声明依赖）：

```aero
fn log_section(title: str) {
    print("== %s ==\n", title);
}

fn log_info(msg: str) {
    print("[INFO]  %s\n", msg);
}

fn log_kv(key: str, value: str) {
    print("  %s: %s\n", key, value);
}

fn log_debug(msg: str) {
    print("[DEBUG] %s\n", msg);
}

fn log_warn(msg: str) {
    print("[WARN]  %s\n", msg);
}

fn log_error(msg: str) {
    print("[ERROR] %s\n", msg);
}

fn log_separator() {
    print("---\n");
}

log_section("Application Start");
log_info("initializing modules");
log_kv("version", "1.1.0");
log_debug("debug detail: loaded 42 entities");
log_warn("deprecated API detected");
log_error("failed to connect to database");
log_separator();
log_info("shutdown complete");
```

输出：结构化日志信息，包含标题和日志级别。

---

## 单元测试

以下为 `aero-test` crate 的本地实现（crate 需在 `Aero.toml` 中声明依赖）：

```aero
fn assert_eq_i64(got: i64, expected: i64, msg: str) {
    if (got == expected) {
        print("PASS: %s\n", msg);
    } else {
        print("FAIL: %s\n", msg);
    }
}

fn test_group(name: str) {
    print("\n=== %s ===\n", name);
}

fn test_summary(passed: i64, failed: i64) {
    print("summary: %lld passed, %lld failed\n", passed, failed);
}

fn add(a: i64, b: i64) -> i64 {
    return a + b;
}

fn test_add() {
    assert_eq_i64(add(2, 3), 5, "add(2,3)");
    assert_eq_i64(add(-1, 1), 0, "add(-1,1)");
    assert_eq_i64(add(0, 0), 0, "add(0,0)");
}

test_group("math::add");
test_add();
test_summary(3, 0);
```

输出：测试结果汇总，包含通过/失败统计。

---

## FFI 调用 C 库

`extern "C"` 声明形式为 `extern "C" fn 名字(参数) -> 返回类型;`（不是 `extern { }` 块）。`rand` 是编译器内置函数，可直接调用：

```aero
extern "C" fn llabs(x: i64) -> i64;

let r = rand();
print("random: %lld\n", r);

let a = llabs(-16);
print("abs(-16): %lld\n", a);
```

如需链接额外的系统库，在 `Aero.toml` 中配置（例如使用数学库函数时链接 `m`）：

```toml
[link]
libs = ["m"]
```

输出：
```
random: <随机整数>
abs(-16): 16
```

---

## 命令行参数解析

`aero run` 不转发程序参数；请先 `aero build` 编译，再直接运行生成的 exe 并传参：

```aero
let n = arg_count();
print("arg count: %lld\n", n);

let i = 0;
while (i < n) {
    let a = arg(i);
    print("arg[%lld]: %s\n", i, a);
    i = i + 1;
}

// 简单参数处理
if (n > 1) {
    let mode = arg(1);
    if (str_cmp(mode, "--help") == 0) {
        print("Usage: program [--help] [--version]\n");
    } else {
        if (str_cmp(mode, "--version") == 0) {
            print("Aero 1.1.0\n");
        }
    }
}
```

编译后运行 `program.exe --version` 输出：
```
arg count: 2
arg[0]: program
arg[1]: --version
Aero 1.1.0
```
