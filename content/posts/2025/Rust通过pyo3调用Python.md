+++
date = '2025-05-03T08:01:33+08:00'
title = 'Rust通过pyo3调用Python'
categories = ["编程语言"]
tags = ["Rust", "Python"]
cover = "/images/posts/2025/16x9_一位银色短发的机械风少女_Rust娘_被绿色长发的蛇形数据链缠绕.webp"
+++

我不太喜欢使用 [Python](https://www.python.org/)，因为它是动态类型的脚本语言，但是某些涉及 `AI` 的项目不得不使用 Python。

对于 Web 接口，我一直使用 [Django](https://www.djangoproject.com/) 的同步模式，性能并不是很理想。我计划后续深入了解 Django 的 `async/await` 异步模式来提升性能。

我个人更偏爱 [Rust](https://www.rust-lang.org/)，但 Rust 的生态系统还存在一些不足，比如阿里云和腾讯云居然没有提供官方的 Rust SDK！

于是我想到了让 Rust 调用 Python 的解决方案！

[`pyo3`](https://github.com/PyO3/pyo3) 这个项目可以实现 Python 和 Rust 的双向调用。虽然目前看来，使用 Python 调用 Rust 并将其封装成 Python 包的应用场景更为主流。

由于 Python 的设计理念本身就是作为一种"胶水语言"，因此 Rust 和 Python 的结合可以非常高效，尽管 Python 的 `GIL`（全局解释器锁）在一定程度上会限制性能。

我打算将一个纯 Python 项目使用 Rust 重写，而对于 AI 相关部分以及阿里云和腾讯云的 SDK 部分，则使用 `pyo3` 来调用 Python 代码。

## 环境配置

使用 [`uv`](https://github.com/astral-sh/uv) 来初始化 `.venv` 虚拟环境目录，并设置必要的环境变量：

```shell
. .venv/bin/activate
export LD_LIBRARY_PATH=$(python -c "import sysconfig; print(sysconfig.get_config_var('LIBDIR'))"):$LD_LIBRARY_PATH
```

## 多线程测试

我特意测试了多线程环境下 `pyo3` 代码的稳定性：

```rust
use pyo3::{IntoPyObjectExt, prelude::*};
use std::sync::mpsc;
use std::thread;

fn main() {
    let mut handles = Vec::with_capacity(10);
    for _ in 0..20 {
        let handle = thread::spawn(move || {
            Python::with_gil(|py| {
                let hello = PyModule::from_code(
                    py,
                    cr#"
import requests
import sys

def get_baidu():
    # URL for the request
    url = 'https://www.baidu.com'
    
    # Send GET request to Baidu
    response = requests.get(url)
    
    # Check if request was successful
    if response.status_code == 200:
        return response.text[:150] + "..."
    else:
        return None                        
    "#
                    ,
                    c"baidu.py",
                    c"baidu",
                )?;

                let response: Option<String> = hello.getattr("get_baidu")?.call0()?.extract()?;

                Ok::<_, PyErr>(())
            })
            .unwrap();
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }
}
```

## 性能考量

使用 `pyo3` 时需要注意的是，每次调用 Python 代码都需要获取 GIL（全局解释器锁），这可能会在高并发场景下成为性能瓶颈。不过对于大多数 I/O 密集型操作，如调用云服务 API，这通常不会成为主要问题。

## 结论

经过实测，这个简单的多线程 demo 运行良好，没有发现明显问题。我认为 `pyo3` 是连接 Rust 高性能与 Python 丰富生态的理想桥梁，特别适合那些需要 Python 特定库但又希望获得 Rust 安全性和性能优势的项目。

接下来，我将开始正式迁移项目代码，并在实际应用中进一步验证这种方案的可行性。
