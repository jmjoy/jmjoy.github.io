---
title: "Wine(playonlinux)中文乱码解决方案"
date: 2019-01-24T20:31:51+08:00
draft: false
categories: ["linux"]
tags: ["wine"]
---

> [相关链接] https://blog.csdn.net/ysy950803/article/details/80326832

1. 将中文字体copy到对应wine的目录（本地安装的wine是`~/.wine`，playonlinux是`.PlayOnLinux/wineprefix/对应目录`）下的`drive_c/windows/Fonts/`。

2. 在wine目录下任意位置添加`modify_font.reg`文件：

    ```ini
    REGEDIT4

    [HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\FontLink\SystemLink]
    "Lucida Sans Unicode"="msyh.ttc"
    "Microsoft Sans Serif"="msyh.ttc"
    "MS Sans Serif"="msyh.ttc"
    "Tahoma"="msyh.ttc"
    "Tahoma Bold"="msyhbd.ttc"
    "msyh"="msyh.ttc"
    "Arial"="msyh.ttc"
    "Arial Black"="msyh.ttc"
    ```
    将`msyh.ttc`改成自己想改的中文字体。

3. 在wine命令提示符运行：

    ```bash
    regedit modify_font.reg
    ```

4. 在wine模拟重启电脑即可。