+++
date = '2025-05-18T15:15:16+08:00'
title = 'sqlx应付动态WHERE条件'
categories = ["编程技术"]
tags = ["Rust", "CRUD"]
+++

Rust的`sqlx` crate的`query!`宏的编译时检查让我眼前一亮，尤其是根据字段自动生成struct，应付小需求时很是省事。

但是`query!`只能接收固定的sql，对于一些动态的查询场景，比如后台列表根据用户的查询条件动态组装WHERE条件，就显得有些无力。

不过还是有解决方法的，比如这样做（MySQL）：

```rust
use sqlx::{MySqlPool, mysql::MySqlPoolOptions};

struct Params {
    pub bar: Option<String>,
    pub baz: Option<u8>,
}

async fn fetch_foo(db_pool: &MySqlPool, params: &Params) {
    let record = sqlx::query!(
        r#"SELECT * FROM foo WHERE (? is NULL OR bar = ?) AND (? is NULL OR baz = ?) LIMIT 10"#,
        &params.bar,
        &params.bar,
        &params.baz,
        &params.baz,
    )
    .fetch_all(db_pool)
    .await
    .unwrap();

    dbg!(record);
}
```

巧妙地避免了需要动态地拼接sql，不过限制条件是字段是`NOT NULL`的，或者不需要筛选出值是`NULL`的字段。
