---
title: "Docker参考配置"
date: 2020-07-29T14:17:12+08:00
categories: ["基础架构"]
tags: []
---

文件： /ect/docker/daemon.json

```json
{
  "registry-mirrors": ["https://hub-mirror.c.163.com/"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {"max-size":"2G", "max-file": "3"},
  "graph": "/data/docker"
}
```
