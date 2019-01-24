---
title: "Ubuntu交换CapLock和Esc"
date: 2016-02-24T20:58:00+08:00
draft: false
categories: ["linux"]
---

~/.Xmodmap

```bash
clear Lock
keysym Caps_Lock = Escape
keysym Escape = Caps_Lock
add Lock = Caps_Lock
```

------

Windows: SharpKeys.

以上