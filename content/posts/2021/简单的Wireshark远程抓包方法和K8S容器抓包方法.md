---
title: "简单的Wireshark远程抓包方法"
date: 2021-07-15T10:04:06+08:00
categories: ["编程技术"]
tags: []
---

## 简单的Wireshark远程抓包方法

一行sh命令即可：

```bash
ssh root@<IP> tcpdump -i <网卡> -p -U -w - 'tcp port 80' | wireshark -i - -k -p
```

## K8S容器抓包方法

1. 首先找到容器所在node和pod ip：

    ```bash
    kubectl -n <NAMESPACE> get pod <POD_NAME> -o wide
    ```
    得到类似下图的输出：

    ![image.png](https://upload-images.jianshu.io/upload_images/18494435-92c978c17c115cb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


1. 登上相应node节点，使用 `ip route | grep <POD_IP>` 来获取虚拟网卡的名字：

    ![image.png](https://upload-images.jianshu.io/upload_images/18494435-7f5c558bc1fa59b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    其中calidb933880aad就是虚拟网卡的名字。

1. 使用tcpdump指定网卡和规则抓包就行，比如应用最上述的Wireshark远程抓包办法。
