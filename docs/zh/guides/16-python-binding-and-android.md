# 16 Python 绑定与 Android/iOS 交叉编译

Aero 不仅能编译成独立可执行文件，还能编成**动态库**给别的语言和平台调用：

- **Python 绑定**：把 Aero 函数导出成 Python C 扩展（`.pyd` / `.so`），Python 直接 `import` 调用；
- **Android / iOS 动态库**：交叉编译出 `lib*.so` / `lib*.dylib`，供移动端 App 通过 JNI / FFI 调用。

这两者的共同地基是"把 Aero 函数变成对外可见的 C ABI 符号"（`#[export]`）和"编出共享库"（`aero build --shared`）。本章从地基讲起。

## 1. 导出函数：`#[export]`

普通 Aero 函数的符号是内部的。想让它变成动态库里别人能调用的 C 函数，加 `#[export]`：

```aero
#[export]
fn add(a: i64, b: i64) -> i64 {
    return a + b;
}

#[export]
fn double(x: f64) -> f64 {
    return x * 2.0;
}
```

`#[export]` 的约束：

- 不能是泛型函数（必须单态化）；
- 不能和 `extern "C"` 混用（导出的就是"真函数定义"）；
- 参数只能是 C ABI 兼容类型：`i32` / `i64` / `f64` / `bool` / `str` / `String` / `*T`；
- 返回类型同理，外加无返回值。

## 2. 共享库：`aero build --shared`

把带 `#[export]` 的文件编成动态库：

```
aero build --shared examples/export_demo.aero
```

按目标平台产出 `.dll`（Windows）/ `.so`（Linux/Android）/ `.dylib`（macOS）。顶层 `main` 语句会保留但隐藏，不成为库的入口。

## 3. Python 绑定：`aero build --pyext`

`#[py_export]` 在 `#[export]` 基础上，额外让编译器**自动生成 CPython 胶水层**——包装函数、方法表、模块定义和 `PyInit_<模块名>` 入口全部自动生成，你一行 C 都不用写。

```aero
// py_bind.aero
#[py_export]
fn add(a: i64, b: i64) -> i64 {
    return a + b;
}

#[py_export]
fn double(x: f64) -> f64 {
    return x * 2.0;
}

#[py_export]
fn greet(name: str) -> str {
    return name;
}
```

编译：

```
aero build --pyext examples/py_bind/py_bind.aero
```

生成 `py_bind.pyd`（Windows，其他平台是 `.so`）。Python 直接导入：

```python
import py_bind
py_bind.add(40, 2)      # 42
py_bind.double(3.5)     # 7.0
py_bind.greet("aero")   # 'aero'
```

### 模块名

默认用源文件名的前缀作为模块名（`py_bind.aero` → 模块 `py_bind`）。要改名：

```
aero build --pyext --py-module mymod examples/py_bind/py_bind.aero
```

模块名必须和生成文件的名字一致（`mymod.pyd` ↔ `import mymod`）。

### Python 定位

编译要链接 Python 的导入库，编译器按顺序找：

1. `--py-home <prefix>`（Python 安装前缀）；
2. 环境变量 `PYTHON_HOME`；
3. 探测 `python` 可执行文件。

Windows 上官方 Python 自带的是 MSVC 导入库，编译器会自动用 `objdump` + `dlltool` 生成 MinGW 兼容的 `libpython3xx.a`。

### 类型转换表

| Aero 类型 | Python 类型 | 参数解析 | 返回构造 |
| --- | --- | --- | --- |
| `i64` | `int` | `l` | `PyLong_FromLongLong` |
| `f64` | `float` | `d` | `PyFloat_FromDouble` |
| `bool` | `bool` | `p` | `PyBool_FromLong` |
| `str` | `str`（UTF-8） | `s` | `PyUnicode_FromString` |
| `String` | `bytes` | `y#`（拷贝进缓冲） | `PyBytes_FromStringAndSize` |
| 无返回值 | `None` | — | `Py_None` |

`String` 就是 Python 的 `bytes`——Aero 的 `String` 是 `{ data, len, cap }` 原始字节缓冲（内嵌 `\0` 安全），天然适合二进制数据。示例：

```aero
#[py_export]
fn bytes_reverse(b: String) -> String {
    let out = String::new();
    let n = b.len();
    let mut i = n - 1;
    while (i >= 0) {
        let c = b.at(i);
        out.push(c);
        i = i - 1;
    }
    return out;
}
```

```python
import py_bind2
py_bind2.bytes_reverse(b"abc")           # b'cba'
py_bind2.bytes_reverse(b"\x00\x01\x02")  # b'\x02\x01\x00'
```

### 冒烟测试

`examples/py_bind/` 和 `examples/py_bind2/` 各带一个 `test_py.py`，编译后：

```
python test_py.py   # 打印 "py_bind smoke: ALL PASS"
```

## 4. Android 动态库：`aero build --shared --target aarch64-linux-android`

把 Aero 核心逻辑编成 Android 的 `lib*.so`，宿主 App 通过 JNI / FFI 调用。

