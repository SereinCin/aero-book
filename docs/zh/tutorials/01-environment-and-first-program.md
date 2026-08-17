# 01 环境与第一个程序

## 需要装什么

Aero 的编译器本身是用 Rust 写的，代码生成走 LLVM，AOT 链接用 MinGW 的 gcc。所以要三样东西：

| 组件 | 用途 | 版本要求 |
| --- | --- | --- |
| Rust 工具链（cargo/rustc） | 编译 Aero 编译器本身 | 1.97+ |
| LLVM 22 | 生成机器码 | 22.x（llvm-sys 221） |
| MinGW UCRT64 gcc | 把目标文件链接成 .exe | 任意较新版本 |

如果你只想用 `aero run`（JIT 方式跑程序），gcc 可以不装；但要用 `aero build` 产出独立可执行文件，gcc 是必需的。

## 安装步骤（Windows）

**1. 装 Rust**

去 https://rustup.rs 下载 rustup-init.exe，一路默认。装完打开新的终端，验证：

```
cargo --version
rustc --version
```

**2. 装 LLVM**

Aero 用的是 LLVM 22。装完记下它的安装目录，比如 `D:\Scripts\LLVM\clang+llvm-22.1.8-x86_64-pc-windows-msvc`。然后设置环境变量：

```
set LLVM_SYS_221_PREFIX=D:\Scripts\LLVM\clang+llvm-22.1.8-x86_64-pc-windows-msvc
```

这个环境变量是给 Rust 的 llvm-sys 库找 LLVM 用的，名字里的 221 对应 LLVM 22.1，别改。

**3. 装 gcc**

装 MSYS2，然后在 MSYS2 终端里：

```
pacman -S mingw-w64-ucrt-x86_64-gcc
```

把 `C:\msys64\ucrt64\bin` 加进 PATH。

## 构建 Aero 编译器

拿到 Aero 的源码目录后，运行项目自带的构建脚本：

```
scripts\build.bat
```

脚本会编译整个工作区，最后在 `target\debug\aero.exe` 产出编译器本体。构建要几分钟，等它跑完。

验证：

```
target\debug\aero.exe
```

没有参数时它会打印用法：

```
Usage:
  aero run <file.aero | package dir>   compile and run
  aero build [file.aero | dir]         compile to a standalone executable (AOT)
  aero new <name>                      create a new package skeleton
  aero test [file.aero]                run tests (default: all in tests/)
```

方便起见，建议把 `target\debug` 目录加入系统 PATH 环境变量，这样可以在任何目录直接敲 `aero` 命令：

```
setx PATH "%PATH%;项目目录\target\debug"
```

## 第一个程序

建一个文件 `hello.aero`：

```aero
// 第一个 Aero 程序
print("Hello, Aero!\n");
```

跑它：

```
aero run hello.aero
```

终端输出：

```
Hello, Aero!
```

恭喜，你的第一个 Aero 程序跑起来了。

## 这条 print 到底干了什么

`print` 是 Aero 的内建函数，它最终调用的是 C 标准库的 `printf`。所以：

- `print("Hello, Aero!\n")` 输出一行文字，`\n` 是换行符；
- 字符串里没有 `\n` 就不会换行；
- 你可以像 C 一样用格式串，比如 `print("%d\n", 42)`（第 3 章细讲）。

写几个试试：

```aero
print(1 + 1);            // 输出 2
print(2 + 3 * 4);        // 输出 14（乘除优先）
print("两行\n文字\n");   // 输出两行
```

## 编译成独立可执行文件

`aero run` 是 JIT 方式：编译完直接在内存里执行，不落地文件。好处是快，坏处是每次都要带编译器。

要得到一个能独立分发的 .exe，用：

```
aero build hello.aero
```

这会在 `hello.aero` 旁边生成 `hello.exe`。直接双击或在终端运行：

```
hello.exe
```

输出一样是 `Hello, Aero!`。这个 exe 不依赖 Aero 编译器，也不依赖 LLVM 的动态库——它是完整的原生程序。把它拷到别的 Windows 机器上，只要目标机有正常的 UCRT 运行库就能跑。

## 错误消息长什么样

Aero 的编译错误是英文的，格式统一：

```
错误: line 3 col 5 [语法分析] expected `;`, found `}`
```

从左到右：出错的行列、出错阶段（词法/语法/类型/借用/代码生成）、具体消息。看到错误先看行列，再看消息，最后顺着行列去代码里找。

## 练习

1. 改 `hello.aero`，让它输出三行：你的名字、今天日期、一句口号。
2. 用 `aero build` 编译，把生成的 exe 改名，拷到另一个目录运行，确认它不依赖原目录。
3. 故意删掉一个分号再运行，读一读报错信息，看看行列号指得准不准。
