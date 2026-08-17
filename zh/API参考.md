# Aero 语言 API 参考手册

## 语言关键字表

| 关键字 | 类别 | 用途 |
|--------|------|------|
| `let` | 声明 | 变量绑定，默认可变 |
| `mut` | 修饰符 | 保留字（可变性默认，`let mut` 不可用） |
| `fn` | 声明 | 函数定义 |
| `if` / `else` | 控制流 | 条件分支 |
| `while` | 控制流 | 条件循环 |
| `loop` | 控制流 | 无限循环 |
| `for` / `in` | 控制流 | 迭代循环 |
| `return` | 控制流 | 函数返回 |
| `break` | 控制流 | 跳出循环 |
| `continue` | 控制流 | 继续下一轮循环 |
| `match` / `as` | 控制流 | 模式匹配 |
| `struct` | 声明 | 结构体定义 |
| `union` | 声明 | 联合体定义 |
| `enum` | 声明 | 枚举定义 |
| `trait` | 声明 | 特征定义 |
| `impl` | 声明 | 实现定义 |
| `type` | 声明 | 类型别名（保留，暂未支持 `type X = ...`） |
| `dyn` | 类型 | 动态分发 |
| `const` | 声明 | 常量定义 |
| `mod` | 模块 | 模块声明（内联 `mod { ... }`） |
| `use` | 模块 | 路径引入 |
| `pub` | 模块 | 可见性修饰 |
| `crate` | 模块 | 当前 crate 根 |
| `extern` | FFI | 外部函数接口 |
| `arena` | 内存 | 竞技场分配器 |
| `tensor` | AI | 张量类型 |
| `gpu` | AI | GPU 操作 |

## 类型系统

### 基础类型

| 类型 | 描述 | 字面量示例 |
|------|------|-----------|
| `i32` | 32位有符号整数 | — |
| `i64` | 64位有符号整数 | `42` |
| `f32` | 32位浮点数 | — |
| `f64` | 64位浮点数 | `3.14`, `1.0e-10` |
| `char` | Unicode 字符 | `'a'`, `'\n'` |
| `bool` | 布尔值 | `true`, `false` |
| `str` | C 风格字符串（i8*） | `"hello"` |
| `void` | 无返回值 | — |

### 复合类型

```
struct Point { x: f64, y: f64 }       // 结构体
enum Color { Red, Green, Blue }         // 枚举
fn(T) U                                 // 函数指针
&T, &mut T                              // 引用
[T; N]                                  // 数组
```

### 泛型

```
fn identity<T>(x: T) -> T { return x; }
struct Pair<T, U> { first: T, second: U }
```

## 运算符优先级（从高到低）

| 优先级 | 运算符 | 结合性 | 说明 |
|--------|--------|--------|------|
| 最高 | `.` `::` | 左结合 | 成员访问、路径分隔 |
| | `*` `/` `%` | 左结合 | 乘除取模 |
| | `+` `-` | 左结合 | 加减 |
| | `<<` `>>` | 左结合 | 移位（1.1.0 起支持） |
| | `>` `<` `>=` `<=` | 左结合 | 比较 |
| | `==` `!=` | 左结合 | 相等判断 |
| | `&` | 左结合 | 按位与（1.1.0 起支持） |
| | `^` | 左结合 | 按位异或（1.1.0 起支持） |
| | `\|` | 左结合 | 按位或（1.1.0 起支持） |
| | `&&` | 左结合 | 逻辑与 |
| | `\|\|` | 左结合 | 逻辑或 |
| | `=` `->` `=>` | 右结合 | 赋值、箭头 |
| 最低 | `;` `,` `#` `?` `:` | — | 分隔符、三元、宏 |

> 注：位运算（`&` `\|` `^` `<<` `>>`）仅适用于整型，不可用于浮点。逻辑非 `!` 不支持，请用 `x == false` 或改写为正向分支；三元 `? :` 也不支持。

## 内置函数完整参考

### 输入输出

**`print(fmt: str, ...)`**
按 printf 风格将格式化结果输出到标准输出。不自动添加换行。格式符：`%lld`（i64）、`%d`（i32/bool）、`%f`（f64）、`%s`（str）、`%c`（char）。

