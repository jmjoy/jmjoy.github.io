+++
date = '2025-03-27T23:59:36+08:00'
title = 'Arduino IDE找不到串口的问题'
categories = ["嵌入式"]
tags = ["Arduino"]
+++

最近玩`Arduino UNO R3`这个板子，发现板子通过USB连接进电脑，`Arduino IDE`找不到相应串口。

对应的串口设备叫`/dev/ttyCH341USB0`，问过ChatGPT说不支持这种命名，只支持类似`/dev/ttyUSB0`的。

## 失败的尝试

研究了[`arduino-cli`](https://github.com/arduino/arduino-cli)的代码，架构好复杂，调用了命令行的[`serial-discovery`](https://github.com/arduino/serial-discovery)，最后定位到是包[`go.bug.st/serial`](https://github.com/bugst/go-serial)的正则没有匹配上：

```go
const devFolder = "/dev"
const regexFilter = "(ttyS|ttyHS|ttyUSB|ttyACM|ttyAMA|rfcomm|ttyO|ttymxc)[0-9]{1,3}"
```

于是想想有没有改名的方法。

[Ubuntu系统上ttyUSB*被挂载为ttyCH341USB*](https://blog.csdn.net/lue_app/article/details/134365993)说的重装驱动方法，感觉不靠谱。

重映射又不生效：

/etc/udev/rules.d/99-rename-ch341.rules

```text
SUBSYSTEM=="tty", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", ENV{ID_MM_DEVICE_IGNORE}="1", NAME="ttyUSB%n", MODE="0666"
```

```shell
sudo udevadm control --reload-rules
sudo udevadm trigger
```

还试过软连接，但是[`go.bug.st/serial`](https://github.com/bugst/go-serial)这个包还会判断`/sys/class`里面是否真的有东西，不行。

## 成功的尝试

尝试卸载驱动又重新加载：

```shell
sudo modprobe -r ch341
sudo modprobe -r wch_ch341

sudo modprobe ch341
sudo modprobe wch_ch341
```

发现`/dev/ttyUSB0`居然出现了，疑惑了一会儿，突然想到这两者可能是冲突的。

`/etc/modprobe.d`加了规则：

```text
blacklist wch_ch341
```

重启机器验证过了，没问题，dmesg日志：

```text
[  452.544902] ch341 1-2:1.0: ch341-uart converter detected
[  452.545444] usb 1-2: ch341-uart converter now attached to ttyUSB0
```

`wch_ch341`这个驱动到底是UOS自带的还是我手贱装的，不清楚了。
