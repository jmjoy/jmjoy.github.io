+++
date = '2025-07-30T20:10:13+08:00'
title = 'Debian 10编译zed编辑器指南'
categories = ["编程技术"]
tags = []
cover = "/images/posts/2025/Debian-10编译zed编辑器指南.webp"
+++

## 背景

Zed 编辑器是一款现代化的高性能代码编辑器，但其官方发布的 Linux 二进制包依赖较新的 libc 和 libstdc++，在 Debian 10（buster）等老旧发行版上无法直接运行。为了解决依赖不满足的问题，我们需要在本地自行编译 zed 编辑器。

本文将详细介绍在 Debian 10 上编译 zed 编辑器的完整流程，包括 GCC 12 的编译、zed 项目的必要修改、构建与安装，以及启动脚本的修正。

---

## 1. 编译新版 GCC（以 GCC 12 为例）

Debian 10 自带的 GCC 版本较低，无法满足 zed 的构建需求。我们需要手动编译并安装 GCC 12。以下是推荐的编译步骤：

### 安装依赖

```bash
sudo apt update
sudo apt install -y build-essential texinfo flex bison wget unzip python3 g++
```

### 下载并编译 GCC 12

```bash
# 安装路径
sudo mkdir -p /opt/gcc/12

# 下载源码
cd /tmp
wget https://github.com/gcc-mirror/gcc/archive/refs/tags/releases/gcc-12.5.0.zip
unzip gcc-12.5.0.zip
cd gcc-releases-gcc-12.5.0/
./contrib/download_prerequisites
cd ..

# 创建构建目录
mkdir objdir
cd objdir

# 配置
../gcc-releases-gcc-12.5.0/configure --prefix=/opt/gcc/12 --enable-languages=c,c++ --disable-multilib

# 编译与安装
make -j$(nproc)
make install
```

> 注意：编译 GCC 过程较长，建议使用多核（-j$(nproc)）。

---

## 2. 修改 zed 项目以适配自编译 GCC

由于官方构建脚本默认使用 clang 和 mold 链接器，而我们需要使用新编译的 GCC 12，因此需要对 zed 项目做如下修改：

### 2.1 修改 .cargo/config.toml

将链接器从 clang 改为 /opt/gcc/12/bin/g++，并移除 mold 相关参数：

```toml
[target.x86_64-unknown-linux-gnu]
linker = "/opt/gcc/12/bin/g++"
```

### 2.2 修改 script/bundle-linux

- 设置 CC/CXX 环境变量为新 GCC 路径
- 如有需要，设置 HTTPS 代理（如构建过程中需下载依赖包）
- 构建前后设置 LD_LIBRARY_PATH，确保运行时能找到新 libstdc++

```bash
export CC=/opt/gcc/12/bin/gcc
export CXX=/opt/gcc/12/bin/g++
export HTTPS_PROXY=127.0.0.1:1090 # 如需代理

# ...

export LD_LIBRARY_PATH="/opt/gcc/12/lib64:$LD_LIBRARY_PATH"
```

---

## 3. 构建 zed 编辑器

执行官方安装脚本：

```bash
./script/install-linux
```

如无意外，编译完成后会在 `~/.local/zed.app/` 目录下生成可执行文件。

---

## 4. 修正启动脚本

默认安装会在 `~/.local/bin/zed` 创建一个软链，但由于依赖新 libstdc++，需要将其替换为如下 shell 脚本：

```bash
#!/bin/bash

export LD_LIBRARY_PATH=/opt/gcc/12/lib64
exec ~/.local/zed.app/bin/zed "$@"
```

将上述内容保存为 `~/.local/bin/zed`，并赋予可执行权限：

```bash
chmod +x ~/.local/bin/zed
```

## 5. 启动器图标

按照zed官方文档，设置`dev.zed.Zed.desktop`文件：

<https://zed.dev/docs/linux#downloading-manually>

---

## 总结

通过上述步骤，即可在 Debian 10 上成功编译并运行最新版 zed 编辑器。核心思路是：

1. 手动编译并安装新版 GCC（如 12.x）
2. 修改 zed 项目配置，强制使用新 GCC
3. 构建并安装 zed
4. 修正启动脚本，确保运行时能找到新 libstdc++

如遇到其他依赖问题，可根据报错信息进一步排查并安装缺失的库。
