# 14 包管理器与测试

## 什么是包

从第 1 章到现在，你写的都是**单文件程序**。真实项目要拆文件、组织目录、跑测试。Aero 的包（package）就是干这个的。

一个包就是一个目录，结构固定：

```
mytool/
├── Aero.toml        # 包描述文件
└── src/
    └── main.aero    # 主程序
```

## 创建包

```bash
aero new mytool
```

自动生成：

- `Aero.toml`：包名、版本、edition；
- `src/main.aero`：一个打印 `Hello, Aero!` 的骨架。

生成的 `Aero.toml`：

```toml
[package]
name = "mytool"
version = "0.1.0"
edition = "2024"
```

## 运行和构建

在包目录里：

```bash
aero run          # 编译并运行 src/main.aero
aero build        # 编译成独立可执行文件
```

`aero build` 的输出在 `target/aero/mytool.exe`。整个构建流程和单文件一样，只是多了包的概念。

## 依赖：拆代码库

项目大了，把公共代码抽成**库包**（只有函数定义、没有 main），主包引用它。

假设 `libs/mathlib` 目录：

```toml
# libs/mathlib/Aero.toml
[package]
name = "mathlib"
version = "0.1.0"
```

```aero
// libs/mathlib/src/main.aero —— 库包，提供函数
fn square(x: i64) -> i64 {
    return x * x;
}
```

主包在 `Aero.toml` 里声明依赖：

```toml
[package]
name = "mytool"
version = "0.1.0"

[dependencies]
mathlib = { path = "../libs/mathlib" }
```

主程序的 `src/main.aero` 直接调用库里的函数：

```aero
print(square(9));   // 81
```

编译时，Aero 会把依赖库的代码和主程序**合并成一份源码**再编译。规则：

- 库包提供函数，主包调用；
- 库包的顶层**不能有可执行语句**（`print(...)` 之类的语句会被拒绝）——库只提供定义；
- 函数名不能重复。

## 断言：assert

```aero
assert(1 + 1 == 2);        // 条件为真，无事发生
// assert(1 + 1 == 3);     // 失败：打印诊断信息并终止程序
```

`assert(条件)` 条件为假时打印诊断并让程序以非零退出码终止。`assert_eq(a, b)` 专门比相等：

```aero
assert_eq(add(2, 3), 5);   // add 的结果必须是 5
```

断言失败长这样（程序退出码非 0）：

```
[assert] assertion failed
```

## 测试框架：test_ 前缀

Aero 的测试约定：**函数名以 `test_` 开头**，就会被 `aero test` 收集并逐个运行。

```aero
// tests 写在 main.aero 里
fn test_square() {
    assert_eq(square(4), 16);
    assert_eq(square(-3), 9);
}

fn test_add() {
    assert_eq(add(1, 2), 3);
}
```

运行：

```bash
aero test
```

输出：

```
PASS  test_square
PASS  test_add
2 tests: 2 passed, 0 failed
```

任何一个测试失败（断言失败），对应的函数打印 `FAIL`，最终退出码非 0。CI 里靠这个判断构建是否通过。

也可以测单个文件：

```bash
aero test mytests.aero
```

或者把测试文件放在 `tests/` 目录（`.aero` 文件），`aero test` 会跑目录里所有测试文件。

## 一个完整流程

```bash
aero new calculator
cd calculator
# 编辑 src/main.aero，写函数和 test_ 测试
aero test        # 先跑测试
aero run         # 再运行
aero build       # 最后出 exe
```

写代码的顺序建议：函数 → 测试 → 运行 → 构建。

## 练习

1. `aero new` 建一个包，改 `main.aero` 实现一个 `is_leap_year(y) -> i64`（闰年判断），写两个 `test_` 函数测 2024（是）和 2023（不是），跑 `aero test`。
2. 建一个库包和一个主包，库提供 `fact`，主包调用并打印 `fact(6)`。
3. 在库包里放一行 `print(1);`，跑 `aero build`，读错误信息，理解"库包顶层不能有语句"。
