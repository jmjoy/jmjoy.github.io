+++
date = '2025-01-17T09:23:49+08:00'
title = '蜂鸟板 Loongarch64新世界 打包alpine Rootfs镜像'
categories = ["嵌入式"]
tags = ["loongarch", "alpine"]
cover = "/images/posts/alpine-2k0300.webp"
+++

最近龙芯蜂鸟板&先锋派发布了新世界（ABI2）试用版，恰逢Alpine之前发布了支持Loongarch64的v3.21版本，于是产生了移植一个Alpine rootfs的想法。

## 做法

需要下载的文件：

1. 网盘里面的buildroot `rootfs.img`
1. [Alpine MINI ROOT FILESYSTEM](https://alpinelinux.org/downloads/) loongarch64版本

### 配置Alpine

需要一个loongarch64的新世界Linux运行环境，可以是安装了buildroot的开发板、龙芯PC、模拟器等。

1. 在该环境中解压alpine-minirootfs文件，可以得到一个根目录文件夹，假设目录是`/tmp/alpine`，运行以下命令chroot到minirootfs：

   ```shell
   mount -o bind /proc proc/
   mount -o bind /dev dev/
   mount -o bind /sys sys/
   chroot . /bin/sh
   ```

1. 配置时间：

   使用`date -s`配置当前时间，或者开启`ntpd`服务。

   不设置时间将导致访问https失败。

1. 执行一些安装步骤：

   ```shell
   echo "nameserver 8.8.8.8" > /etc/resolv.conf
   apk update
   apk add alpine-conf
   setup-hostname
   setup-interface
   ```

1. 添加启动服务：

   ```shell
   apk add acpid openrc busybox-openrc busybox-extras busybox-mdev-openrc
   
   rc-update add acpid default
   rc-update add bootmisc boot
   rc-update add crond default
   rc-update add devfs sysinit
   rc-update add dmesg sysinit
   rc-update add hostname boot
   rc-update add hwclock boot
   rc-update add hwdrivers sysinit
   rc-update add killprocs shutdown
   rc-update add mdev sysinit
   rc-update add modules boot
   rc-update add mount-ro shutdown
   rc-update add networking boot
   rc-update add savecache shutdown
   rc-update add seedrng boot
   rc-update add swap boot
   ```

1. 修改密码：

   ```shell
   passwd
   ```

1. 配置tty：

   ```shell
   echo ttyS0 > /etc/securetty
   ttyS0
   ```

   修改/etc/inittab文件，删除tty1到tty6开始的行，这一步一定要做，否则串口启动会卡死在找ttyX上。

   确保getty这一行：`ttyS0::respawn:/sbin/getty -L 115200 ttyS0 vt100`。

1. 退出设置：

   ```shell
   exit
   umount proc
   umount dev/
   umount sys/
   cd /
   umount /tmp/alpine
   ```

### 打包镜像

1. 挂载buildroot `rootfs.img`：

   由于`rootfs.img`是压缩过的，需要先解压。

   ```shell
   mv rootfs.img rootfs.img.gz
   gunzip rootfs.img.gz

   mkdir /tmp/alpine-mnt
   sudo mount -o loop,offset=1048576 rootfs.img /tmp/alpine-mnt
   ```

1. 将`/tmp/alpine-mnt`里面的文件全删掉，只保留`/boot`，然后将alpine minirootfs的文件全复制到这里。

1. 重新压缩`rootfs.img`：

   ```shell
   gzip rootfs.img
   mv rootfs.img.gz rootfs.img
   ```

1. 完成，可以通过u-boot来刷入rootfs了。

## 参考资料

1. <https://gist.github.com/lidgnulinux/4b40d72d358528e76c6f9be4ad6cbaa5>
1. <https://wiki.luckfox.com/zh/Luckfox-Pico/Luckfox-Pico-Alpine-Linux-2/>
