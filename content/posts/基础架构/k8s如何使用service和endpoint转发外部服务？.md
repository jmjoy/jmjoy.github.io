---
title: "k8s如何使用service和endpoint转发外部服务？"
date: 2021-01-08T18:00:23+08:00
categories: ["基础架构"]
tags: []
---

并不是service暴露一个外部ip，而是service转发外部ip+port，做法如下：

首先，创建endpoint：

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: http
  namespace: default
subsets:
- addresses:
  - ip: 10.2.1.1
  ports:
  - name: http
    port: 8080
    protocol: TCP
```
其中`10.2.1.1:8080`是外部服务。

其次，创建service，其中名字一定要和endpoint的一致：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: http
  namespace: default
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
```

然后访问service自动生成的虚拟ip加port即可。