**`format(fmt: str, ...) -> str`**
按 printf 风格格式化字符串并返回。

**`read_file(path: str) -> str`**
读取文件全部内容为字符串。

**`write_file(path: str, contents: str) -> i64`**
将字符串写入文件，返回写入的字节数。

### 断言

**`assert(cond: bool)`**
条件为 false 时终止程序。

**`assert_eq(a: i64, b: i64)`**
两个整数不等时终止程序。

### 字符串操作

**`len(s: str) -> i64`**
返回字符串字节长度。

**`int_to_str(n: i64) -> str`**
整数转字符串。

**`str_to_int(s: str) -> i64`**
字符串转整数，失败返回 0。

**`str_contains(haystack: str, needle: str) -> bool`**
判断字符串是否包含子串。

**`str_find(haystack: str, needle: str) -> i64`**
返回子串起始位置索引，未找到返回 -1。

**`str_cmp(a: str, b: str) -> i64`**
比较两字符串，返回差值（与 strcmp 语义一致）。

**`str_free(s: str)`**
释放字符串内存。

**`substr(s: str, start: i64, end: i64) -> str`**
截取子串。`start` 为起始索引（含），`end` 为结束索引（不含）。

### 哈希

**`hash_i64(n: i64) -> i64`**
对整数做哈希。

**`str_hash(s: str) -> i64`**
对字符串做哈希。

### 数学与张量

**`tensor(dims...) -> Tensor`**
创建 N 维零初始化张量，例如 `tensor(2, 3)`；元素类型可用 `tensor<f64>(2, 3)` 指定。

**`matmul(a: Tensor, b: Tensor) -> Tensor`**
二维张量矩阵乘法，要求 `a` 的列数等于 `b` 的行数。

**`sum(t: Tensor) -> T`**
张量求和，返回元素类型 `T`。同类还有 `mean`、`max`、`min`。

**`tensor_add(a: Tensor, b: Tensor) -> Tensor`**
张量逐元素加法。同类还有 `tensor_sub`、`tensor_mul`、`tensor_div`、`tensor_neg`。

**`blas_dot(a: Tensor, b: Tensor) -> T`**
BLAS 点积（内积），返回标量。同类还有 `blas_nrm2`、`blas_asum`、`blas_amax`、`blas_scal`、`blas_axpy`。

### 命令行

**`arg_count() -> i64`**
返回命令行参数个数（含程序名）。

**`arg(i: i64) -> str`**
返回第 `i` 个命令行参数。

## 标准库 API

### Option\<T\>

`Option<T>` / `Result<T, E>` 由标准库自动注入，无需自行定义（重复定义会报错）：

```aero
let o = Option::Some(42);
print("is_some=%d\n", o.is_some());
print("val=%lld\n", o.unwrap_or(0));

let none: Option<i64> = Option::None;
print("is_none=%d\n", none.is_none());
print("none val=%lld\n", none.unwrap_or(-1));
```

| 方法 | 签名 | 说明 |
|------|------|------|
| `is_some` | `(o: Option<T>) -> bool` | 是否为 Some |
| `is_none` | `(o: Option<T>) -> bool` | 是否为 None |
| `unwrap_or` | `(o: Option<T>, def: T) -> T` | 取 Some 值或默认值 |

### Result\<T, E\>

```aero
let r = Result::Ok(7);
print("is_ok=%d\n", r.is_ok());
print("ok val=%lld\n", r.unwrap_or(-1));

let e: Result<i64, str> = Result::Err("boom");
print("is_err=%d\n", e.is_err());
print("err val=%s\n", e.unwrap_err_or("none"));
```

| 方法 | 签名 | 说明 |
|------|------|------|
| `is_ok` | `(r: Result<T, E>) -> bool` | 是否为 Ok |
| `is_err` | `(r: Result<T, E>) -> bool` | 是否为 Err |
| `unwrap_or` | `(r: Result<T, E>, def: T) -> T` | 取 Ok 值或默认值 |
| `unwrap_err_or` | `(r: Result<T, E>, def: E) -> E` | 取 Err 值或默认值 |

### 内置 Trait

