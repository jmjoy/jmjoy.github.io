+++
date = '2025-10-15T14:21:09+08:00'
title = '在Docker中运行LoongArch教学操作系统MaQueOS'
categories = ["编程技术"]
tags = ["loongarch"]
+++

## MaQueOS简介

MaQueOS是一个基于LoongArch架构的开源教学版操作系统。作为一个教学项目，它的代码虽然只有1000多行，但却精巧地实现了操作系统的几大核心功能子系统：

- 进程管理
- 内存管理
- 文件系统
- 中断管理
- 外设驱动

此外，它还为应用程序提供了16个系统调用接口，是学习操作系统原理的绝佳实践平台。

项目地址：<https://gitee.com/dslab-lzu/maqueos>

## 为什么使用Docker？

项目官方文档推荐使用Ubuntu 20.04虚拟机来搭建实验环境。然而，对于已经在使用Linux作为主机的开发者来说，虚拟机显得过于笨重。一个更轻量、更高效的替代方案是使用Docker。

使用Docker的主要挑战在于如何让容器内的QEMU图形化界面（GUI）能够显示在宿主机上。通过X11转发技术，我们可以轻松解决这个问题。

## 操作步骤

### 1. 克隆项目代码

首先，在您的Linux宿主机上选择一个合适的工作目录，然后使用git克隆项目仓库：

```shell
git clone https://gitee.com/dslab-lzu/maqueos.git
```

这会将项目文件下载到当前目录下的`maqueos`文件夹中。

### 2. 授权X11服务访问

为了让Docker容器能够访问宿主机的显示服务，我们需要在运行容器之前，先在宿主机的终端里执行以下命令。这是一种相对安全的授权方式，它允许来自Docker的本地连接。

```shell
xhost +local:docker
```

### 3. 启动Docker容器

接下来，进入刚刚克隆的`maqueos`项目目录，运行以下命令来启动一个配置好的Docker容器：

```shell
cd maqueos
docker run --name maqueos -it -v $PWD:/maqueos --workdir /maqueos -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix ubuntu:20.04 bash
```

命令解析：

- `--name maqueos`: 为容器指定一个名字，方便后续管理。
- `-it`: 以交互模式运行容器。
- `-v $PWD:/maqueos`: 将当前宿主机的项目目录挂载到容器的`/maqueos`目录。
- `--workdir /maqueos`: 将容器的默认工作目录设置为`/maqueos`。
- `-e DISPLAY=$DISPLAY`: 将宿主机的`DISPLAY`环境变量传递给容器，让容器知道图形界面显示在何处。
- `-v /tmp/.X11-unix:/tmp/.X11-unix`: 将X11的socket文件挂载到容器中，用于GUI通信。
- `ubuntu:20.04 bash`: 使用`ubuntu:20.04`镜像，并启动`bash`作为入口。

如果您退出了容器，可以使用以下命令再次进入，无需重新创建：

```shell
docker start maqueos -ai
```

### 4. 在容器内配置环境

进入容器后，我们需要安装编译和运行MaQueOS所需的依赖包。执行以下命令：

```shell
apt update
apt install -y libspice-server-dev libsdl2-2.0-0 libfdt-dev libusbredirparser-dev libfuse3-dev libcurl4 build-essential gcc-multilib libpython2.7 libnettle7 git vim libepoxy0 liblzo2-2 libnuma1 libusb-1.0-0 libgbm1 libgtk-3-0 libbabeltrace1
```

### 5. 运行第一个示例

所有环境都准备就绪了！现在，让我们来运行第一个示例程序，体验一下MaQueOS的启动过程。

```shell
cd /maqueos/code1/run
./run.sh
```

执行脚本后，如果一切顺利，您应该能看到一个QEMU窗口弹出，并顺序执行程序，这标志着您已成功在Docker中运行了MaQueOS！

![maqueos](/images/posts/2025/maqueos.webp)

## 总结

通过以上步骤，我们成功地使用Docker替代了传统的虚拟机方案，搭建了一个轻量、快速的MaQueOS实验环境。这种方法不仅节省了系统资源，也简化了环境配置的流程，让我们可以更专注于操作系统本身的学习和探索。
