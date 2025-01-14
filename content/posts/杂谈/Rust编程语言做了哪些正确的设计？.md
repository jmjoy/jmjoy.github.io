---
title: "Rust编程语言做了哪些正确的设计？"
date: 2023-09-15T17:58:56+08:00
categories: ["杂谈"]
tags: []
---

Rust编程语言目前势头很猛，受到了几大互联网巨头的青睐，比如谷歌、微软、Meta，亚马逊等，还作为第二语言进入了Linux内核，国内的华为和字节跳动等公司也有Rust相关的应用。

本人是一个Rust的粉丝，写过不少的Rust应用，这里我总结一下我认为的Rust相比于其他编程语言的更为正确的设计：

## Sum  type

Sum type是比较学术的说法，常见于函数式语言，但是sum type并非函数式语言的专属，老大哥C语言的`union`加上tag之后就是sum type（俗称tagged union），而动态类型语言的变量的类型其实就是封装出来的sum type类型，比如PHP的变量定义：

```c
typedef union _zend_value {
	zend_long         lval;				/* long value */
	double            dval;				/* double value */
	zend_refcounted  *counted;
	zend_string      *str;
	zend_array       *arr;
	zend_object      *obj;
	zend_resource    *res;
	zend_reference   *ref;
	zend_ast_ref     *ast;
	zval             *zv;
	void             *ptr;
	zend_class_entry *ce;
	zend_function    *func;
	struct {
		uint32_t w1;
		uint32_t w2;
	} ww;
} zend_value;
```

但是为啥某些主流的静态类型语言却缺失了这个概念（比如Java、Go等）？我是没想明白的，毕竟老大哥C语言都有这个概念了，缺失了sum type会导致某些需要动态的场景非常的别扭，比如序列化JSON，举个例子，需要反序列化如下的JSON：

```json
[1, "foo", null]
```

Java要怎么做？只能序列化到`List<Object>`，然后靠类型推断来判断是Integer还是String还是Object(null)了，这样就丧失掉静态类型的优势了。

而Rust的sum type关键字`enum`就很好地解决了这个问题，serde_json库的`Value`就已经完美地定义了JSON的结构：

```rust
pub enum Value {
    Null,
    Bool(bool),
    Number(Number),
    String(String),
    Array(Vec<Value>),
    Object(Map<String, Value>),
}
```

## 没有null

这是个老生常谈的问题，null的发明者Tony Hoare将null的设计称为十亿美元错误，意思是null造成的无法计数的错误、漏洞和系统崩溃可能已经造成了十亿美元的损失。

而Rust的解决方式是通过sum type，即Option，来强制用户必须处理null的情况：

```rust
pub enum Option<T> {
    None,
    Some(T),
}
```
在其他一些没有sum type语言中也有类似的做法，比如dart的null safety符号`?`，java的`Optional`（可惜出来得太晚也不是强制性的）。

## 赋值默认是移动语义

这个是我认为非常正确的设计，普遍的编程语言的赋值都是复制语义，对于像bool、int、指针之类的基本类型，复制成本低，因此复制语义对于他们来说比较合适，包括Rust也为基本类型提供了`Copy` trait来转换成复制语义。

但是对于大型的复合体，复制语义明显不适合，因此某些语言区分了值类型和引用类型，成了各种面试题喜欢问到的东西。

而引用类型的本质是封装了指针，赋值的时候其实是复制的指针，指针指向的是同一份内存，产生了“引用”的效果。

像Go这类存在指针的语言，那是否“引用”则清晰多了，不过依然存在slice和map这类引用类型，来对抗复制语义带来的深拷贝造成的影响。

而Rust则更加正确，同一份内存在同一时间只能被一个变量所拥有（所有权的概念），避免了复制语义带来的影响，更加精妙的是，因为所有权的概念，因此可以实现RAII，这是Rust能够在无gc的情况下管理内存的关键。
