---
title: "【Rust错误处理】使用`thiserror`+`anyhow`来优雅便捷地处理错误"
date: 2020-03-08T17:37:00+08:00
description: ""
featured_image: ""
categories: ["编程语言"]
tags: ["Rust", "错误处理"]
---

错误处理，一直是编程语言的一个设计难点，各种编程语言的错误处理机制都不尽相同，并且各有优劣，Rust也不例外。

Rust使用`Result`这个enum来代表正确或错误的情况，思路来源于`Haskell`和`OCaml`等函数式语言，可以说是十分的优雅，特别体现在对`Option`和`Iterator`的交互上，但是对于新人入门来说需要一定的学习成本。

从我自身的使用情况来说，我总结了一下目前Rust（版本1.41.1）的错误处理的几个缺点：

1. 多种类型的`Error`在同一个函数中需要处理时，必须要有一个共同的可以`Into`的`Error`类型作为函数返回值，比如下面的代码：

    ```rust
    fn foo() -> Result<(), FooError> {
        let one: Result<(), OneError> = fn_one();
        one?;

        let two: Result<(), TwoError> = fn_two();
        two?;

        Ok(())
    }
    ```

    那么`FooError`必须有实现`From<OneError>`和`From<TwoError>`，惯常的做法是写一个enum将所有的错误都封装起来，并且实现`Display`、`Debug`、`Error`还有`From<T>`这几个trait，手写的话比较枯燥乏味。

2. `Error` trait的`backtrace`还未稳定，虽然说`backtrace`对性能会有损耗，但是没有`backtrace`在遇到错误抛出的时候，如果错误没有一些明确的信息，就很难定位到是哪里的代码抛的这个错误，特别是对于第三方库抛的错误，对于开发调试和生产排查问题都造成了不便，在[Fix the Error trait](https://github.com/rust-lang/rust/issues/53487)这个RFC里面有提出。

----

针对第一点情况，Rust界的大佬`dtolnay`设计了两个crate：[thiserror](https://github.com/dtolnay/thiserror)和[anyhow](https://github.com/dtolnay/anyhow)。在这之前已经有很多提升错误处理的库了，比如`failure`、`error-chain`和`quickerror`等等，这些库我都使用过，而且这些库都有一定程度的互相借鉴，但最后最终还是解决`thiserror`+`anyhow`是最易用和实用的。

> 注意事项：`thiserror`是给lib使用的，而`anyhow`是给bin程序使用，当然bin程序可以使用`thiser`+`anyhow`，但是切记`anyhow`不要在lib里面使用。

这两个crate的使用方法在README中也说得很清楚明白了，所以下面只是简单得阐述一下：

### thiserror

*复制粘贴的`thiserror`示例：*

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum DataStoreError {
    #[error("data store disconnected")]
    Disconnect(#[from] io::Error),
    #[error("the data for key `{0}` is not available")]
    Redaction(String),
    #[error("invalid header (expected {expected:?}, found {found:?})")]
    InvalidHeader {
        expected: String,
        found: String,
    },
    #[error("unknown data store error")]
    Unknown,
}
```

这个其实就是用enum封装的思路，但是因为是用`derive macro`来自动生成的，节省了很多的时间和敲键成本，也没有那么无聊。

### anyhow

*复制粘贴的`anyhow`示例：*

```rust
use anyhow::Result;

fn get_cluster_info() -> Result<ClusterMap> {
    let config = std::fs::read_to_string("cluster.json")?;
    let map: ClusterMap = serde_json::from_str(&config)?;
    Ok(map)
}
```

这个就更加简洁了，直接将`anyhow::Result`作为返回值就可以了，因为它直接实现了`From<E> where E: Error`，所以就不需要再维护一个enum类型了，但是注意只有在bin程序里面才可以这么搞，因为在lib里面这么做的话，别人在使用的时候想特别处理某种情况的错误，只能通过`downcast`来处理，这个设计是很不友好的。

在bin程序里面如果有一点需要特别处理的错误，比如后端接口中的用户权限失败这种错误，是需要明确标记出来（区分错误码等），让前端弹出登录框这种需求的，这里就可以将`thiserror`和`anyhow::Result`相结合使用了：

*复制粘贴的`thiserror`示例：*

```rust
#[derive(Error, Debug)]
pub enum MyError {
    ...

    #[error(transparent)]
    Other(#[from] anyhow::Error),  // source and Display delegate to anyhow::Error
}

```

对于需要特别对待的错误，则手工去用`thiserror`维护，对于无关重要的错误，则统一用`Other`去包装就可以了。