```
Copy: 复制语义
Clone: 显式克隆
Drop: 析构
Iterator: 迭代器
IntoIterator: 可迭代
Add<RHS, Output>: 加法
Eq<RHS>: 相等比较
Ord<RHS>: 顺序比较
```

### Vec\<T\>

| 函数/方法 | 签名 | 说明 |
|-----------|------|------|
| `Vec::new` | `() -> Vec<T>` | 创建空向量 |
| `Vec::with_cap` | `(cap: i64) -> Vec<T>` | 预分配容量 |
| `push` | `(v: &mut Vec<T>, val: T)` | 末尾追加 |
| `pop` | `(v: &mut Vec<T>) -> T` | 弹出末尾元素 |
| `get` | `(v: &mut Vec<T>, i: i64) -> T` | 索引读取 |
| `set` | `(v: &mut Vec<T>, i: i64, val: T)` | 索引写入 |
| `len` | `(v: &mut Vec<T>) -> i64` | 元素个数 |
| `is_empty` | `(v: &mut Vec<T>) -> bool` | 是否为空 |
| `free` | `(v: &mut Vec<T>)` | 释放内存 |
| `v[i]` | 索引操作 | 读取元素 |
| `v[i] = x` | 索引操作 | 写入元素 |

### String

| 函数/方法 | 签名 | 说明 |
|-----------|------|------|
| `String::new` | `() -> String` | 创建空字符串 |
| `String::from` | `(s: str) -> String` | 从 C 字符串创建 |
| `String::with_cap` | `(cap: i64) -> String` | 预分配容量 |
| `len` | `(s: &mut String) -> i64` | 字符数 |
| `is_empty` | `(s: &mut String) -> bool` | 是否为空 |
| `at` | `(s: &mut String, i: i64) -> char` | 取第 i 个字符 |
| `data` | `(s: &mut String) -> str` | 转为 C 字符串 |
| `push` | `(s: &mut String, c: char)` | 追加字符 |
| `push_str` | `(s: &mut String, str: str)` | 追加字符串 |
| `pop` | `(s: &mut String) -> char` | 弹出末尾字符 |
| `starts_with` | `(s: &mut String, p: str) -> bool` | 前缀判断 |
| `ends_with` | `(s: &mut String, p: str) -> bool` | 后缀判断 |
| `clear` | `(s: &mut String)` | 清空 |
| `free` | `(s: &mut String)` | 释放内存 |
| `s[i]` | 索引操作 | 取第 i 个字符 |

### HashMap\<V\>

创建：`hash_map_new() -> HashMap<V>`。注意：键类型固定为 `i64`，`get` 必须提供默认值。

| 方法 | 签名 | 说明 |
|------|------|------|
| `insert` | `(m: &mut HashMap<V>, key: i64, val: V)` | 插入键值对 |
| `get` | `(m: &mut HashMap<V>, key: i64, def: V) -> V` | 读取值，缺省返回 def |
| `contains` | `(m: &mut HashMap<V>, key: i64) -> bool` | 键是否存在 |
| `remove` | `(m: &mut HashMap<V>, key: i64) -> bool` | 删除键值对 |
| `len` | `(m: HashMap<V>) -> i64` | 条目数 |
| `is_empty` | `(m: HashMap<V>) -> bool` | 是否为空 |

### BTreeMap

创建：`btree_map_new() -> BTreeMap`。注意：键和值类型固定为 `i64`，`get` 必须提供默认值。

| 方法 | 签名 | 说明 |
|------|------|------|
| `insert` | `(m: &mut BTreeMap, key: i64, val: i64)` | 插入键值对 |
| `get` | `(m: &mut BTreeMap, key: i64, def: i64) -> i64` | 读取值，缺省返回 def |
| `contains` | `(m: &mut BTreeMap, key: i64) -> bool` | 键是否存在 |
| `remove` | `(m: &mut BTreeMap, key: i64) -> bool` | 删除键值对 |
| `len` | `(m: BTreeMap) -> i64` | 条目数 |
| `is_empty` | `(m: BTreeMap) -> bool` | 是否为空 |
| `keys` | `(m: &mut BTreeMap) -> Vec<i64>` | 获取所有键（升序） |

### BTreeSet

创建：`btree_set_new() -> BTreeSet`。注意：元素类型固定为 `i64`。

