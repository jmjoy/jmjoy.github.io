---
title: "PHP：array_merge使用误区"
date: 2020-12-31T15:13:47+08:00
categories: ["编程语言"]
tags: []
---

今天被一个小问题浪费了一个上午，代码如下：

```php
curl_setopt_array($ch, array_merge($headers, [
    CURLOPT_URL => "http://localhost:9501/",
    CURLOPT_RETURNTRANSFER => 1,
]));
```

实际上上面的代码是不符合预期的，加上`curl_setopt_array`也没有提示错误，所以顿时感到很疑惑。
究其原因，是`array_merge`在处理int类型index的数组时，会将index重置从0开始排序，而且因为`CURLOPT_URL`等常量是int类型的，所以悲剧发生了！

合理的做法是不使用`array_merge`了，比如用原始方法如下：
```php
$headers[CURLOPT_URL] = "http://localhost:9501/";
$headers[CURLOPT_RETURNTRANSFER] = 1;
curl_setopt_array($ch, $headers);
```
