---
title: "kubernetes查找kube-scheduler和kube-controller-manager的leader的方法"
date: 2021-02-05T15:20:53+08:00
categories: ["基础架构"]
tags: []
---

kubernetes查找kube-scheduler和kube-controller-manager的leader的方法

## kube-scheduler

```bash
kubectl -n kube-system get endpoints kube-scheduler -o jsonpath='{.metadata.annotations.control-plane\.alpha\.kubernetes\.io/leader}'
```

## kube-controller-manager
```bash
kubectl -n kube-system get endpoints kube-controller-manager -o jsonpath='{.metadata.annotations.control-plane\.alpha\.kubernetes\.io/leader}'
```

## 参考
https://support.d2iq.com/s/article/Determining-the-kube-scheduler-and-kube-controller-manager-leader-for-troubleshooting
