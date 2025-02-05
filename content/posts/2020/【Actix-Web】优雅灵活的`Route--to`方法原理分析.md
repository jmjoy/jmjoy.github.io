---
title: "【Actix Web】优雅灵活的`Route::to`方法原理分析"
date: 2020-03-04T18:32:00+08:00
description: ""
featured_image: ""
categories: ["编程语言"]
tags: ["Rust", "actix-web", "源码分析"]
---

Rust的著名web框架`actix-web`(当前版本2.0.0)有一个方法[Route::to](https://docs.rs/actix-web/2.0.0/actix_web/struct.Route.html#method.to "Route::to")，个人认为非常优雅灵活，比如下面是`examples/basic.rs`的例子：
```rust
#[get("/resource1/{name}/index.html")]
async fn index(req: HttpRequest, name: web::Path<String>) -> String {
    println!("REQ: {:?}", req);
    format!("Hello: {}!\r\n", name)
}

async fn index_async(req: HttpRequest) -> &'static str {
    println!("REQ: {:?}", req);
    "Hello world!\r\n"
}

#[get("/")]
async fn no_params() -> &'static str {
    "Hello world!\r\n"
}
```
展开第一个`attribute marco`标记的函数可以得到：
```rust
#[allow(non_camel_case_types, missing_docs)]
pub struct index;
impl actix_web::dev::HttpServiceFactory for index {
    fn register(self, __config: &mut actix_web::dev::AppService) {
        async fn index(req: HttpRequest, name: web::Path<String>) -> String {
            {
                ::std::io::_print(::core::fmt::Arguments::new_v1(
                    &["REQ: ", "\n"],
                    &match (&req,) {
                        (arg0,) => [::core::fmt::ArgumentV1::new(arg0, ::core::fmt::Debug::fmt)],
                    },
                ));
            };
            {
                let res = ::alloc::fmt::format(::core::fmt::Arguments::new_v1(
                    &["Hello: ", "!\r\n"],
                    &match (&name,) {
                        (arg0,) => [::core::fmt::ArgumentV1::new(
                            arg0,
                            ::core::fmt::Display::fmt,
                        )],
                    },
                ));
                res
            }
        }
        let __resource = actix_web::Resource::new("/resource1/{name}/index.html")
            .name("index")
            .guard(actix_web::guard::Get())
            .to(index);
        actix_web::dev::HttpServiceFactory::register(__resource, __config)
    }
}
```
事实上，`Resource::to`是`Route::to`的封装，`Route::to`的参数是实现了`Factory`的类型，而且看上去接受0个或多个参数并返回一个值的`async fn`都实现了`Factory`（当然并不是全部），这是怎么做到呢？

先看看`Factory`的定义（src/handler.rs）：
```rust
pub trait Factory<T, R, O>: Clone + 'static
where
    R: Future<Output = O>,
    O: Responder,
{
    fn call(&self, param: T) -> R;
}

impl<F, R, O> Factory<(), R, O> for F
where
    F: Fn() -> R + Clone + 'static,
    R: Future<Output = O>,
    O: Responder,
{
    fn call(&self, _: ()) -> R {
        (self)()
    }
}
```
定义了trait `Factory`的一个需要实现的方法`call`，接受任意类型参数T，可以是返回实现了`Responder`的`async fn`，并且提供了`Fn() -> R`的实现，可以理解`async fn no_params() -> &'static str`为什么可以作为`Route::to`的参数了。

对于多个参数的情况，有如下的代码（src/handler.rs）：
```rust
factory_tuple!((0, A));
factory_tuple!((0, A), (1, B));
factory_tuple!((0, A), (1, B), (2, C));
factory_tuple!((0, A), (1, B), (2, C), (3, D));
factory_tuple!((0, A), (1, B), (2, C), (3, D), (4, E));
factory_tuple!((0, A), (1, B), (2, C), (3, D), (4, E), (5, F));
factory_tuple!((0, A), (1, B), (2, C), (3, D), (4, E), (5, F), (6, G));
factory_tuple!((0, A), (1, B), (2, C), (3, D), (4, E), (5, F), (6, G), (7, H));
factory_tuple!((0, A), (1, B), (2, C), (3, D), (4, E), (5, F), (6, G), (7, H), (8, I));
factory_tuple!((0, A), (1, B), (2, C), (3, D), (4, E), (5, F), (6, G), (7, H), (8, I), (9, J));
```
展开其中一个可以得到：
```rust
impl<Func,A,B,Res,O>Factory<(A,B,),Res,O>for Func where Func:Fn(A,B,) -> Res+Clone+'static,Res:Future<Output = O>,O:Responder,{
  fn call(&self,param:(A,B,)) -> Res {
    (self)(param.0,param.1,)
  }
}
```
为参数1到10个的`Fn`分别实现了`Factory`，所以多于10个参数的`Fn`是不能作为`Route::to`的参数的。

那么[FromRequest](https://docs.rs/actix-web/2.0.0/actix_web/trait.FromRequest.html "FromRequest")是怎么转化为不定数量的`fn`参数的各种类型呢，比如`web::Form`, `web::Json`, `web::Path`等。

下面是  相关的实现（src/extract.rs）：
```rust
impl FromRequest for () {
    type Config = ();
    type Error = Error;
    type Future = Ready<Result<(), Error>>;

    fn from_request(_: &HttpRequest, _: &mut Payload) -> Self::Future {
        ok(())
    }
}
```
为0个参数的情况实现了`FromRequest`。
```rust
tuple_from_req!(TupleFromRequest1, (0, A));
tuple_from_req!(TupleFromRequest2, (0, A), (1, B));
tuple_from_req!(TupleFromRequest3, (0, A), (1, B), (2, C));
tuple_from_req!(TupleFromRequest4, (0, A), (1, B), (2, C), (3, D));
tuple_from_req!(TupleFromRequest5, (0, A), (1, B), (2, C), (3, D), (4, E));
tuple_from_req!(TupleFromRequest6, (0, A), (1, B), (2, C), (3, D), (4, E), (5, F));
tuple_from_req!(TupleFromRequest7, (0, A), (1, B), (2, C), (3, D), (4, E), (5, F), (6, G));
tuple_from_req!(TupleFromRequest8, (0, A), (1, B), (2, C), (3, D), (4, E), (5, F), (6, G), (7, H));
tuple_from_req!(TupleFromRequest9, (0, A), (1, B), (2, C), (3, D), (4, E), (5, F), (6, G), (7, H), (8, I));
tuple_from_req!(TupleFromRequest10, (0, A), (1, B), (2, C), (3, D), (4, E), (5, F), (6, G), (7, H), (8, I), (9, J));
```
同样用宏来分别为1到10个参数实现了`FromRequest`。

而且`FromRequest`还为`Options`和`Result`做了实现，所以下面这种写法是可以传入`Route::to`的：
```rust
async fn index(form: Result<web::Form<Data>, Error>) -> &'static str {
    todo!()
}
```
确实是一种很值得借鉴的API设计方式。