| 方法 | 签名 | 说明 |
|------|------|------|
| `insert` | `(s: &mut BTreeSet, val: i64)` | 插入值 |
| `contains` | `(s: &mut BTreeSet, val: i64) -> bool` | 是否包含 |
| `remove` | `(s: &mut BTreeSet, val: i64) -> bool` | 删除值 |
| `len` | `(s: BTreeSet) -> i64` | 元素个数 |
| `is_empty` | `(s: BTreeSet) -> bool` | 是否为空 |
| `to_vec` | `(s: &mut BTreeSet) -> Vec<i64>` | 转为向量（升序） |

### LinkedList\<T\>

创建：`linked_list_new() -> LinkedList<T>`。

| 方法 | 签名 | 说明 |
|------|------|------|
| `len` | `(l: LinkedList<T>) -> i64` | 元素个数 |
| `is_empty` | `(l: LinkedList<T>) -> bool` | 是否为空 |
| `front` | `(l: &mut LinkedList<T>) -> Option<T>` | 取首元素 |
| `back` | `(l: &mut LinkedList<T>) -> Option<T>` | 取尾元素 |
| `push_front` | `(l: &mut LinkedList<T>, val: T)` | 头部插入 |
| `push_back` | `(l: &mut LinkedList<T>, val: T)` | 尾部插入 |
| `pop_front` | `(l: &mut LinkedList<T>) -> Option<T>` | 弹出头部 |
| `pop_back` | `(l: &mut LinkedList<T>) -> Option<T>` | 弹出尾部 |
| `get` | `(l: &mut LinkedList<T>, i: i64, def: T) -> T` | 索引读取，越界返回 def |
| `set` | `(l: &mut LinkedList<T>, i: i64, val: T) -> bool` | 索引写入 |
| `remove` | `(l: &mut LinkedList<T>, i: i64) -> bool` | 删除指定位置 |
| `reverse` | `(l: &mut LinkedList<T>)` | 反转链表 |
| `clear` | `(l: &mut LinkedList<T>)` | 清空链表 |

### 排序算法

作用于 `Vec<i64>`：

| 函数 | 签名 | 说明 |
|------|------|------|
| `sort` | `(v: &mut Vec<i64>)` | 原地升序排序 |
| `binary_search` | `(v: &mut Vec<i64>, target: i64) -> i64` | 二分查找，返回索引或 -1 |
| `reverse` | `(v: &mut Vec<i64>)` | 原地反转 |

### 函数式编程

目前为内部实现（`_` 前缀，暂未开放公开 API），作用于 `Vec<i64>`：

| 函数 | 签名 | 说明 |
|------|------|------|
| `_filter_impl` | `(v: &mut Vec<i64>, pred: FnPred) -> Vec<i64>` | 过滤 |
| `_map_impl` | `(v: &mut Vec<i64>, trans: FnTrans) -> Vec<i64>` | 映射 |
| `_reduce_impl` | `(v: &mut Vec<i64>, red: FnRed, init: i64) -> i64` | 归约 |

其中 `FnPred = fn(i64) -> bool`，`FnTrans = fn(i64) -> i64`，`FnRed = fn(i64, i64) -> i64`。

### 路径操作

返回 `String`：

| 函数 | 签名 | 说明 |
|------|------|------|
| `path_join` | `(a: str, b: str) -> String` | 路径拼接 |
| `path_basename` | `(p: str) -> String` | 获取文件名 |
| `path_dirname` | `(p: str) -> String` | 获取目录名 |
| `path_extension` | `(p: str) -> String` | 获取扩展名 |

### JSON 编解码

编码函数返回 `String`：

| 函数 | 签名 | 说明 |
|------|------|------|
| `json_escape` | `(s: str) -> String` | 字符串 JSON 转义 |
| `json_string` | `(s: str) -> String` | 生成 JSON 字符串值 |
| `json_number_i64` | `(n: i64) -> String` | 生成 JSON 整数值 |
| `json_number_f64` | `(n: f64) -> String` | 生成 JSON 浮点值 |
| `json_bool` | `(b: bool) -> String` | 生成 JSON 布尔值 |
| `json_null` | `() -> String` | 生成 JSON null |
| `json_join` | `(a: String, sep: str, b: String) -> String` | 拼接两个 JSON 片段 |
| `json_parse_i64` | `(s: str) -> i64` | 解析字符串起始处的整数 |
| `json_parse_bool` | `(s: str) -> bool` | 解析字符串起始处的布尔值 |
| `json_unescape` | `(s: str) -> String` | 反转义 JSON 字符串 |
| `json_find_key` | `(s: str, key: str) -> i64` | 查找键后值的位置索引 |

