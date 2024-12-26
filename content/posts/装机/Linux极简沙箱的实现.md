## 目的

目的很简单，就是运行一些有风险的应用程序的时候，针对该程序把个人文件目录给屏蔽掉，防止个人文件被读取和篡改。

## 实现方式

用到了`unshare`这个命令，脚本如下：

```shell
#!/bin/bash

set -e

MY_USER=$USER

sudo unshare -f -m -p --mount-proc bash -c "\
    mount -t tmpfs none /data && \
    mount --bind $PWD/sandbox/home /home && \
    /usr/bin/docker-init -- su $MY_USER"
```

## 原理

`unshare`命令用到了Linux的Namespace特性，是 Linux 内核用来隔离内核资源的方式，也是Docker的原理之一。可以通过`man 1 unshare`查看参数的含义。

通过`-m`参数，隔离与其他进程之间的mount操作，防止mount影响到其他进程。

这里把有个人文件的/data目录mount到tmpfs，实现屏蔽的效果，再通过`mount --bind`来屏蔽/home目录，对/home的操作转移到沙箱的文件夹。

因为通过`-p`和`--mount-proc`隔离了PID，所以使用`docker-init`作为1号进程，处理其他进程产生的`SIGCHLD`信号，防止僵尸进程的产生等作用。
