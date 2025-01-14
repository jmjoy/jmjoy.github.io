---
title: "Linux开发板如何通过Linux PC上网？"
date: 2024-09-03T22:04:36+08:00
categories: ["嵌入式"]
tags: []
---

通常Linux开发板可以通过网线将开发板和电脑的网口直接相连，达到通讯的效果，便于开发调试。

使用Windows系统电脑的时候，想让开发板通过电脑的另外一张网卡上网（比如无线网卡），比较简单，在控制面板里勾选“允许其他网络用户通过你的计算机的Internet连接进行连接”即可。

而使用Linux系统电脑的时候，就让复杂一些了，下面是配置命令：

```bash
#!/bin/bash

set -xe

# 首先需要设置转发允许，需要一般Linux系统都默认允许，所以这里注释掉
# net.ipv4.ip_forward = 1
# net.ipv4.conf.all.forwarding = 1
# net.ipv6.conf.all.forwarding = 1

NET_BOARD=enp8s0  # 跟开发板相连的物理网卡，配置成你电脑实际的
NET_INTER=wlp7s0  # 能上网的网卡，配置成你电脑实际的

iptables -t nat -A POSTROUTING -o $NET_INTER -j MASQUERADE
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i $NET_BOARD -o $NET_INTER -j ACCEPT
```
