+++
date = '2025-06-16T17:39:38+08:00'
title = 'VSCode强行修正依赖关系'
categories = ["编程技术"]
tags = ["VSCode"]
+++

在 `UOS 22` / `Deepin 20` / `Debian 10` 上安装[微软官方的 `code.deb`](https://code.visualstudio.com/Download) 时，会遇到依赖关系不满足的问题：

```text
下列软件包有未满足的依赖关系：
 code : 依赖: libxkbfile1 (>= 1:1.1.0) 但是 1:1.0.9.1-1+rebuild 正要被安装
E: 无法修正错误，因为您要求某些软件包保持现状，就是它们破坏了软件包间的依赖关系。
```

仅仅是差了一个小版本号，实在可惜！而且很可能并没有使用到这个依赖库的新特性（即使使用了也不一定会触发相关功能）。

既然如此，那就强行修正依赖关系。由于我之前安装过 `VSCode`，系统已经自动添加了 `VSCode` 官方软件源，因此可以使用 `apt download` 来下载安装包：

```shell
# 下载 .deb 安装包
cd /tmp
apt download code

# 解压 .deb 安装包
mkdir deb-work
cd deb-work
dpkg-deb -R /tmp/code_*.deb extracted/
cd extracted/

# 修改依赖关系，将 Depends 字段中的 `libxkbfile1 (>= 1:1.1.0)` 改成 `libxkbfile1 (>= 1:1.0.9)`
vim DEBIAN/control 

# 重新打包并安装
cd ..
dpkg-deb -b extracted/ code.deb
sudo apt install ./code.deb 
```

通过这种方式，我们成功绕过了版本依赖检查，强行安装了 `VSCode`！

---

其实吧，通过下面的命令就能强制安装了，但是有可能被某些应用商店莫名其妙地干掉！

```shell
apt download code
sudo dpkg -i --force-depends code_*.deb
```
