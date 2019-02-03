---
title: "Linux使用overlayfs测试驱动安装情况"
date: 2019-02-03T10:30:00+08:00
draft: false
---

最近在deepin下使用`deepin-graphics-driver-manager`尝试安装bumblebee方案一直失败，所以被迫研究`deepin-graphics-driver-manager`的源代码，发现其实通过shell脚本来安装的，但是失败日志只说明是apt依赖出了问题而没有更详细的说明。通过`design.md`得知是使用了linux的`overlayfs`这个功能来先模拟安装驱动测试安装情况，然后调用小茶壶测试程序让用户确认安装情况（画面是否撕裂等），用户确认后才真正同步到磁盘，所以`overlayfs`是可以做到类似还原精灵的效果的。



其中`overlayroot`这个工具提供了`overlayroot-enable`和`overlayroot-disable`这两个命令可以开启或者关闭还原精灵的功能，都是需要重启才能生效的，原理是自动修改`/etc/overlayroot.conf`这个文件的配置。



相关链接：

https://blog.csdn.net/luckyapple1028/article/details/77916194

https://blog.csdn.net/luckyapple1028/article/details/78075358