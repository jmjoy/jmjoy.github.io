+++
date = "2017-02-21T10:35:01+08:00"
title = "linux下共享samba文件夹的方法"
categories = ["linux"]
tags = ["samba"]
+++

```bash
sudo mount -t cifs //SERVER/SHARE /MOUNT_DIR -o user=USER,password=PW,uid=$USER,gid=$USER
```
