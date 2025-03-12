+++
date = '2025-03-07T13:38:41+08:00'
title = 'Docker搭建loongarch64新世界交叉编译环境'
categories = ["嵌入式"]
tags = ["交叉编译", "UOS", "loongarch"]
+++

> 本文跟龙芯2K0300相关。

> 前提是参照<https://gitee.com/open-loongarch/docs-2k0300>文档下载好交叉编译工具链。

## 背景

为啥这么麻烦？因为我目前使用的Linux发行版是`UnionTech OS Desktop 20 Home`，glibc版本是2.28，运行`/opt/loongarch64-linux-gnu-gcc13.3/bin/loongarch64-linux-gcc`会报错：

```text
/opt/loongarch64-linux-gnu-gcc13.3/bin/loongarch64-linux-gcc: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.34' not found (required by /opt/loongarch64-linux-gnu-gcc13.3/bin/loongarch64-linux-gcc)
```

## 步骤

使用Docker搭建环境来规避glibc版本问题，也不算太麻烦，配置如下：

1. 首先起码给[`open-loongarch`](https://gitee.com/open-loongarch)一个独立的文件夹吧，然后将[`u-boot`](https://gitee.com/open-loongarch/u-boot)和[`linux-6.12`](https://gitee.com/open-loongarch/linux-6.12)等项目`git clone`到这个文件夹里面。

2. 在`open-loongarch`文件夹里面添加两个文件：

    Dockerfile:

   ```dockerfile
   FROM debian:12
   
   # 更新包列表并安装 sudo 等
   RUN apt-get update && apt-get install -y sudo make gcc flex bison bc libelf-dev libssl-dev bsdmainutils git
   
   # 创建 uid 为 1000 的用户，用户名为 loong，并添加到 sudoers 中（允许无密码使用 sudo）
   RUN useradd -m -u 1000 -s /bin/bash loong && echo "loong ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
   
   # 切换到 loong 用户
   USER loong
   
   WORKDIR /home/loong
   ```

   docker-compose.yml:

   ```yaml
   services:
     toolchain:
       build: .
       user: "1000:1000"
       tty: true
       stdin_open: true
       network_mode: "host"
       volumes:
       - /opt/loongarch64-linux-gnu-gcc13.3:/opt/loongarch64-linux-gnu-gcc13.3
       - .:/home/loong/workspace/open-loongarch
       working_dir: /home/loong/workspace/open-loongarch
   ```

   理论上发行版的个人账号UID是1000，docker容器使用用户1000来跑的好处是权限跟宿主机的用户一致，构建的时候不会生成一堆root所有的文件，而且镜像加上了sudo，可以使用sudo来跑一些诸如`apt install`的命令。

3. 运行`docker compose run toolchain`即可创建并进入到容器，如果不想每次run都创建一个容器，可以加上`--rm`参数，或者先`docker compose up -d`然后再`docker compose exec toolchain bash`。

## 后记

如果是需要阅读`u-boot`或者`linux`的源码，并且依赖`clangd`，那么在容器里面运行bear或者linux源码里面的`./scripts/clang-tools/gen_compile_commands.py`脚本生成的`compile_commands.json`，其中的file字段就不太对了，解决方法也简单，Dockerfile里面的用户名改成跟你宿主机的名字一样，并且确保挂载映射的路径跟宿主机的一致就可以。
