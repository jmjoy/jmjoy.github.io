---
title: "使用纯Rust开发PHP扩展"
date: 2019-12-22T00:05:00+08:00
description: ""
featured_image: ""
categories: ["编程语言"]
tags: ["Rust", "PHP"]
---

## 背景

最近由于工作需要使用某个开源的PHP扩展，发现扩展并不成熟，在某些情况下会产生内存段错误使php-fpm退出，从而产生502错误。联想到PHP源码和PHP扩展都是用C语言写的，虽然C语言在性能和内存精细控制等方面很强大，但是需要手动处理内存对程序员要求也很高，即使再牛叉的程序员也可能有疏忽的时候，导致内存问题。而我最近的时间在研究Rust这门新兴的现代化语言，深深被它的零开销抽象、内存安全、并发安全等理念所吸引，而且性能上可以和C/C++媲美，并且可以和C语言做FFI，所以就萌生了用纯Rust写PHP扩展的想法。

目前很多文章都提到过用Rust编写PHP扩展，但思路大致有两个：

1. 使用PHP7.4的FFI功能，用Rust编写代码生成.so文件供PHP FFI调用。这个方法受限于PHP的版本必须大于7.4，而且据说PHP FFI的性能损耗较大，特别是字符串方面。
2. 使用C语言编写PHP扩展，在某些地方比如函数逻辑等使用Rust做FFI，然后link Rust生成的静态链接库。这个方法可以使用到Rust的生态，也可以减少一部分手动管理内存的麻烦，但是还是无法避免地需要使用PHP内置的C函数/宏等。

所以并没有真正使用纯Rust写PHP扩展的方法，那么我就想尝试自己搞一套。

## 目标

使用纯Rust并且尽可能使用safe Rust写PHP扩展。

## 进展

目前基本的轮廓已经写好了，发布了0.1版本，可以看作是一个预览的版本。使用纯Rust编写是可以的，但是离尽可能使用safe Rust，保证内存安全方面还有很长的距离，基本思路是用Rust的struct, trait等去封装裸指针，封装不安全函数等等，这将涉及到API设计和实现等等繁重的工作。

仓库地址：https://github.com/jmjoy/phper 。

会PHP和Rust或者有兴趣的的朋友可以尝试使用一下。:)

## 题外话

发现phalcon搞了一门专门编译到PHP扩展的语言zephir，用于简化PHP扩展的开发，挺有意思的，对于某些PHP扩展的场景应该挺适用。仓库地址：https://github.com/phalcon/zephir 。
