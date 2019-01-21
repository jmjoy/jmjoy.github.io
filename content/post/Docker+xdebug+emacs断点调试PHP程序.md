+++
date = "2017-02-03T11:06:14+08:00"
title = "Docker+xdebug+emacs断点调试PHP程序"
showonlyimage = false
image = ""

+++

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**

- [Docker 配置](#docker-配置)
- [浏览器配置](#浏览器配置)
- [Emacs配置](#emacs配置)

<!-- markdown-toc end -->

# Docker 配置 #

首先得有一个配置好的`docker image`, 这里使用[laraedit](https://hub.docker.com/r/laraedit/laraedit/) ，homestead的docker替代版，已经包含了php的`xdebug`扩展。

然后运行`docker run ...`，这里省略。

查看宿主机的docker0的ip

``` shell
$ ip addr show docker0
7: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::f4d2:49ff:fedd:28a0/64 scope link 
       valid_lft forever preferred_lft forever
```

运行 `docker exec -it _the_name_of_your_container_ bash`

配置`/etc/php/7.0/fpm/conf.d/20-xdebug.ini`如下
*（其中172.17.0.1地址是通过上面`ip addr show docker0`这条命令得到的地址）*

``` ini
zend_extension=xdebug.so
xdebug.remote_enable=On
xdebug.remote_host=172.17.0.1
```

重启php-fpm

``` shell
supervisorctl restart php-fpm7.0
```

通过`phpinfo()`确保`php-xdebug`配置无误：

![phpinfo](/images/Docker+xdebug+emacs断点调试PHP程序/phpinfo.png)

# 浏览器配置 #

我使用的是firefox，需要一个叫`theeasiestxdebug`的扩展来控制`xdebug`的行为，chrome差不多，
也可以不安装扩展，但是需要自己设置url的query参数或者cookie。

# Emacs配置 #

Emacs安装`geben`这个package，如果使用spacemacs的话可以使用`geben layer`

这里说一下spacemacs的配置方法

``` shell
git clone https://github.com/rubberydub/spacemacs-geben.git ~/.spacemacs.d/geben
```

然后将`geben`添加到`dotspacemacs-configuration-layers`

然后就可以愉快得使用了～

![debug](/images/Docker+xdebug+emacs断点调试PHP程序/debug.png)

PS: geben默认是通过创建新窗口来打开debug到的新buffer的，我个人觉得有点烦，所以我就配置了如下（切换buffer）：

``` emacs-lisp
(custom-set-variables
 '(geben-display-window-function (quote popwin:switch-to-buffer)))
```
