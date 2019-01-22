+++
date = "2016-03-20T21:41:58+08:00"
draft = false
title = "Emacs(Spacemacs) on Windows"
categories = ["emacs"]
+++

虽说Emacs是一个跨平台的软件，但在Microsoft Windows上使用Emacs会出现很多其他平台上没有的问题，
但是要用的始终要用，那就慢慢解决问题呗。

# 用哪个Emacs？

选择很多，但是我用Spacemacs推荐的吧。

https://github.com/syl20bnr/spacemacs#windows

当然这个是非官方的编译版，但是不错。

# 解决问题

## quelpa在Windows下的问题

我是一名使用Spacemacs的PHPer，需要用过`php`这个layer，其中的`php-extras`这个包在Windows下肯定是安装不成功的（linux下没问题）。究其原因，是因为这个包使用quelpa从github上pull的，而quelpa需要 `tar`这个工具，而且还不能是Windows原生编译出来的`tar`。

解决办法如下：

https://github.com/quelpa/quelpa#tar

## gtags的问题

这个问题会提示莫名其妙的 ```No such file or directory, gtags```，其实原因很简单，因为Spacemacs推荐的 Emacs缺少了gtags.el这个文件。

解决办法如下：

https://github.com/arnested/drupal-mode/issues/67#issuecomment-198870045

## diff

在使用 Spacemacs 的 go layer 时会遇到 diff 缺失的问题，这个解决办法就是直接下载 GNU Win32 的
diffutil使用。

http://gnuwin32.sourceforge.net/packages/diffutils.htm

