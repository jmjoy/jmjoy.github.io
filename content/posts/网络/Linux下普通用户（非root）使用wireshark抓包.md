---
title: "Linux下普通用户（非root）使用wireshark抓包"
date: 2020-05-10T01:21:25+08:00
categories: ["网络"]
tags: []
---

Wireshark是不推荐使用root用户去跑的，原因大家都懂，就是不够安全，而且wireshark有些个性化配置的东西，当然是保存到普通用户下比较好。

可是抓包需要root权限。

官方的解决方案如下：

[https://wiki.wireshark.org/CaptureSetup/CapturePrivileges](https://wiki.wireshark.org/CaptureSetup/CapturePrivileges)

1. 如果是Ubuntu/Debian，可以运行：

   ```bash
   sudo dpkg-reconfigure wireshark-common
   ```
   
   然后
   
   ```bash
   sudo usermod -a -G wireshark {username}
   ```
   
   注销后重新登录就可以了。

1. 其他Linux发行版，则需要手动设置`/usr/sbin/dumpcap`的权限，上述官方文档和网上很多资料都有说，就不重复了。
