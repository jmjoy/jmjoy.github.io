+++
date = '2025-06-02T08:33:11+08:00'
title = 'Linux硬盘迁移完整指南'
categories = ["装机"]
tags = []
+++

最近需要将Linux系统从旧硬盘迁移到新硬盘，经过实际操作验证，整个迁移过程顺利完成，系统运行正常。在此记录详细步骤供参考。

## 迁移步骤

### 1. 准备新硬盘分区

使用gparted等分区工具为新硬盘创建分区：

- 分区表格式：GPT
- 创建一个FAT32格式的EFI分区，大小建议512MB~1GB
- 根据需要创建其他分区（如根分区、home分区等）

### 2. 全盘复制

使用rsync命令复制全盘数据。为确保迁移过程的稳定性，建议从Live CD系统启动进行操作：

   ```shell
   sudo rsync -aHAXv \
     --exclude=/proc \
     --exclude=/sys \
     --exclude=/dev \
     --exclude=/tmp \
     --exclude=/run \
     --exclude=/mnt \
     --exclude=/media \
     --exclude=/lost+found \
     /path/to/source/ /path/to/target/
   
   sudo mkdir -p /path/to/target/{home,proc,sys,dev,tmp,run}
   ```

   需要注意：

   1. 必须确保参数`-aHAXv`，保证复制文件的权限和信息内容一致！
   2. `rsync`的源目录后面的`/`很重要，带`/`表示复制文件夹里面的内容，不带`/`表示复制文件夹自身，而目标目录最好也带上`/`以保持一致性。
   3. `--exclude`参数是相对于源目录的，所以不需要根据源目录的位置而变化。

### 3. 更新fstab文件

通过`blkid`查看分区UUID，然后修正`/etc/fstab`文件：

   ```shell
   sudo blkid
   sudo vim /path/to/target/etc/fstab
   ```

### 4. 重新安装GRUB到新硬盘
  
   ```shell
   # 在新系统内挂载EFI分区
   sudo mkdir -p /path/to/target/boot/efi
   sudo mount /dev/<efi分区设备> /path/to/target/boot/efi

   # 绑定必要的虚拟文件系统
   sudo mount --bind /dev /path/to/target/dev
   sudo mount --bind /proc /path/to/target/proc
   sudo mount --bind /sys /path/to/target/sys
   sudo mount --bind /run /path/to/target/run
   
   # chroot到新系统
   sudo chroot /path/to/target
   
   # 重新安装GRUB（EFI模式）
   grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=<BOOTLOADER-ID>
   
   # 更新GRUB配置
   update-grub
   
   # 退出chroot
   exit
   ```

   需要注意：

   1. `/path/to/target/boot/efi`里面生成的`/EFI/<BOOTLOADER-ID>/grub.cfg`可能需要修改，因为挂载的磁盘号可能不一样。
   2. `grub-install`指定的`bootloader-id`要跟原来系统的一致，否则GRUB启动失败。
   3. 如果GRUB启动失败但是能进入GRUB交互界面，可以通过`echo $root`和`echo $prefix`等命令调试。
