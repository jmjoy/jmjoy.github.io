---
title: "Rust的多继承和mixin机制"
date: 2016-05-23T19:22:03+08:00
draft: false
categories: ["其它"]
---

先上一段Rust代码：

```rust
trait A {
    fn say(self);
}

trait B {
    fn say(self);
}

struct S;

impl A for S {
    fn say(self) {
        println!("S say: A"); 
    }
}

impl B for S {
    fn say(self) {
        println!("S say: B"); 
    }
}

fn say_a<T: A>(t: T) {
    t.say();
}

fn say_b<T: B>(t: T) {
    t.say();
}

fn main() {
    say_a(S);
    say_b(S);
}
```

给java和php的童鞋的解释就是：两个接口A和B，一个类S，S分别针对A和B实现了say()这个方法。

因为这在java或php(抄袭java的面向对象)是不可能的，万恶的根源在于他们是将类的所有方法都写在类这个域里面，然后得出“多继承”可能会导致函数冲突(两个父类拥有相同原型的函数)的荒谬理论。

在Rust中，得益于成员函数写在了struct之外(而且可以是多个impl的域)，函数冲突的问题重来就没有过。

另外Rust的trait可以有默认实现，和有函数实体的父类异曲同工。

====

2016.6.10

Rust没有继承，struct之间并不能复用代码。