### 自动微分 (Grad)

所有操作都接收 `&mut Grad` 作为第一个参数，节点用 `i64` 索引表示：

| 函数 | 签名 | 说明 |
|------|------|------|
| `grad_new` | `() -> Grad` | 创建梯度计算图 |
| `g_leaf` | `(g: &mut Grad, v: f64) -> i64` | 创建叶子节点 |
| `g_add` | `(g: &mut Grad, a: i64, b: i64) -> i64` | 加法节点 |
| `g_mul` | `(g: &mut Grad, a: i64, b: i64) -> i64` | 乘法节点 |
| `g_neg` | `(g: &mut Grad, a: i64) -> i64` | 取负 |
| `g_inv` | `(g: &mut Grad, a: i64) -> i64` | 取倒数 |
| `g_sub` | `(g: &mut Grad, a: i64, b: i64) -> i64` | 减法节点 |
| `g_div` | `(g: &mut Grad, a: i64, b: i64) -> i64` | 除法节点 |
| `g_backward` | `(g: &mut Grad, root: i64)` | 反向传播 |
| `g_grad` | `(g: &mut Grad, node: i64) -> f64` | 获取梯度值 |
| `g_val` | `(g: &mut Grad, node: i64) -> f64` | 获取节点值 |

## 官方核心 Crate API

> 以下 crate 属于第三方库（需在 `Aero.toml` 中声明依赖并链接对应系统库）。代码块中的函数体为便于编译的简化占位，真实实现见各 crate 源码；函数签名与当前版本一致。

### aero-json

```aero
struct JsonValue {
    is_null: bool,
    bool_val: bool,
    int_val: i64,
    str_val: str,
    raw: str,
}

fn parse_json(s: str) -> JsonValue {
    return JsonValue { is_null: false, bool_val: false, int_val: 0, str_val: "", raw: s };
}

fn json_get_int(s: str, key: str) -> i64 {
    return 0;
}

fn json_get_str(s: str, key: str) -> str {
    return "";
}

fn json_number(n: i64) -> str {
    return int_to_str(n);
}

let v = parse_json("{\"count\": 1}");
print("raw: %s\n", v.raw);
print("num: %s\n", json_number(42));
```

### aero-csv

```aero
// 输入为整段内容字符串（而非文件路径）
fn csv_count_lines(s: str) -> i64 {
    let src = String::from(s);
    let count = 0;
    let i = 0;
    while (i < src.len()) {
        if (src.at(i) == 10) {
            count = count + 1;
        }
        i = i + 1;
    }
    return count;
}

fn csv_get_record(s: str, n: i64) -> str {
    return substr(s, 0, 999);
}

fn csv_count_fields(record: str) -> i64 {
    return str_find(record, ",") + 1;
}

fn csv_get_field(record: str, n: i64) -> str {
    return substr(record, 0, 999);
}

let csv = "name,age\nAlice,30\n";
print("lines: %lld\n", csv_count_lines(csv));
```

### aero-logger

```aero
struct LogLevel {
    value: i64,
    label: str,
}

fn log_error(msg: str) {
    print("[ERROR] %s\n", msg);
}

fn log_warn(msg: str) {
    print("[WARN]  %s\n", msg);
}

fn log_info(msg: str) {
    print("[INFO]  %s\n", msg);
}

fn log_debug(msg: str) {
    print("[DEBUG] %s\n", msg);
}

fn log(level: str, msg: str) {
    print("[%s] %s\n", level, msg);
}

fn log_separator() {
    print("---\n");
}

fn log_kv(key: str, value: str) {
    print("  %s: %s\n", key, value);
}

fn log_section(title: str) {
    print("== %s ==\n", title);
}

log_section("Application Start");
log_info("boot complete");
```

### aero-test

