# 17 C++ 绑定（bindgen）

Aero 不仅能编成独立 exe、Python 扩展、Android/iOS 动态库，还能**直接给 C++ 项目调用**。`aero build --cpp` 一次生成两样东西：

- **共享库**（`.dll` / `.so` / `.dylib`）：里面是每个 `#[export]` 函数的 C ABI 符号；
- **C++ 头文件**（`<名字>.hpp`）：`extern "C"` 声明这些函数，附带类型映射。

C++ 侧 `#include` 头文件、链接库，就能直接调用 Aero 写的函数——不需要胶水层、不需要手写桥接。这类似 Rust 生态的 `cxx`，但面向"C++ 调用 Aero"方向（v1）。

## 快速开始

```aero
// calc.aero
#[export]
fn add(a: i64, b: i64) -> i64 {
    return a + b;
}

#[export]
fn double(x: f64) -> f64 {
    return x * 2.0;
}

#[export]
fn is_even(n: i64) -> bool {
    if (n % 2 == 0) {
        return true;
    }
    return false;
}

extern "C" fn strlen(s: str) -> i64;

#[export]
fn str_len(s: str) -> i64 {
    return strlen(s);
}
```

生成绑定：

```
aero build --cpp calc.aero
```

产出 `calc.dll`（Windows）和 `calc.hpp`：

```cpp
// calc.hpp（自动生成）
#include <cstdint>

extern "C" {
int64_t add(int64_t a, int64_t b);
double double_(double x) asm("double");
bool is_even(int64_t n);
int64_t str_len(const char* s);
} // extern "C"
```

C++ 里直接用：

```cpp
// main.cpp
#include <cstdio>
#include <cassert>
#include "calc.hpp"

int main() {
    assert(add(40, 2) == 42);
    assert(double_(3.5) == 7.0);
    assert(is_even(10) == true);
    assert(str_len("hello") == 5);
    std::printf("ALL PASS\n");
}
```

编译链接（MinGW 可直接链 DLL）：

```
g++ main.cpp -I. calc.dll -o main.exe
```

## 类型映射（v1）

| Aero | C++ | 说明 |
| --- | --- | --- |
| `i64` | `int64_t` | 有符号 64 位整数 |
| `i32` | `int32_t` | 有符号 32 位整数 |
| `f64` | `double` | 双精度浮点 |
| `bool` | `bool` | 布尔 |
| `str` | `const char*` | NUL 结尾的 C 字符串 |
| 无返回值 | `void` | 空返回 |

v1 不支持 `String`、`Vec`、裸指针 `*T` 跨边界。出现这些类型时编译器会报错并列出支持范围。

## C++ 关键字冲突

如果 Aero 函数名恰好是 C++ 关键字（比如 `double`），生成的 C++ 标识符会加下划线（`double_`），并用 `asm("符号")` 标注真实符号名，链接时不受影响：

```cpp
double double_(double x) asm("double");   // 源码里写 double_，链接符号是 double
```

## 交叉编译

`--cpp` 复用 `--shared` 的交叉编译链路：

```
# Android
aero build --cpp --target aarch64-linux-android --ndk <NDK路径> calc.aero

# iOS（macOS + Xcode）
aero build --cpp --target aarch64-apple-ios calc.aero
```

## 和 Python 绑定的区别

| | `--pyext` | `--cpp` |
| --- | --- | --- |
| 消费方 | Python `import` | C++ `#include` + 链接 |
| 导出标记 | `#[py_export]` | `#[export]` |
| 生成物 | `<name>.pyd` + 胶水层 | `<name>.dll` + `<name>.hpp` |
| 额外运行库 | CPython | 无（纯 C ABI） |

`#[py_export]` 的函数同时是 `#[export]`，所以 C++ 也能调用它导出的原始符号。

## 练习

1. 写一个 `#[export] fn fib(n: i64) -> i64`，用 `aero build --cpp` 生成绑定，在 C++ 里调用 `fib(10)`。
2. 写一个返回 `f64` 的 `#[export]` 函数，确认 C++ 侧收到 `double`。
3. 故意给函数取名 `class` 或 `new`，看生成的头文件如何处理。
