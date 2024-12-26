**只在X11下有效。**

## 临时设置

1. caps:escape: 将Capslock映射成Esc。
2. shift:both_capslock: 同时按下两个Shift等于按下Capslock。

```shell
setxkbmap -option caps:escape,shift:both_capslock
```

其他规则请见： `/usr/share/X11/xkb/rules/xorg.lst`

## 永久设置

修改`/etc/default/keyboard`的`XKBOPTIONS`参数:

```ini
XKBOPTIONS="caps:escape,shift:both_capslock"
```

> 参考： https://n1ghtmare.github.io/2021-05-19/remapping-caps-lock-to-esc-on-arch-linux/
