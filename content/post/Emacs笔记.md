---
title: "Emacs笔记"
date: 2016-02-25T17:05:00+08:00
draft: false
categories: ["emacs"]
---

Emacs转移HOME配置：

(setenv "HOME" "path/to/dir")
(load "~/.emacs.d/init.el")

------

emacs-client用root权限修改文件：

/sudo:root@localhost:/etc/fstab

------

用sudo去编辑文本：

(defun sudo ()

"Use TRAMP to `sudo' the current buffer"

(interactive)

(when buffer-file-name

(find-alternate-file

(concat "/sudo:root@localhost:"

buffer-file-name))))