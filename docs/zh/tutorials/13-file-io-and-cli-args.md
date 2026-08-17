# 13 文件 IO 与命令行参数

## 读文件

```aero
let content = read_file("data.txt");
print("%s\n", content);
```

`read_file(path)` 把整个文件读成字符串返回。**文件不存在或读取失败时，返回空字符串**（不是报错，是返回空串，注意区分）。

空文件读出来也是空串——所以"空串"可能代表"文件是空的"，也可能代表"读取失败"。要区分的话，先判断文件是否存在，或者用返回值设计约定。

## 写文件

```aero
let n = write_file("out.txt", "hello world");
print("%d\n", n);   // 11，写入的字节数
```

`write_file(path, contents)` 把内容写进文件，返回写入的字节数；失败返回 `-1`。

## 小工具：复制文件

```aero
// copy.aero：把参数1 复制到 参数2
let src = read_file(arg(1));
if (len(src) == 0) {
    print("cannot read %s\n", arg(1));
    // 这里没有 return 语句退出主程序的能力，0.1 版先继续往下走
}
let n = write_file(arg(2), src);
print("wrote %d bytes\n", n);
```

注意：`arg(1)`、`arg(2)` 是命令行参数，下一节讲。

## 命令行参数

`aero build` 出来的可执行文件，运行时可以带参数：

```
myapp.exe input.txt output.txt
```

程序里用两个内建函数拿参数：

```aero
let n = arg_count();   // 参数的个数
print("arg count: %d\n", n);

print("%s\n", arg(0));   // 程序自身路径
print("%s\n", arg(1));   // 第一个参数
print("%s\n", arg(2));   // 第二个参数
```

规则：

- `arg_count()`：参数总个数（**包含**程序名本身，等价于 C 的 `argc`）。
- `arg(i)`：第 i 个参数（从 0 开始），**越界返回空串**。
- `arg(0)` 是程序自身的路径；真正的"第一个参数"是 `arg(1)`。

## 完整例子：简易 grep

写一个"在文件里查找子串"的小工具：

```aero
// find.aero：find <文件> <子串>，输出匹配行的行号
extern "C" fn atoi(s: str) -> i32 = "atoi";

let path = arg(1);
let needle = arg(2);
let content = read_file(path);

if (str_contains(content, needle)) {
    print("found in %s\n", path);
} else {
    print("not found\n");
}
```

编译：

```
aero build find.aero
```

运行：

```
find.exe data.txt aero
```

`atoi` 是 C 标准库的字符串转整数函数，这里用不上，只是演示怎么声明。用 `arg_count` 检查参数数量是个好习惯：

```aero
if (arg_count() < 2) {
    print("usage: find <file> <substring>\n");
}
```

## JIT 下没有参数

`aero run file.aero` 时，`arg_count()` 是 0，`arg(i)` 返回空串——JIT 模式不传命令行参数。命令行参数在 `aero build` 的独立 exe 里才有意义。

## 练习

1. 写一个程序：读 `data.txt`，打印它的字符数（用 `len`）。
2. 写一个程序：接收两个命令行参数（两个数字的字符串），把它们转成整数相加，打印结果。
3. 写一个程序：读一个文件，统计里面有几行（提示：数 `\n` 的个数，可以用 `str_find` 循环找）。
4. 用 `write_file` 生成一个 100 行的文件（循环里拼字符串），再用练习 3 的程序验证行数。
