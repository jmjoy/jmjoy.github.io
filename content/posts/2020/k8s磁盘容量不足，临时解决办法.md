---
title: "k8s磁盘容量不足，临时解决办法"
date: 2020-12-30T14:55:19+08:00
categories: ["基础架构"]
tags: []
---

kubelet启动参数添加以下配置：
```
--eviction-hard=memory.available<500Mi,nodefs.available<1Gi,imagefs.available<5Gi
```
然后
```
systemctl daemon-reload
systemctl restart kubelet
```
等应用都起来之后，解决了故障问题，再慢慢清理磁盘吧。
