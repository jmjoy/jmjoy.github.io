---
title: "Nginx信号的坑"
date: 2021-01-09T22:35:25+08:00
categories: ["基础架构"]
tags: []
---

Nginx有一个坑，应该是作者理解反了SIGTREM和SIGQUIT的含义，引用[官方文档](http://nginx.org/en/docs/control.html
)的描述：
> TERM, INT	fast shutdown
> QUIT	graceful shutdown

事实上按照GNU的标准，SIGTREM是平滑退出的，SIGQUIT是立即退出。

对于平常使用`nginx -s`来控制nginx，问题不大，但是对于k8s来说，则是个大问题，因为k8s的Pod默认退出是发送的SIGTREM信号的，所以按照默认的方式，会导致在k8s上的nginx退出时不平滑。

解决办法，是将Pod的Lifecycle的PreStop钩子设置成：`nginx -s quit`。