```aero
fn assert_true(cond: bool, msg: str) {
    if (cond) {
        print("PASS: %s\n", msg);
    } else {
        print("FAIL: %s\n", msg);
    }
}

fn assert_eq_i64(got: i64, expected: i64, msg: str) {
    if (got == expected) {
        print("PASS: %s\n", msg);
    } else {
        print("FAIL: %s\n", msg);
    }
}

fn assert_eq_i32(got: i32, expected: i32, msg: str) {
    if (got == expected) {
        print("PASS: %s\n", msg);
    } else {
        print("FAIL: %s\n", msg);
    }
}

fn assert_eq_str(got: str, expected: str, msg: str) {
    if (str_cmp(got, expected) == 0) {
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

test_group("math");
assert_eq_i64(2 + 3, 5, "add");
test_summary(1, 0);
```

### aero-collections

```aero
struct Stack {
    data: Vec<i64>,
    top: i64,
}

fn stack_new() -> Stack {
    return Stack { data: Vec::new(), top: 0 };
}

fn stack_push(s: &mut Stack, val: i64) {
    s.data.push(val);
    s.top = s.top + 1;
}

fn stack_pop(s: &mut Stack) -> i64 {
    s.top = s.top - 1;
    return s.data.get(s.top);
}

fn stack_peek(s: &mut Stack) -> i64 {
    return s.data.get(s.top - 1);
}

fn stack_is_empty(s: &mut Stack) -> bool {
    return s.top == 0;
}

fn stack_len(s: &mut Stack) -> i64 {
    return s.top;
}

struct Queue {
    data: Vec<i64>,
    head: i64,
    tail: i64,
    count: i64,
}

fn queue_new() -> Queue {
    return Queue { data: Vec::new(), head: 0, tail: 0, count: 0 };
}

fn queue_enqueue(q: &mut Queue, val: i64) {
    q.data.push(val);
    q.count = q.count + 1;
}

fn queue_dequeue(q: &mut Queue) -> i64 {
    q.count = q.count - 1;
    return q.data.get(q.head);
}

fn queue_is_empty(q: &mut Queue) -> bool {
    return q.count == 0;
}

fn queue_len(q: &mut Queue) -> i64 {
    return q.count;
}

let st = stack_new();
stack_push(&mut st, 10);
stack_push(&mut st, 20);
let popped = stack_pop(&mut st);
print("stack len=%lld pop=%lld\n", stack_len(&mut st), popped);
```

### aero-time

```aero
fn format_timestamp(secs: i64) -> str {
    return format("%lld.000", secs);
}

fn format_duration(secs: i64) -> str {
    if (secs < 60) {
        return format("%llds", secs);
    }
    let minutes = secs / 60;
    let remain_secs = secs % 60;
    if (minutes < 60) {
        return format("%lldm %llds", minutes, remain_secs);
    }
    let hours = minutes / 60;
    let remain_min = minutes % 60;
    return format("%lldh %lldm %llds", hours, remain_min, remain_secs);
}

fn format_bytes(bytes: i64) -> str {
    if (bytes < 1024) {
        return format("%lld B", bytes);
    }
    let kb = bytes / 1024;
    if (kb < 1024) {
        return format("%lld KB", kb);
    }
    let mb = kb / 1024;
    if (mb < 1024) {
        return format("%lld MB", mb);
    }
    let gb = mb / 1024;
    return format("%lld GB", gb);
}

fn timestamp() -> str {
    return "now";
}

print("%s\n", format_duration(3661));
print("%s\n", format_bytes(1536));
print("%s\n", timestamp());
```

### aero-io

```aero
// 文本读取 / 存在性检查复用编译器内置函数
fn read_text_file(path: str) -> str {
    return read_file(path);
}

fn path_exists(path: str) -> bool {
    return file_exists(path);
}

fn file_extension(path: str) -> str {
    let p = String::from(path);
    let n = p.len();
    let dot = -1;
    let i = 0;
    while (i < n) {
        if (p.at(i) == 46) {
            dot = i;
        }
        i = i + 1;
    }
    if (dot < 0) {
        return "";
    }
    return substr(path, dot + 1, n);
}

fn basename(path: str) -> str {
    let p = String::from(path);
    let n = p.len();
    let sep = -1;
    let i = 0;
    while (i < n) {
        if ((p.at(i) == 47) || (p.at(i) == 92)) {
            sep = i;
        }
        i = i + 1;
    }
    return substr(path, sep + 1, n);
}

print("ext: %s\n", file_extension("docs/report.txt"));
print("base: %s\n", basename("docs/report.txt"));
```

