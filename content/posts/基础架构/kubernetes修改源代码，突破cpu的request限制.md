---
title: "kubernetes修改源代码，突破cpu的request限制"
date: 2021-02-04T17:18:48+08:00
categories: ["基础架构"]
tags: []
---

## 背景

由于业务方配置Deployment时设置resource的request过大，以及linux内核在4.19版本之前的关于cgroup的cpu限流问题，导致node的资源使用率并不高的情况下，node却不能被调度更多的Pod，故采取修改kubernetes源码的方式来解决。

## kubernetes版本

使用和修改的版本是v1.16.9。

## 思路

默认情况下，kubernetes对于node节点的resource的request到达100%的时候，就不再允许Pod被调度到该节点上，可以用`kubectl describe node <NODE>`来查看，在Allocated resources这一项：

![image.png](https://upload-images.jianshu.io/upload_images/18494435-d9c91d57b943c8ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

默认情况下这一项是不可以超过100%的。

现在的思路就是让kubernetes可以突破这个限制，可以超过100%，最高不超过200%。

## 代码修改

修改的是`pkg/scheduler/algorithm/predicates/predicates.go`文件的`PodFitsResources`方法，修改如下这一行：
```go
	if allocatable.MilliCPU < podRequest.MilliCPU+nodeInfo.RequestedResource().MilliCPU {
		predicateFails = append(predicateFails, NewInsufficientResourceError(v1.ResourceCPU, podRequest.MilliCPU, nodeInfo.RequestedResource().MilliCPU, allocatable.MilliCPU))
	}
```
只要把`allocatable.MilliCPU`乘以2，就可以达到200%那个效果了。

影响到的组件有`kube-scheduler`和`kubelet`，其中`kubelet`被影响的位置是`pkg/kubelet/lifecycle/predicate.go`的`Admit`方法中的这一行：

```go
	fit, reasons, err := predicates.GeneralPredicates(podWithoutMissingExtendedResources, nil, nodeInfo)
```

## 重新部署

`kube-scheduler`和`kubelet`都需要重新部署，首先将所有节点的`kubelet`都替换成hack之后的`kubelet`再重启，然后替换master节点的`kube-scheduler`。

最终效果如下：

![image.png](https://upload-images.jianshu.io/upload_images/18494435-dc6747a3d7cf4756.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 能够超过100%啦，间接地实现一种cpu超卖的效果。

# 题外

有点喜感的是，使用hack之后的`kubelet`，在`kubectl get nodes`显示的版本号后缀带了`-dirty`，这个是指修源码后git未做版本提交导致的，如果提交版本之后，后缀就是版本的hash，如果打上tag，那后缀就是tag。

![image.png](https://upload-images.jianshu.io/upload_images/18494435-5d296f3aa7d4fe66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
