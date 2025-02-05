+++
date = '2025-01-16T15:36:21+08:00'
title = 'Loongarch64新世界交叉编译指南'
tags = ["loongarch"]
+++

## C语言项目

使用zig cc来交叉编译：

1. 下载zig，选择master版本：

   <https://ziglang.org/download/>

2. 假设需要编译mian.c，命令如下：

   ```shell
   zig cc -target loongarch64-linux-gnu.2.36 -o main main.c
   ```

   其中2.36是glibc版本，这个根据新世界系统的glibc版本自行选择。

## Rust项目

1. 同样使用zig cc来交叉编译，首先安装cargo-zigbuild：

   ```shell
   cargo install cargo-zigbuild
   ```

2. 然后在项目中运行：

   ```shell
   cargo zigbuild --target loongarch64-unknown-linux-gnu.2.36 --release
   ```

## Go项目

1. Go交叉编译比较简单：

   ```shell
   GOOS=linux GOARCH=loong64 go build
   ```
