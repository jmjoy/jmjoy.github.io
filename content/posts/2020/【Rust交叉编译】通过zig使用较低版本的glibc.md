---
title: "【Rust交叉编译】通过zig使用较低版本的glibc"
date: 2020-03-08T17:37:00+08:00
description: ""
featured_image: ""
categories: ["编程语言"]
tags: ["Rust", "错误处理"]
---

在前一篇关于Rust交叉编译的文章中（[【Rust交叉编译】cross使用较低版本的glibc](https://www.jianshu.com/p/bb12797ee48a)），我提到了使用`cross`来方便地将Rust源代码编译产物链接到低版本的glibc，现在有了更加方便的方法，那就是[cargo-zigbuid](https://github.com/rust-cross/cargo-zigbuild)。

[Zig](https://ziglang.org/) 是一种通用编程语言和工具链，用于维护健壮、最佳和可重用的软件，而[交叉编译](https://ziglang.org/zh/learn/overview/#%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91%E7%9A%84%E4%B8%80%E6%B5%81%E6%94%AF%E6%8C%81)是Zig的一个卖点。

虽然我们没真正用上Zig语言，但是我们可以使用Zig的编译链，来增强Rust的交叉编译（心疼Zig为Rust作嫁衣）。
