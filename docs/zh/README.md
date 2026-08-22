# Aero 程序设计

一本写给人的 Aero 语言入门书。目标：看完这本书，你能用 Aero 写出完整的、能跑的、编译成独立可执行文件的程序。

对应 Aero 版本：**1.2.0**（2026）。

## 目录

### 教程

- [00 前言](tutorials/00-preface.md)
- [01 环境与第一个程序](tutorials/01-environment-and-first-program.md)
- [02 变量与类型](tutorials/02-variables-and-types.md)
- [03 运算符与表达式](tutorials/03-operators-and-expressions.md)
- [04 流程控制](tutorials/04-flow-control.md)
- [05 函数](tutorials/05-functions.md)
- [06 泛型](tutorials/06-generics.md)
- [07 数组与元组](tutorials/07-arrays-and-tuples.md)
- [08 字符串](tutorials/08-strings.md)
- [09 引用与借用](tutorials/09-references-and-borrowing.md)
- [10 Arena 内存管理](tutorials/10-arena-memory-management.md)
- [11 Tensor 与矩阵](tutorials/11-tensor-and-matrix.md)
- [12 FFI：调用 C](tutorials/12-ffi-calling-c.md)
- [13 文件 IO 与命令行参数](tutorials/13-file-io-and-cli-args.md)

### 指南

- [14 包管理器与测试](guides/14-package-manager-and-testing.md)
- [15 编译原理速览](guides/15-compiler-overview.md)
- [16 Python 绑定与 Android/iOS 交叉编译](guides/16-python-binding-and-android.md)
- [17 C++ 绑定](guides/17-cpp-binding.md)

### 附录

- [附录 B：常见错误与解决](guides/appendix-b-common-errors.md)
- [附录 C：与 C 和 Rust 对照](guides/appendix-c-comparison-with-c-and-rust.md)

### API 参考

- [内建函数速查](api/builtin-functions-reference.md)

## 使用说明

- 全部示例代码都经过验证，可以直接跑。
- 建议边读边敲，不要只看。
- 每章末尾有练习，做完再进下一章。
- 本书按 Windows 环境讲解（Aero 目前主要跑在 Windows），macOS/Linux 用户只需改安装步骤，语法完全一样。
