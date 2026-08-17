# 04 流程控制

## if / else

```aero
let x = 10;
if (x > 0) {
    print("positive\n");
} else {
    print("non-positive\n");
}
```

条件写在括号里，必须是 `bool` 表达式。`else` 可以省略：

```aero
let x = 10;
if (x > 100) {
    print("big\n");
}
// x 不大于 100 时什么也不做
```

条件里 `x` 这种数字不能直接当条件用。`if (x)` 在 C 里合法（非零即真），在 Aero 里是编译错误——你必须写 `if (x != 0)`。这是刻意设计：类型严格，不搞隐式转换。

## if / else if 链

```aero
let score = 85;
if (score >= 90) {
    print("A\n");
} else if (score >= 80) {
    print("B\n");
} else if (score >= 70) {
    print("C\n");
} else {
    print("D\n");
}
```

从上往下找第一个条件为真的分支执行，其余跳过。

## while 循环

```aero
let i = 0;
while (i < 5) {
    print("%d\n", i);
    i = i + 1;
}
// 输出 0 1 2 3 4
```

循环体里记得改条件变量，否则死循环。按 `Ctrl+C` 能中断一个死循环程序。

while 的条件和 if 一样必须是 bool：

```aero
let i = 10;
while (i > 0) {
    i = i - 1;
}
print("done\n");
```

## 组合使用

```aero
// 打印 1 到 100 之间所有能被 3 整除的数
let i = 1;
while (i <= 100) {
    if (i % 3 == 0) {
        print("%d\n", i);
    }
    i = i + 1;
}
```

`%` 是取模（余数）。`i % 3 == 0` 表示 i 能被 3 整除。

## 当前没有的：break 和 continue

Aero 0.1 的 while **没有** `break` 和 `continue`。想提前退出循环，只能用条件控制：

```aero
// 模拟 break：找到第一个大于 100 的 7 的倍数
let i = 1;
let found = 0;   // 0 = 假，1 = 真，Aero 0.1 用整数模拟
while (i <= 1000 && found == 0) {
    if (i % 7 == 0) {
        print("%d\n", i);
        found = 1;
    }
    i = i + 1;
}
```

这是当前版本的已知短板，后续会补上。

## 作用域提醒

if 和 while 的 `{ }` 也是块，块内声明的变量出块即失效（第 2 章讲过）。注意循环里声明变量的代价：

```aero
let total = 0;
let i = 0;
while (i < 10) {
    let square = i * i;   // 每次循环都重新声明
    total = total + square;
    i = i + 1;
}
print(total);   // 285
```

`square` 每次循环新建，用完即弃，没问题。但如果你在循环里声明一个"要跨循环保存"的变量，它会被反复重置——这是新手最容易踩的坑之一：

```aero
let i = 0;
while (i < 5) {
    let sum = 0;          // 错！每次循环 sum 都归零
    sum = sum + i;
    i = i + 1;
}
```

`sum` 应该声明在循环外面。

## 练习

1. 打印 1 到 20 之间所有奇数。
2. 用 while 计算 1 到 100 的和，输出 5050。
3. 写一个程序：从 1 开始，每次乘 2，打印，直到超过 1000。看它打印几个数。
4. 用 if/else if 把百分制成绩映射成等级（A/B/C/D/E），打印出来。
