# 12 FFI：调用 C

## Aero 不是孤岛

现实世界有一堆现成的 C 库：系统 API、图形库、数学库……Aero 提供 FFI（Foreign Function Interface，外部函数接口），让你直接调用 C 函数，不用写胶水代码。

## 声明一个 C 函数

```aero
extern "C" fn strlen(s: str) -> i64;
extern "C" fn abs(x: i32) -> i32;

print(strlen("hello"));   // 5
print(abs(-42));          // 42
```

要点：

- `extern "C"` 表示"这是 C ABI 的外部函数"；
- **没有函数体**，以分号结束——它只是声明，函数本体在某个 C 库里；
- 调用方法和普通函数一样。

`strlen` 是 C 标准库的函数（返回字符串长度）。`abs` 是取绝对值。

## 符号别名：Aero 名 ≠ C 符号名

默认情况下，Aero 里的函数名就是链接时找的 C 符号名。但有些 C 函数名和你想用的 Aero 名不一样，用 `= "符号"` 指定：

```aero
extern "C" fn string_len(s: str) -> i64 = "strlen";
extern "C" fn abs_c(x: i32) -> i32 = "abs";

print(string_len("hi"));   // 2
```

这里 Aero 里叫 `string_len`，但链接时找的 C 符号是 `strlen`。不写 `= "..."` 就用函数名本身。

## 类型限制

C 的 ABI 很古老，不是所有 Aero 类型都能穿过去。0.1 允许的：

| 位置 | 允许的类型 |
| --- | --- |
| 参数 | `i32`、`i64`、`str`（即 `char*`）、`*T` 指针 |
| 返回 | `i32`、`i64`、`*T` 指针、无返回值 |

**`bool` 不行**，`数组/元组` 不行：

```aero
// extern "C" fn bad(x: bool) -> i64;   // 编译错误：bool 不是 C ABI 兼容类型
```

## 字符串和 C 的 `char*` 是一回事

Aero 的 `str` 底层就是 C 的 `char*`（`i8*`）。所以：

```aero
extern "C" fn putchar(c: i32) -> i32;

putchar(65);   // 输出字符 'A'（ASCII 65）
```

`str` 传进 C 函数，C 函数拿到的就是 NUL 结尾的字符串指针。反过来，C 函数返回的 `char*` 在 Aero 里就是 `str`。

## 链接系统库：[link] 段

声明了函数，还要让链接器把库接上。C 标准库（strlen、abs）默认就有；**别的库要在 `Aero.toml` 里声明**。

比如调 Windows API `GetTickCount`（返回系统启动以来的毫秒数），它住在 `kernel32.dll`：

```toml
[package]
name = "winpkg"
version = "0.1.0"

[link]
libs = ["kernel32"]   # 链接 -lkernel32
```

```aero
extern "C" fn GetTickCount() -> i32;

let t = GetTickCount();
print("%d\n", t);   // 比如 12560265
```

`[link].libs` 是库名列表（`-l<名字>`），`[link].lib_paths` 是额外的库搜索路径（`-L<路径>`）。

## 链接自己的 C 库

你可以先用 gcc 编译一个 C 静态库，再在 Aero 里调它。假设 `mylib.c`：

```c
int aero_add(int a, int b) { return a + b; }
int aero_mul3(int a) { return a * 3; }
```

编成库：

```
gcc -c mylib.c -o mylib.o
ar rcs libmylib.a mylib.o
```

然后 `Aero.toml`：

```toml
[link]
libs = ["mylib"]
lib_paths = ["."]
```

Aero 代码：

```aero
extern "C" fn aero_add(a: i32, b: i32) -> i32;
extern "C" fn aero_mul3(a: i32) -> i32;

print(aero_add(2, 3));    // 5
print(aero_mul3(4));      // 12
```

注意库文件名是 `libmylib.a`，`libs` 里写 `mylib`（去掉 `lib` 前缀和 `.a` 后缀），这是链接器的惯例。

## JIT 和 AOT 的区别

- `aero run`（JIT）：函数符号在运行时从系统的导出表里找。大部分 C 标准库能直接用，但个别名字可能有出入（比如某些 CRT 函数在 Windows 导出表里叫 `_snprintf`）。
- `aero build`（AOT）：交给 gcc 链接，符号解析和普通 C 程序完全一致，`[link]` 段在这里才真正生效。

**结论：需要调外部库的程序，用 `aero build` 出 exe 最可靠。**

## 练习

1. 声明并调用 `toupper(c: i32) -> i32`（C 标准库，把小写字母转大写），把 `'a'` 转成 `'A'` 的 ASCII 码打印。
2. 声明 `strcmp(a: str, b: str) -> i32`，比较两个字符串，打印结果，再和 Aero 的 `str_cmp` 对比。
3. 写一个 C 文件提供 `int square(int)`，编译成静态库，用 `[link]` 段在 Aero 里调用，用 `aero build` 出 exe 验证。
4. 试试声明一个带 `bool` 参数的 extern 函数，看编译错误信息。
