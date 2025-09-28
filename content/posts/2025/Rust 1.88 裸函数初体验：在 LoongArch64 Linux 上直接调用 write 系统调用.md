+++
date = '2025-09-28T20:03:00+08:00'
title = 'Rust 1.88 裸函数初体验：在 LoongArch64 Linux 上直接调用 write 系统调用'
categories = ["编程语言"]
tags = ["Rust", "loongarch64", "Linux", "裸函数", "汇编"]
+++

最近试了下 Rust 1.88 新增的「裸函数」（naked function），在 loongarch64 的 Linux 上直接用内联汇编做了一次 `write` 系统调用，代码很短，但很有意思。

## 代码与效果

下面是我在 loongarch64 上运行的最小示例，直接通过 `syscall` 写到标准输出：

```rust
use core::{arch::naked_asm, ffi::c_char};

fn main() {
    let message = c"Hello Rust!\n";
    let rc = write_stdout_naked(message.as_ptr(), message.to_bytes().len());
    if rc < 0 {
        panic!("write syscall failed with {}", rc);
    }
}

#[unsafe(naked)]
extern "C" fn write_stdout_naked(buf: *const c_char, len: usize) -> isize {
    naked_asm! {
        "addi.d $a2, $a1, 0",
        "addi.d $a1, $a0, 0",
        "li.d $a0, 1",
        "li.d $a7, 64",
        "syscall 0",
        "ret",
    }
}
```

运行结果如预期：

```text
Hello Rust!
```

## 发生了什么？

- `#[unsafe(naked)]` 标注让函数成为「裸函数」：
  - 编译器不会生成常规的函数前序/后序（prologue/epilogue），栈与寄存器全由你自己负责；
  - 函数体里只能是内联汇编，不能使用局部变量、Rust 表达式等。
- loongarch64 的 Linux 系统调用约定：
  - 参数使用 `$a0..$a7`；系统调用号放在 `$a7`；返回值在 `$a0`；
  - 这里我们把传入的 `buf`、`len` 分别搬到 `$a1`、`$a2`，把 `$a0` 设为 `1`（stdout 的 fd），再把 `$a7` 设为 `64`（`write` 的 syscall 号），然后执行 `syscall 0`；
  - 系统调用返回后，`$a0` 就是返回值（写入的字节数或负的错误码），`ret` 直接把它当作 Rust 函数返回值带回去。

对应地，每条指令的作用大致是：

- `addi.d $a2, $a1, 0`：`len -> a2`（把第二个参数挪到 a2）
- `addi.d $a1, $a0, 0`：`buf -> a1`（把第一个参数挪到 a1）
- `li.d $a0, 1`：标准输出 fd
- `li.d $a7, 64`：`write` 的系统调用号（loongarch64 上为 64）
- `syscall 0`：触发内核入口
- `ret`：返回到 Rust（返回值在 a0）

## 使用裸函数的注意事项

- 裸函数完全跳过了 ABI 前后序，务必确保：
  - 不使用栈或在使用前自行建立有效的栈；
  - 遵守目标架构的调用约定，必要时保存/恢复需要保持的寄存器；
  - 用 `extern "C"` 明确 ABI，避免编译器调整参数传递方式。
- 错误返回值：Linux 约定系统调用失败返回负的错误码（`-errno`）。上面的代码用 `rc < 0` 做了简单检查。
- 可移植性：该示例只适用于 loongarch64。若要跨平台，建议加上 `#[cfg(target_arch = "loongarch64")]` 条件编译或提供常规的 `std::io` 备用实现。

## 小结

Rust 1.88 的裸函数让我们可以在不引入函数栈框架的情况下，直接用最纯粹的汇编与平台交互——比如像上面这样手工发起一次 `write` 系统调用。对需要极致启动开销、OS 引导、嵌入式或特殊 ABI/调用序需求的场景尤其有用。即便是一个“Hello Rust!”的小实验，也足以感受到它带来的掌控感与乐趣。
