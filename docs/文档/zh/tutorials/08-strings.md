# 08 字符串

## 字符串字面量

```aero
let name = "Aero";
print("hello\n");   // \n 是换行
```

字符串类型叫 `str`，字面量用双引号。转义字符：

| 写法 | 含义 |
| --- | --- |
| `\n` | 换行 |
| `\t` | 制表符 |
| `\"` | 双引号 |
| `\\` | 反斜杠 |

```aero
print("say \"hi\"\n");   // say "hi"
print("a\tb\n");         // a 然后制表符然后 b
```

## 拼接：`+`

```aero
print("%s\n", "foo" + "bar");   // foobar
print("%s\n", "a" + "b" + "c"); // abc
```

`+` 可以把字符串接起来。两个细节：

- 字面量 + 字面量（`"foo" + "bar"`）在**编译期**就合并成一个常量，零运行时开销。
- 只要有一边是变量，就是**运行时拼接**：会分配新内存，用完要 `str_free`（见下文"谁负责释放"）。

```aero
let s = "x";
print("%s\n", s + "y");   // xy（运行时拼接，结果要 str_free）
```

## 比较

```aero
let a = "abc";
let b = "abd";
print(a == "abc");   // true
print(a != b);       // true
print(a < b);        // true，按字典序
print(a > b);        // false
print(a <= "abc");   // true
print(a >= b);       // false
```

六个比较运算符字符串全支持，比较的是**字典序**（逐字节比）。这是 Aero 字符串系统的一个特色——很多语言里字符串只有 `==` 和 `!=`。

## 长度和索引

```aero
let s = "hello";
print(len(s));   // 5
print(s[0]);     // 104，'h' 的字节值
print(s[1]);     // 101，'e'
```

- `len(s)` 返回字符（字节）数。
- `s[i]` 返回第 i 个字节，类型是 `i64`（Aero 0.1 没有单字符类型）。`'h'` 的 ASCII 码是 104，所以 `s[0]` 是 104。

想打印某个字符本身，用 `%c` 不行——Aero 0.1 的格式串只支持 `%d` 和 `%s`。把字节转回字符串可以用 `substr`：

```aero
let s = "hello";
print("%s\n", substr(s, 1, 2));   // e
```

## 子串：substr

```aero
let s = "hello world";
print("%s\n", substr(s, 0, 5));    // hello
print("%s\n", substr(s, 6, 11));   // world
print("%s\n", substr(s, 3, 3));    // （空串，end <= start）
print("%s\n", substr(s, 0, 100));  // hello world（越界自动截断）
```

`substr(s, start, end)` 取从 `start` 到 `end`（不含 `end`）的部分。边界会自动限制在字符串长度内，`end <= start` 得到空字符串。返回值是新分配的内存，用完 `str_free`。

## 数字与字符串互转

```aero
let n = int_to_str(42);
print("%s\n", n);   // 42
str_free(n);

let m = str_to_int("123");
print("%d\n", m);   // 123

print("%d\n", str_to_int("abc"));   // 0，解析失败返回 0
```

- `int_to_str(n)`：整数 → 字符串。
- `str_to_int(s)`：字符串 → 整数（底层是 C 的 strtoll，解析失败返回 0）。

## 查找与比较

```aero
let s = "hello world";

print("%d\n", str_contains(s, "world"));   // 1（真）
print("%d\n", str_contains(s, "xyz"));     // 0（假）

print("%d\n", str_find(s, "world"));       // 6，位置
print("%d\n", str_find(s, "xyz"));         // -1，找不到

print("%d\n", str_cmp("abc", "abd"));      // 负数（abc 更小）
print("%d\n", str_cmp("abc", "abc"));      // 0
```

- `str_contains(haystack, needle)`：布尔，haystack 里有没有 needle。
- `str_find(haystack, needle)`：needle 第一次出现的位置，找不到返回 -1。
- `str_cmp(a, b)`：C 风格比较，返回负数/0/正数。

## 谁负责释放（重点）

字符串的内存分两种来源：

1. **字面量**（`"hello"`、编译期拼接的结果）：存在程序的只读数据区，**永远不需要释放**，也**不能释放**。
2. **运行时分配的结果**（`int_to_str`、`substr`、运行时 `+` 拼接）：用 `malloc` 分配的新内存，**用完要 `str_free`**。

```aero
// 对的比例
let a = int_to_str(100);
print("%s\n", a);
str_free(a);          // 释放

// 错的比例：循环里不释放，内存一直涨
let i = 0;
while (i < 1000) {
    let t = int_to_str(i);   // 每次分配
    print("%s\n", t);
    // str_free(t);          // 忘了释放 → 内存泄漏
    i = i + 1;
}
```

判断规则很简单：**只有字面量不用管；函数调用返回的字符串，除非你能证明它是字面量，否则用完 `str_free`**。

这一套"谁分配谁释放"的规矩，到第 10 章你会看到 Aero 更优雅的答案（Arena）。

## 练习

1. 打印 `len("Aero")`，验证是 4。
2. 把 `"Hello, Aero!"` 用 `substr` 拆成 `"Hello"` 和 `"Aero"` 分别打印。
3. 用 `str_to_int` 把 `"2026"` 转成数字，加 1，再用 `int_to_str` 转回字符串打印。
4. 判断 `"a" + "b"` 的结果需不需要 `str_free`，然后写代码验证你的判断。
