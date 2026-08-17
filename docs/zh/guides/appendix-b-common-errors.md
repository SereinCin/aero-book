# 17 附录 B：常见错误与解决

这里收集初学者最常撞上的错误，每条给：错误长什么样、为什么会发生、怎么改。

## 漏分号

```
line 1 col 9 [语法分析] expected `;`, found ...
```

**原因**：语句结尾没写 `;`。**解决**：去上一行结尾补分号。Aero 的语句（`let`、赋值、`print`、`return`）都需要分号。

## 类型不匹配（隐式窄化）

```
[类型] type mismatch: variable `y` expected `i32`, got `i64`
```

**原因**：把 `i64` 的值塞给 `i32` 的变量。Aero 拒绝隐式窄化。**解决**：要么两边都用 `i64`，要么声明时用 `i64`。整数字面量可以适配任意整数类型，但变量不行：

```aero
let x = 10000000000;      // i64
// let y: i32 = x;        // 错误
let y: i32 = 5;           // 字面量可以
```

## 未定义变量 / 未定义函数

```
[名称解析] undefined variable `x`
[名称解析] undefined function `foo`
```

**原因**：名字拼错了，或者变量在块外使用（作用域问题）。**解决**：检查拼写；确认变量声明在使用它的块里（或更外层）。

## 变量重复声明

```
[名称解析] variable `x` is already declared in this scope
```

**原因**：同一个作用域里 `let` 了两次同名变量。**解决**：换个名字，或把声明放外层。注意 Aero 不允许遮蔽（shadowing）。

## 借用冲突

```
[借用检查] cannot mutably borrow: the variable is already borrowed
[借用检查] cannot immutably borrow: the variable is already mutably borrowed
[借用检查] variable cannot be assigned while borrowed
```

**原因**：违反了第 9 章的借用规则——可变借用不排他、借用期间写源变量。**解决**：让冲突的借用提前结束（利用 NLL：最后使用后借用自动结束），或调整代码顺序。

## if / while 条件不是布尔

```
[类型] `if condition` requires a boolean type, got `i64`
```

**原因**：写了 `if (x)`，Aero 不允许整数当条件。**解决**：写 `if (x != 0)`。

## 函数定义嵌套

```
[名称解析] function definitions cannot be nested inside function bodies
```

**原因**：在函数体里写了 `fn`。**解决**：函数都放顶层。

## 元组索引问题

```
[类型] tuple index must be an integer constant within range
```

**原因**：元组索引用了变量，或超出元组长度。**解决**：元组索引必须写死常量。要动态索引，用数组。

## 字符串忘了释放

不是编译错误，是运行时内存慢慢涨。**症状**：程序跑得越久内存越高。

**原因**：`int_to_str`、`substr`、运行时拼接的结果没 `str_free`。**解决**：用完释放（见第 8 章"谁负责释放"）。

## Arena 越界

**症状**：程序运行时突然终止（`abort`）。

**原因**：`alloc` 要的内存超出 `arena(N)` 的容量。**解决**：把 `N` 调大，或检查分配逻辑。

## 库包顶层有语句

```
[包管理器] library packages must not have top-level statements
```

**原因**：依赖库的 `main.aero` 里写了 `print(...)` 这类可执行语句。**解决**：库包只放函数定义。

## extern 函数类型不允许

```
[类型] extern "C" parameter `x` type `bool` is not C ABI compatible
```

**原因**：`bool`（或数组、元组）不能穿过 C ABI。**解决**：参数用 `i32`/`i64`/`str`/`*T`，返回用 `i32`/`i64`/`*T`/void。

## matmul 维度不匹配

```
[类型] matmul dimension mismatch: 2x3 and 2x3 cannot be multiplied
```

**原因**：左矩阵列数 ≠ 右矩阵行数。**解决**：检查两个矩阵的形状（`tensor(行, 列)`）。

## 泛型参数名冲突

```
[类型] generic type parameter `i64` collides with a builtin type name
```

**原因**：用 `i64`/`i32`/`bool`/`str` 当泛型参数名。**解决**：换 `T`、`U` 这类名字。

## 原生数组越界

编译期不一定能查到（字面量长度的常量情况会查），运行时越界是未定义行为。**解决**：自己保证下标在 `[0, len)` 内。

> **Vec 动态数组**：`get(i)` 越界返回 0，`set(i, v)` 越界忽略写入，不会崩溃或损坏内存。

## 除零

Aero 会检查除数为 0 的情况，除零结果返回 0，不会崩溃。但依赖"除零返回 0"来写逻辑是不好的习惯，建议自己确保除数不为 0。

## 记不住怎么办

- 看错误第一行：行列号 + 阶段名。
- 阶段名告诉你去哪找：`[语法分析]` 看语法、`[类型]` 看类型、`[借用检查]` 看借用、`[代码生成]` 少见。
- 消息是英文的，但格式固定。翻到本章对应条目对照。
