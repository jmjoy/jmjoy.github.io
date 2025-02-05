---
title: "Kubernets: Pod内部curl调用k8s api"
date: 2019-09-19T17:49:06+08:00
categories: ["基础架构"]
tags: []
---

在Pod容器内部调用api：
```bash
curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt --header "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"     https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api
```

查询Pod命名空间的的所有pods（前提是serviceaccount有get和list pods的权限）：
```bash
curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt  --header "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api/v1/namespaces/$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)/pods
```
