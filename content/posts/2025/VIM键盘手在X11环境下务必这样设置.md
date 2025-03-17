+++
date = '2025-03-17T11:23:47+08:00'
title = 'VIM键盘手在X11环境下务必这样设置'
categories = ["编程技术"]
tags = ["VIM"]
+++

`ESC`是Vim中最常用的键，但是他的位置却很尴尬，`CapsLock`占据着最好的位置却很少用上，我认为以下配置是最合理的：

1. 临时设置：

   ```shell
   setxkbmap -option caps:escape,shift:both_capslock
   ```

2. 永久设置（Debian系）：

   修改`/etc/default/keyboard`的`XKBOPTIONS`这一项：

   ```ini
   XKBOPTIONS="caps:escape,shift:both_capslock"
   ```

> 这个命令的作用是更改键盘映射，具体来说：  
>
> - `caps:escape`：将 **Caps Lock** 键的功能改为 **Escape** 键。  
> - `shift:both_capslock`：当 **左右 Shift 键同时按下** 时，触发 **Caps Lock**。  
>
> 也就是说：
>
> 1. **单独按 Caps Lock** → 变成 **Escape**。
> 2. **同时按左右 Shift** → 触发 **Caps Lock**。  
>
> 这种配置适合经常使用 Vim 或者想减少 Caps Lock 误触的人，同时保留一种触发 Caps Lock 的方式。
