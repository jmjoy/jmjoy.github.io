+++
date = '2025-07-30T19:56:11+08:00'
title = '探索Bevy的动态链接：一次加速Rust编译速度的尝试'
categories = ["编程语言"]
tags = ["Rust"]
cover = "/images/posts/2025/探索Bevy的动态链接：一次加速Rust编译速度的尝试.webp"
+++

最近在深入研究 [Bevy](https.bevyengine.org/) 游戏引擎时，我注意到一个非常有意思的特性：`dynamic_linking`。
启用这个特性后，可以显著加快开发阶段的编译速度。这激起了我的好奇心：它究竟是如何工作的？
为了弄清底层原理，我决定自己动手创建一个最小化的demo项目来模拟这个功能。

## 创建Demo项目

我的目标是构建一个包含两个crate的Cargo Workspace：

1. `foo`: 一个动态链接库（dylib）。
2. `bar`: 一个依赖于 `foo` 的可执行文件。

### 步骤1: 初始化项目结构

首先，我创建了一个新的Cargo Workspace：

```bash
mkdir fast-compile
cd fast-compile
```

然后，在 `fast-compile` 目录下创建两个子crate `foo` 和 `bar`：

```bash
cargo new foo --lib
cargo new bar --bin
```

最后，我将 `foo` 和 `bar` 添加到根目录的 `Cargo.toml` 的 `[workspace]`成员中。

```toml
# fast-compile/Cargo.toml
[workspace]
resolver = "3"
members = [ "bar","foo"]
```

### 步骤2: 编写代码

#### `foo` Crate: 动态链接库

`foo` 是我们的动态库。关键的配置在于它的 `Cargo.toml` 文件。通过指定 `crate-type` 为 `["dylib"]`，我们告诉Rust编译器将这个Crate编译成一个动态链接库。

**`foo/Cargo.toml`**

```toml
[package]
name = "foo"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["dylib"]
```

它的Rust代码非常简单，只包含一个打印消息的函数。

**`foo/src/lib.rs`**

```rust
pub fn hello() {
    println!("hello from dylib!");
}
```

#### `bar` Crate: 可执行文件

`bar` 是我们的主程序，它会调用 `foo` 库中的函数。首先，在 `bar/Cargo.toml` 中添加对 `foo` 的依赖。

**`bar/Cargo.toml`**

```toml
[package]
name = "bar"
version = "0.1.0"
edition = "2021"

[dependencies]
foo = { version = "0.1.0", path = "../foo" }
```

然后，在 `main.rs` 中调用 `foo::hello()`。

**`bar/src/main.rs`**

```rust
use foo;

fn main() {
    foo::hello();
}
```

## 编译：我走过的弯路

项目设置好后，我习惯性地在工作区根目录运行了 `cargo build`。然而，我遇到了一个意想不到的错误：

```text
error: cannot satisfy dependencies so `foo` only shows up once
```

这个错误让我困惑了一段时间。经过一番研究，我意识到问题出在Cargo处理工作区和动态库的方式上。当在工作区根目录运行时，Cargo试图构建所有的成员，但它无法正确处理 `bar` 对 `foo` dylib的依赖关系。

## 正确的编译方式

正确的解决方法是明确告诉Cargo我们要构建哪一个包。通过使用 `-p` (或 `--package`) 参数，我们可以指定目标包：

```bash
cargo build -p bar
```

这个命令成功了！它首先编译了 `foo` 作为一个动态库，然后编译了 `bar` 并将其链接到 `foo`。运行生成的可执行文件，我们看到了预期的输出：

```bash
cargo run -p bar
# 输出: hello from dylib!
```

## 结论

这次小实验让我深刻理解了Bevy `dynamic_linking` 特性背后的原理。核心就是将一部分代码（特别是那些不经常变动的）编译成动态库。这样，在开发过程中，当我们修改主程序时，编译器只需要重新编译主程序本身，而不需要重新编译整个依赖树，从而大大缩短了编译时间。

关键的教训是：当在Cargo Workspace中使用动态库时，直接在顶层运行 `cargo build` 可能会导致依赖解析错误。正确的做法是使用 `cargo build -p <your_executable_crate>` 来指定你要构建的目标。

希望这次分享能帮助到同样对Rust编译速度优化感兴趣的开发者！
