+++
date = '2025-04-04T23:18:49+08:00'
title = '使用crosstool-ng构建交叉编译工具链'
categories = ["嵌入式"]
tags = ["交叉编译", "loongarch"]
+++

在前文[《Docker搭建loongarch64新世界交叉编译环境》](/posts/2025/docker搭建loongarch64新世界交叉编译环境/)，我提到使用Docker来规避龙芯提供的`loongarch64-linux-gnu-gcc13.3`工具链的GLIBC兼容性问题，但还是不太够方便。

于是尝试自行构建一套。

在手动构建失败后，我发现有一个叫[`crosstool-ng`](https://github.com/crosstool-ng/crosstool-ng)的项目专门用来干这事的，实测可用，下面就说说怎么使用的。

## 安装crosstool-ng

首先编译安装`crosstool-ng`，步骤如下：

```shell
wget http://crosstool-ng.org/download/crosstool-ng/crosstool-ng-1.27.0.tar.xz
tar xf crosstool-ng-1.27.0.tar.xz
cd crosstool-ng-1.27.0
./configure --prefix=/opt/crosstool-ng
make -j
sudo make install
```

将`/opt/crosstool-ng/bin`目录加入PATH环境变量。

## 构建loongarch64-linux-gnu工具链

1. 使用`ct-ng list-samples`能查看支持的架构三元组，还蛮多的，`loongarch64-unknown-linux-gnu`赫然在列。

   然后创建`loongarch64-unknown-linux-gnu`的构建配置：

   ```shell
   mkdir build && cd build
   ct-ng loongarch64-unknown-linux-gnu
   ```

2. 交互式配置界面：

   ```shell
   ct-ng menuconfig
   ```

   配置项请参照文档或者咨询大模型，重点是`Paths and misc options --> Prefix directory`这一项，指定安装的目录，比如`/opt/loongarch64-linux-gnu`。

3. 执行构建：

   ```shell
   ct-ng build
   ```

   这个命令会下载源码tarballs，解压，执行复杂的编译和安装等。

等待一段漫长的时间后，工具链就构建完成了，易用性拉满。
