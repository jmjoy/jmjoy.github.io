---
title: "【Rust交叉编译】cross使用较低版本的glibc"
date: 2020-03-08T17:37:00+08:00
description: ""
featured_image: ""
categories: ["编程语言"]
tags: ["Rust", "错误处理"]
---

众所周知，glibc已经成为了Linux二进制程序在各种发行版之间不兼容的重要因素了，究其原因，是glibc的版本兼容性机制。比如在高版本glibc的Linux机器上编译和链接的二进制，在低版本glibc的Linux运行会报如下错误：

> /lib64/libc.so.6: version `GLIBC_2.14' not found

并且，glibc做静态链接时会出现比较奇怪的问题（nss等），所以各发行版一致不推荐glibc静态链接。那么目前比较好的方案是，需要发行的应用，在比较低版本的glibc做编译和链接。

而Rust官方提供了[cross](https://github.com/cross-rs/cross)这个工具做交叉编译的工作，而常用的taget`x86_64-unknown-linux-gnu`的glibc版本为2.15，对于某些老到掉牙的发行版来说，可能还是会有兼容性问题，所以我基于Centos6打包了一个镜像：<https://hub.docker.com/repository/docker/jmjoy/cross>，内置的glibc版本为2.12。

使用方法：

在Cross.toml中：

```toml
[target.x86_64-unknown-linux-gnu]
image = "jmjoy/cross:x86_64-linux-centos6"
```
