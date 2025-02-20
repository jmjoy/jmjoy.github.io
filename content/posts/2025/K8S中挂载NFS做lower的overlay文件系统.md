+++
date = '2025-02-20T14:02:54+08:00'
title = 'K8S中挂载NFS做lower的overlay文件系统'
categories = ["运维"]
tags = ["kubernetes", "nfs", "overlay"]
+++

最近有这么一个需求，需要将nfs的共享文件存储只读挂载给多个Pod，但是多个Pod又需要对数据做一些临时性的修改，很自然就想到了overlay的方案。

实施如下：

1. 首先需要在Deployment里面配置nfs volume：

   ```yaml
         volumes:
           - name: nas
             nfs:
               path: /
               server: <my-nfs-server>
   ```

   在containers字段里配置挂载到/data目录：

   ```yaml
             volumeMounts:
               - mountPath: /data
                 name: nas
   ```

2. 然后进入容器里面进行mount overlay的操作：

   ```shell
   mkdir /mnt/upper /mnt/work /mnt/merged
   mount -t overlay overlay -o lowerdir=/data,upperdir=/mnt/upper,workdir=/mnt/work /mnt/merged
   ```

   意料之中就报错了：`permission denie`。因为mount需要特权，修改containers字段的配置：

   ```shell
          securityContext:
            privileged: true
   ```

3. 修改之后再次执行上述的mount overlay操作，结果报新的错误：

   ```txt
   mount: /mnt/merged: wrong fs type, bad option, bad superblock on overlay, missing codepage or helper program, or other error.
       dmesg(1) may have more information after failed mount system call.
   ```

   一开始觉得可能是nfs不支持overlay，但细想又不对，因为之前在阿里云ECS上实施过是可以的，那问题可能出在其他目录上。

   因为容器的文件系统本身就是overlay挂载出来了，于是尝试将/mnt改成emptyDir：

   ```yaml
      volumes:
        - emptyDir: {}
          name: empty
   ```

   ```yaml
          volumeMounts:
            - mountPath: /mnt
              name: empty
   ```

   再次执行上述的mount overlay操作。

   成功了。
