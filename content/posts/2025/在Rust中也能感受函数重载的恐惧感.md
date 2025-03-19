+++
date = '2025-03-19T13:17:18+08:00'
title = '在Rust中也能感受函数重载的恐惧感'
categories = ["编程语言"]
tags = ["Rust"]
+++

最近在设计Rust库API的时候遇到了一个类似`函数重载`带来的恐惧感，特意来分享一下。

## 背景

为了使得我的API使用起来很灵活，我设计了这么一个struct和trait，函数参数类型标记为`impl Trait`，可以传递多种类型的参数，达到一种`函数重载`的效果。

## 代码

函数原型：

```rust
fn post_prompt(prompt: impl TryIntoPrompt) {
    // ...
}
```

定义struct和trait：

```rust
pub struct Prompt(pub(crate) serde_json::Value);

pub trait TryIntoPrompt {
    fn try_into_prompt(self) -> ClientResult<Prompt>;
}
```

实现trait：

```rust
impl TryIntoPrompt for &str {
    fn try_into_prompt(self) -> ClientResult<Prompt> {
        Ok(Prompt(serde_json::from_str(self)?))
    }
}

impl<T: Serialize> TryIntoPrompt for &T {
    fn try_into_prompt(self) -> ClientResult<Prompt> {
        Ok(Prompt(serde_json::to_value(self)?))
    }
}
```

## 问题

编译没有报错，感觉设计得很完美，但是`cargo test`却失败了，我还以为是啥偶发性原因，重跑了几遍，都是一样的结果。

测试代码：

```rust
let prompt = "...".to_string();
post_prompt(&prompt);
```

## 原因

经过我的一番研究，我发现跟我幻想中的dispatch发生了冲突。我本来是期望上述代码能分发到`impl TryIntoPrompt for &str`，结果却分发到了`impl<T: Serialize> TryIntoPrompt for &T`，原来prompt的类型不是`&str`而是`&String`。

首先[`serde::Serialize`](https://docs.rs/serde/latest/serde/trait.Serialize.html)分别为`str`和`String`做了实现：

```rust
impl Serialize for String;

impl Serialize for str;
```

按理来说，`impl TryIntoPrompt for &str`跟`impl<T: Serialize> TryIntoPrompt for &T`应该是冲突的。为啥会编译通过？

因为我试图加上`impl TryIntoPrompt for &String`，如无意外的编译报错了。

## 解惑

问了一下ChatGPT，我才醒悟过来：

> **为什么 `impl TryIntoPrompt for &str` 和 `impl<T: Serialize> TryIntoPrompt for &T` 不冲突？**
>
> **1. `impl TryIntoPrompt for &str`**
> 它是一个具体实现，直接适用于 `&str` 类型。
>
> **2. `impl<T: Serialize> TryIntoPrompt for &T`**
> 这是一个泛型实现，适用于任何实现了 `Serialize` 的 `&T`。
>
> 你可能认为 `str` 符合 `T: Serialize`，所以 `&str` 也符合 `&T: Serialize`，从而产生了冲突。但这里有一个关键点：
>
> > **`T` 不能是 `str`**。
>
> 因为 `str` 是**一个动态大小类型（DST）**，不能直接用作泛型参数 `T`，除非用 `T: ?Sized` 这样显式放宽约束。而 `impl<T: Serialize> TryIntoPrompt for &T` 中的 `T` **默认是 `Sized` 的**，因此 `T` 不能是 `str`，导致 `&T` 不能是 `&str`。
>
> ---
>
> **如何制造冲突？**
> 如果你允许 `T` 是 `?Sized`，那么 `T` 就可以是 `str` 了，这样就会产生冲突。例如：
>
> ```rust
> impl<T: Serialize + ?Sized> TryIntoPrompt for &T {
>     fn try_into_prompt(self) -> ClientResult<Prompt> {
>         Ok(Prompt(serde_json::to_value(self)?))
>     }
> }
> ```
>
> 这时候，`T = str` 也是合法的，那 `&T = &str`，就和 `impl TryIntoPrompt for &str` 直接冲突了。
