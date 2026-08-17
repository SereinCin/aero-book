# Aero — Official Documentation

> **Aero** is a compiled systems language that pairs developer ergonomics with native runtime speed. Built on an LLVM backend, it ships with a borrow checker, Arena-based region memory management, and first-class tensor/`matmul` support for AI workloads. Current version **1.1.0**.

This repository hosts the official English documentation for the Aero programming language.

## Contents (`en/`)

- [Guide](en/guide.md) — environment setup, installation verification, and CLI basics
- [API Reference](en/api-reference.md) — language features, operator precedence, and the full CLI command reference
- [Cookbook](en/cookbook.md) — recipes spanning the 8 pure crates and 4 FFI-bound crates (`http`, `crypto`, `regex`, `net`)

## Chinese Docs (`zh/`)

- [指南](zh/指南.md) — 环境搭建、安装验证与 CLI 基本用法
- [API 参考](zh/API参考.md) — 语言特性、运算符优先级与完整 CLI 命令
- [Cookbook](zh/Cookbook.md) — 覆盖 8 个纯 crate 与 4 个 FFI 绑定 crate 的实例手册

## Related Repositories

- [aero-lang](https://github.com/SereinCin/aero-lang) — the Aero compiler
- [aero-lang-vscode](https://github.com/SereinCin/aero-lang-vscode) — VS Code extension (syntax highlighting + LSP client)

## License

Documentation is released under the MIT License. The Aero compiler itself is Copyright (c) 2026 Serein.