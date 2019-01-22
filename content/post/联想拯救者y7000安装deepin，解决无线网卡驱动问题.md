---
title: "联想拯救者y7000安装deepin 15.9，解决无线网卡驱动问题"
date: 2019-01-22T13:15:13+08:00
draft: false
categories: ["linux"]
tags: ["linux", "联想拯救者", "y7000", "deepin"]
typora-root-url: ../../../static
---

> 前段时间买了个新的笔记本（联想拯救者y7000），将服役了6年的老笔记本（联想Z480）上安装了deepin系统的固态硬盘直接搬过来作为新笔记本的副硬盘，启动，显卡、声卡、有线网卡等都运作良好，但是无线网卡出现了问题，后面google了很多方案终于解决了，所以记录一下。

## 分析原因

总结了一下，主要有可能出现的是三个问题：

- 驱动问题

  如果通过命令`ifconfig`没能发现类似`wlp7s0`的无线网络，可以判断是这个问题。

- 无线网卡被hard blocked的问题

  运行命令`rfkill list all`，出现如下结果

  ```
  0:ideapad_wlan: Wireless LAN 
  Soft blocked: no 
  Hard blocked:yes 
  1:ideapad_bluetooth: Bluetooth 
  Soft blocked: no 
  Hard blocked: yes 
  2:phy0: Wireless LAN 
  Soft blocked: no 
  Hard blocked:no 
  3:hci0: Bluetooth 
  Soft blocked: yes 
  Hard blocked: no 
  ```

  可以看到，优先级前的ideapad_wlan的Hard blocked 默认为yes，即deepin默认关闭了硬件wifi开关，而联想拯救者Y7000的wifi只有软件开关，没有硬件开关的启动，所以引起了wifi无法开启的问题。

- 网卡驱动的电源管理问题

  使用命令`dmesg`查看日志，如果出现

  ![dmesg_r8822de](/images/linux/dmesg_r8822de.png)

  那很可能就是这个问题。

## 解决方案

### 查看无线网卡型号

```bash
lspci
```

找到如下输出：

![lspci](/images/linux/lspci.png)

可以知道无线网卡的型号是rtl8822be。

### 安装驱动

如果确认是驱动问题，则可以尝试通过重装驱动解决：

```bash
sudo apt install --reinstall firmware-realtek
```

### 解决无线网卡被hard blocked的问题

禁用ideapad_laptop驱动

```bash
echo "blacklist ideapad_laptop" | sudo tee /etc/modprobe.d/backlist-ideapad.conf
```

### 解决网卡驱动的电源管理问题

```bash
echo "blacklist ideapad_laptop" | sudo tee /etc/modprobe.d/disableideapad.conf
```



> 相关链接：
>
> https://my.oschina.net/u/3552749/blog/1929081
>
> http://forum.ubuntu.org.cn/viewtopic.php?t=489020