```
aero build --shared --target aarch64-linux-android --ndk <NDK路径> examples/export_demo.aero
```

产出 `libexport_demo.so`（Android 惯例：共享库必须叫 `lib<名字>.so`）。

### NDK 定位

编译器按顺序找 NDK：

1. `--ndk <path>`；
2. 环境变量 `ANDROID_NDK_HOME`；
3. 常见安装位置（`%LOCALAPPDATA%\Android\Sdk\ndk`、`~/Android/Sdk/ndk` 等）。

NDK 根目录要求有 `toolchains/llvm/prebuilt/<宿主平台>/`，编译器用其中的 `clang` + `sysroot` 链接。如果 `ndk/` 目录下装了多个版本，自动选最新的。

### 支持的目标三元组

| 三元组 | ABI | NDK `--target` |
| --- | --- | --- |
| `aarch64-linux-android` | arm64-v8a | `aarch64-linux-android21` |
| `armv7-linux-androideabi` | armeabi-v7a | `armv7a-linux-androideabi21` |
| `x86_64-linux-android` | x86_64 | `x86_64-linux-android21` |
| `i686-linux-android` | x86 | `i686-linux-android21` |

API 级别固定为 21（Android 5.0+），编译器只调用 bionic libc，无版本门槛。

### 验证导出符号

有 NDK 时本地就能编出 `.so`；没有 NDK 时可用 CI（GitHub Actions Ubuntu 装 NDK）冒烟。用 `llvm-nm` / `readelf` 检查导出符号：

```
llvm-nm -D --defined-only libexport_demo.so   # 应看到 T add / T double
readelf -h libexport_demo.so                  # Machine: AArch64
```

## 5. iOS 动态库：`aero build --shared --target aarch64-apple-ios`

把 Aero 核心逻辑编成 iOS 的 `lib*.dylib`，宿主 App 通过 FFI / Swift/ObjC 桥接调用。

```
aero build --shared --target aarch64-apple-ios examples/export_demo.aero
```

产出 `libexport_demo.dylib`。

### 硬性前提

iOS 工具链（Xcode + `xcrun`）**只能跑在 macOS 上**，Windows 开发机无法本地验证。这条命令在两种场景下工作：

- **macOS 开发机**（装了 Xcode）：直接跑；
- **CI**：GitHub Actions `macos-latest` runner 自带 Xcode，仓库已配好 [`ios.yml`](https://github.com/SereinCin/aero-lang/blob/main/.github/workflows/ios.yml) 冒烟流水线。

在非 macOS 机器上运行会报 "cannot locate Xcode toolchain" 并返回非零。

### SDK 与架构

编译器用 `xcrun --sdk <sdk> --show-sdk-path` 定位系统库，用 `xcrun --sdk <sdk> -f clang` 定位编译器。SDK 按三元组自动选择：

| 三元组 | 平台 | SDK | `-arch` |
| --- | --- | --- | --- |
| `aarch64-apple-ios` | 真机（arm64） | `iphoneos` | `arm64` |
| `aarch64-apple-ios-sim` | Apple Silicon 模拟器 | `iphonesimulator` | `arm64` |
| `x86_64-apple-ios` | Intel 模拟器 | `iphonesimulator` | `x86_64` |

部署版本固定为 iOS 13（真机与模拟器都安全）。

### Mach-O 符号名

苹果的 Mach-O 格式给所有全局符号加 `_` 前缀。编译器内部为 Windows CRT 生成的 `_snprintf` 在交叉编译时会被自动改名为 `snprintf`，这样链接后符号恰好是 libSystem 导出的 `_snprintf`，无需手写符号别名。

### 验证导出符号

```
nm -gU libexport_demo.dylib    # 应看到 T _add / T _double
lipo -info libexport_demo.dylib  # Architecture: arm64
```

## 小结

- `#[export]` 让 Aero 函数变成 C ABI 符号，`#[py_export]` 额外自动生成 Python 胶水；
- `aero build --shared` 编动态库，`--pyext` 编 Python 扩展，`--target <android-triple>` / `--target <ios-triple>` 交叉编译；
- Python 绑定目前锁定 CPython 完整 ABI（升级 Python 需重编）；
- Android 需要安装 NDK（`--ndk` 或 `ANDROID_NDK_HOME`）；
- iOS 需要 macOS + Xcode，Windows 上只能靠 CI（macOS runner）验证。

## 练习

1. 写一个 `#[py_export] fn fib(n: i64) -> i64`，编成 `.pyd` 后 `import` 并调用 `fib(10)`。
2. 写一个接收 `String`（bytes）并返回 `i64` 校验和的函数，用 `b"hello"` 验证。
3. 有 NDK 时，把 `export_demo.aero` 编成 `aarch64-linux-android` 的 `.so`，用 `readelf` 确认 `add` 符号可见。
4. 有 macOS + Xcode 时，把 `export_demo.aero` 编成 `aarch64-apple-ios` 的 `.dylib`，用 `nm -gU` 确认 `_add` 符号可见。
