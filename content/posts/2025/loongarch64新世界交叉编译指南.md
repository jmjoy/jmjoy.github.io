+++
date = '2025-01-16T15:36:21+08:00'
title = 'Loongarch64新世界交叉编译指南'
categories = ["编程技术"]
tags = ["loongarch", "交叉编译"]
+++

## C语言项目

使用zig cc来交叉编译：

1. 下载zig，选择master版本：

   <https://ziglang.org/download/>

2. 假设需要编译mian.c，命令如下：

   ```shell
   zig cc -target loongarch64-linux-gnu.2.36 -o main main.c
   ```

   其中2.36是glibc版本，这个根据新世界系统的glibc版本自行选择，也可以使用musl静态或者动态链接，target改成`loongarch64-linux-musl`即可。

## Rust项目

1. 同样使用zig cc来交叉编译，首先安装cargo-zigbuild：

   ```shell
   cargo install cargo-zigbuild
   ```

2. 然后在项目中运行：

   ```shell
   cargo zigbuild --target loongarch64-unknown-linux-gnu.2.36 --release
   ```

然而，由于[Rust 1.84版本已默认启用LoongArch架构的LSX目标特性](https://releases.rs/docs/1.84.0/#compatibility-notes)，因此采用上述方法使用最新版Rust为不支持LSX的2K0300等SoC进行LoongArch程序交叉编译将无法实现。

目前的解决方式是使用nightly channel的Rust：

```shell
RUSTFLAGS="-Ctarget-feature=-lsx" cargo +nightly zigbuild -Zbuild-std --target loongarch64-unknown-linux-gnu.2.36 --release
```

> 参考：[How to remove default target feature `lsx` for `loongarch64`?](https://users.rust-lang.org/t/how-to-remove-default-target-feature-lsx-for-loongarch64/124486?u=jmjoy)

## Go项目

1. Go交叉编译比较简单：

   ```shell
   GOOS=linux GOARCH=loong64 go build
   ```
