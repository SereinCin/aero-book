# Aero — 官方文档

> **Aero** 是一门兼顾开发体验与运行速度的编译型系统语言，基于 LLVM 后端，具备借用检查、Arena 区域内存管理与原生张量/矩阵（matmul）支持。当前版本 **1.1.0**。

本仓库存放 Aero 编程语言的官方中文与英文文档。

## 文档结构

- `zh/` 中文文档
  - [指南](zh/指南.md) — 环境搭建、安装验证与 CLI 基本用法
  - [API 参考](zh/API参考.md) — 语言特性、运算符优先级与完整 CLI 命令
  - [Cookbook](zh/Cookbook.md) — 覆盖 8 个纯 crate 与 4 个 FFI 绑定 crate 的实例手册
- `en/` 英文文档
  - [Guide](en/guide.md)
  - [API Reference](en/api-reference.md)
  - [Cookbook](en/cookbook.md)

## 相关仓库

- [aero-lang](https://github.com/SereinCin/aero-lang) — Aero 编译器源码
- [aero-lang-vscode](https://github.com/SereinCin/aero-lang-vscode) — VS Code 插件（语法高亮 + LSP 客户端）

## 许可

文档基于 MIT 许可发布。Aero 编译器本体版权 Copyright (c) 2026 Serein。