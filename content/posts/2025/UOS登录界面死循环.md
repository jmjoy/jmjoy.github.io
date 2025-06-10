+++
date = '2025-06-10T10:27:06+08:00'
title = 'UOS登录界面死循环'
categories = ["装机"]
tags = ["UOS"]
+++

今天启动`UOS`系统时，遇到了一个奇怪的问题：在登录界面输入密码后，无法进入桌面，而是重新回到登录界面，如此反复循环。

首先尝试查看各种日志文件，包括`lightdm`和`xorg`的日志，虽然发现了一些报错信息，但无法确定这些错误是否是导致问题的根本原因。

接着使用`apt install --reinstall`命令重新安装了一些相关软件包，但问题依然存在。

最后在`Claude`的提示下，检查了`~/.xsession-errors`文件，发现了关键的报错信息：

```text
WARNING: you should run this program as super-user.
WARNING: output may be incomplete or inaccurate, you should run this program as super-user.
/etc/X11/Xsession: 37: /etc/X11/Xsession.d/00deepin-dde-env: [[: not found
/etc/X11/Xsession: 30: .: Can't open ~/.local/share/../bin/env
```

我昨天确实手痒删掉了`~/.local/share/../bin/env`文件，因为路径看起来不太美观。幸好我有备份，恢复文件后问题就解决了。

## 总结

这次问题的根本原因是误删了系统启动时需要的环境配置文件。`~/.xsession-errors`日志文件在排查桌面环境启动问题时非常有用，值得重点关注。
另外，在清理系统文件时要格外小心，即使路径看起来"不美观"，也可能是系统正常运行所必需的。