### aero-toml

```aero
struct TomlValue {
    key: str,
    value: str,
}

fn toml_parse_line(line: str) -> TomlValue {
    let eq = str_find(line, "=");
    if (eq < 0) {
        return TomlValue { key: "", value: "" };
    }
    let key = substr(line, 0, eq);
    return TomlValue { key: key, value: substr(line, eq + 1, 999) };
}

fn toml_is_section(line: str) -> bool {
    return str_find(line, "[") == 0;
}

fn toml_is_comment(line: str) -> bool {
    return str_find(line, "#") == 0;
}

let lv = toml_parse_line("name = \"Aero\"");
print("key=%s val=%s\n", lv.key, lv.value);
print("section=%d comment=%d\n", toml_is_section("[build]"), toml_is_comment("# hi"));
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

fn http_post(url: str, body: str) -> HttpResponse {
    return HttpResponse { data: "", ok: false };
}

fn http_ok(resp: HttpResponse) -> bool {
    return resp.ok;
}

fn http_body(resp: HttpResponse) -> str {
    return resp.data;
}

let resp = http_get("https://example.com");
print("ok=%d\n", http_ok(resp));
```

### aero-crypto

```aero
fn hex_encode(data: str, len: i64) -> str {
    return "";
}

fn sha256(input: str) -> str {
    return sha256_hash(input);
}

fn sha256_hash(input: str) -> str {
    return "";
}

fn xor_obfuscate(data: str, key: str) -> str {
    return data;
}

fn constant_time_eq(a: str, b: str) -> bool {
    return str_cmp(a, b) == 0;
}

print("eq=%d\n", constant_time_eq("abc", "abc"));
print("hex=%s\n", hex_encode("hi", 2));
```

### aero-regex

```aero
struct RegexMatch {
    matched: bool,
    start: i32,
    end: i32,
}

struct Regex {
    preg: [i32; 16],
    valid: bool,
}

fn regex_compile(pattern: str) -> Regex {
    return Regex { preg: [0; 16], valid: false };
}

fn regex_is_match(re: &Regex, text: str) -> bool {
    return false;
}

fn regex_find(re: &Regex, text: str) -> RegexMatch {
    return RegexMatch { matched: false, start: 0, end: 0 };
}

fn regex_free(re: &Regex) {
}

fn regex_test(pattern: str, text: str) -> bool {
    return regex_is_match(regex_compile(pattern), text);
}

fn regex_group(text: str, m: RegexMatch) -> str {
    return "";
}

let re = regex_compile("[0-9]+");
print("compiled valid=%d\n", re.valid);
print("match=%d\n", regex_test("[0-9]+", "abc123"));
```

### aero-net

```aero
struct TcpSocket {
    fd: *i32,
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
print("created connected=%d\n", sock.connected);
net_cleanup();
```

## 编译器 CLI 命令参考

```
aero run <file.aero | 包目录>      编译并运行
aero run <file.aero> -O0|-O1|-O2|-O3  指定优化级别运行
aero build [file.aero | 目录]      编译为独立可执行文件（AOT）
aero build <file.aero> -O0|-O1|-O2|-O3  指定优化级别编译
aero build <file> --target <triple>  交叉编译（如 x86_64-unknown-linux-gnu）
aero new <name>                   创建新项目骨架
aero test [file.aero]             运行测试
aero fmt <file.aero>              格式化源码（--check 只检查；--width/--indent 调参）
aero bench <file.aero>            性能基准测试
aero clippy <file.aero>           静态分析（100+ 规则）
aero cov <file.aero>              覆盖率报告
aero --lsp | aero lsp            启动语言服务器（JSON-RPC 2.0）
aero lock [dir]                 解析依赖并生成 Aero.lock
aero publish <dir>              发布库包到 registry
aero ls                         列出 registry 中的包与版本
aero registry                   打印 registry 位置
```
