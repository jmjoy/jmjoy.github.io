+++
date = '2025-04-12T12:28:14+08:00'
title = '龙芯2K300蜂鸟板适配Realtek无线网卡'
categories = ["嵌入式"]
tags = ["LINUX", "LOONGARCH", "Realtek"]
+++

由于龙芯2K300蜂鸟板并没有WIFI模块，但是我又想玩玩，所以在淘宝上买了个便宜的`Realtek RTL8188 USB无线网卡`，价格+邮费才7.6元人民币，不得不说人民币在国内的购买力是真的强。

## 安装驱动模块

open-loongarch Linux内核源码：<https://gitee.com/open-loongarch/linux-6.12>

把USB无线网卡插到开发板上，运行`lsusb`，确认无线网卡能被识别到。

在内核源码文件夹里面运行`make xconfig`，由于USB无线网卡是使用Realtek的芯片，搜索`rtl`，把你需要的驱动勾选上。不确定使用哪个驱动的话，看看`dmesg | grep rtl`会出来啥，

对于我的USB无线网卡，结果就是在`.config`文件里增加以下选项：

```ini {name=".config"}
CONFIG_RTL8XXXU=m
CONFIG_RTL8XXXU_UNTESTED=y
```

由于`CONFIG_RTL8XXXU`只能编译成模块而不能编译进内核，因此需要：

```shell
make ARCH=loongarch CROSS_COMPILE=loongarch64-linux-gnu-VERSION- KSRC=/path/of/open-loongarch/linux-6.12 modules -j10
make INSTALL_MOD_PATH=/tmp/rootfs modules_install
scp -r /tmp/rootfs/lib root@192.168.1.10:/
```

scp会花很长时间，其实文件大小很小，就是数量多。

## 下载闭源驱动

插拔USB无线网卡之后，会看到类似以下的内核日志：

```text
[   44.211991] usb 1-1: rtl8xxxu: Loading firmware rtlwifi/rtl8188fufw.bin
[   44.212179] usb 1-1: Direct firmware load for rtlwifi/rtl8188fufw.bin failed with error -2
[   44.212197] usb 1-1: request_firmware(rtlwifi/rtl8188fufw.bin) failed
```

原因是Realtek的网卡驱动本质上是闭源的，Linux内核里面那点东西只是一个开源的壳，调用的闭源的.bin文件，因此需要在网上找到对应的.bin文件，复制到`/lib/firmware`目录。

这里我走了一个弯路，找了一个比较老的`rtlwifi/rtl8188fufw.bin`，看着日志没有报错，实际上又是wifi验证不过，又是dhcp不通过，调来调去，浪费了很多时间。

这里放出个比较新的下载路径：<https://gitlab.com/kalilinux/packages/firmware-nonfree/-/tree/kali/master/rtlwifi>

## 设置网络

我目前使用的新世界系统是Alpine，下面说说使用传统networking服务怎么设置wifi：

1. 安装必要的软件包：

   ```bash
   apk add wpa_supplicant wireless-tools
   ```

2. 创建wpa_supplicant配置文件：

   ```bash
   wpa_passphrase "SSID名称" "密码" > /etc/wpa_supplicant/wpa_supplicant.conf
   ```

3. 编辑网络接口配置文件 `/etc/network/interfaces`：

   ```text
   auto lo
   iface lo inet loopback
   
   auto wlan0
   iface wlan0 inet dhcp
       wireless_mode managed
       wireless_essid SSID名称
       wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
   ```

4. 重启网络服务：

   ```bash
   rc-service networking restart
   ```
