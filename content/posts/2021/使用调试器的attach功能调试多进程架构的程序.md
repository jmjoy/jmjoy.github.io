---
title: "使用调试器的attach功能调试多进程架构的程序"
date: 2021-06-24T09:59:14+08:00
categories: ["编程技术"]
tags: []
---

对于多进程架构的程序，如服务器程序nginx和php-fpm等，调试子进程需要用到调试器的attach功能。

下面以Clion和php-fpm为例，调试步骤如下，其他编辑器/IDE和程序可以作为参考：

1. 首先，需要将程序的worker子进程数量调成固定的1个，以便请求/任务被调度到固定的进程，php-fpm的调整如下：

```ini
# File: etc/php-fpm.d/www.conf
pm = static
pm.max_children = 1
```

2. 设置内核参数：

```bash
sudo sysctl kernel.yama.ptrace_scope=0
```

若不设置这个参数会有以下的报错：

> com.jetbrains.cidr.execution.debugger.backend.gdb.GDBDriver$GDBCommandException: ptrace: 不允许的操作.
> Debugger detached

3. 启动程序，最好使用本用户来启动：

  ![php-fpm启动](https://upload-images.jianshu.io/upload_images/18494435-729aa1b71e10e6c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 点击Clion的菜单"Run -> Attach to Process"，或者使用快捷键Ctrl+Alt+5，来启动Attach界面，筛选进程名字，选择子进程：

![Attach界面](https://upload-images.jianshu.io/upload_images/18494435-7b36930b2d315f03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5. 在Clion上打上调试断点，然后用Postman等工具做请求，就可以看到调试器进入到相应的位置了：

![调试](https://upload-images.jianshu.io/upload_images/18494435-a2c4ae154176c655.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
