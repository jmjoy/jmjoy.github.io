---
title: "Let’s Encrypt证书吊销事故的思考"
date: 2020-03-28T00:11:54+08:00
description: "如何保障代码的安全性？"
featured_image: ""
categories: ["编程"]
tags: []
---

前段时间一则新闻引爆了程序界：[《一行Golang代码引发的血案——全网最详细分析2020年3月Let’s Encrypt证书吊销事故》](https://zhuanlan.zhihu.com/p/111639968)。

这个事故的原因上文已经说得很清楚了，这里摘抄一下：

> 那么这个软件到底出现了什么问题才会导致如此滑稽的故障？我翻看着Let's Encrypt最近的commit，找到了一个Pull Request：[#4690](https://github.com/letsencrypt/boulder/pull/4690/files)。看完这个Pull Request后，我马上意识到问题所在：Golang最经典的错误——循环迭代变量陷阱。
>
> ……
> 
> 作为『21世纪的C语言』，Golang同样存在这一问题：
> 
> ```go
> func main() {
>     var out []*int
>     for i := 0; i < 3; i++ {
>         out = append(out, &i)
>     }
>     fmt.Println("Values:", *out[0], *out[1], *out[2])
>     fmt.Println("Addresses:", out[0], out[1], out[2])
> }
> ```
> 输出结果为：
> ```
> Values: 3 3 3
> Addresses: 0x40e020 0x40e020 0x40e020
> ```

对此很多同学也发表了自己的见解，比如说代码写得不够认真，单元测试和集成测试覆盖率不够高，`Code review`不够严谨等等，其实都忽略了一个事实：`人类容易犯错的本质和高昂的测试成本之间的矛盾`。

## 人类容易犯错的本质

制造错误本质就是生物界的一种合理现象，如果没有原始细菌DNA复制过程产生错误，原始细菌始终都只会分裂成原始细菌，而不会变异成多细胞生物、小型动物、大型动物，人类也不可能诞生。没有程序员敢打报票说自己编写的系统永远不存在BUG，即使你是google、微软资深的工程师。况且人的状态不是永远都保持良好的，即使你平时头脑非常灵敏可靠，但不排除在长时间加班或喝醉酒的情况下，无意给自己的系统留下几个难以排查的BUG。

## 高昂的测试成本

每一个系统都希望测试覆盖率能够达到100%，虽然说是可以做到的，但是消耗的时间和人力是非常大的，一般来说写单元测试的时间是逻辑实现代码的3到4倍。而且即使覆盖率到达100%，也不能保证系统没有BUG：也存在某些情况是测试用例没有覆盖到的。

而`Code review`本身是人为的检测，必然存在疏忽的可能性，特别是人力不足和代码修改量大的情况下，比如这次`Let’s Encrypt证书吊销事故`的PR修改代码行数就有2750行，就更不能指望Review者在这么多行代码中找出这一小段有问题的代码了。

## 如何避免？

既然上面的观点貌似都否认了人为避免错误发生的可靠性，那怎么去避免这种故障呢？答案是靠计算机。计算机相比人来说，要可靠得多的，可以长时间进行大量运算而不会出错，相反人类很长时间工作很容易会疲劳产生不确定性。

比如Rust有着很严格的语法和规矩，其实都是吸取了这几十年来软件工程的血与泪教训而积累下来的经验，而设置的条条框框。这个时候监督我们的就不是人或者软性的最佳实践，而是编译器`rustc`，一旦代码不符合硬性的语法规范，编译就无法通过，从而杜绝某些错误的发生。

初学者往往觉得写Rust代码非常难编译通过，而产生了沮丧心理，甚至抗拒、愤怒和谩骂。正如我们面对复杂的交通秩序，有时候也会产生抗拒心理，但其实这些都是以往的各种交通事故，多少人付出了鲜血和生命的代价才积累下来的，所以我们应该去敬畏它，遵守它。

上面的代码尝试用Rust来重写：

```rust
fn main() {
    let mut out = [&0usize; 3];
    for i in 0..out.len() {
        out[i] = &i;
    }
}
```

编译不通过：

```
error[E0597]: `i` does not live long enough
 --> src/main.rs:4:18
  |
4 |         out[i] = &i;
  |         ------   ^^ borrowed value does not live long enough
  |         |
  |         borrow later used here
5 |     }
  |     - `i` dropped here while still borrowed

error: aborting due to previous error
```

或者：

```rust
fn main() {
    let mut out = [&mut 0usize; 3];
    for i in 0..out.len() {
        out[i] = &mut i;
    }
}
```

编译也不通过：

```
error[E0277]: the trait bound `&mut usize: std::marker::Copy` is not satisfied
 --> src/main.rs:2:19
  |
2 |     let mut out = [&mut 0usize; 3];
  |                   ^^^^^^^^^^^^^^^^ the trait `std::marker::Copy` is not implemented for `&mut usize`
  |
  = help: the following implementations were found:
            <usize as std::marker::Copy>
  = note: `std::marker::Copy` is implemented for `&usize`, but not for `&mut usize`
  = note: the `Copy` trait is required because the repeated element will be copied
```

事实上用safe Rust是编写不出来的，因为压根编译不过。这就体现了Rust的安全性，并不只有内存安全和并发安全，还有很多防止低级错误的特性。
