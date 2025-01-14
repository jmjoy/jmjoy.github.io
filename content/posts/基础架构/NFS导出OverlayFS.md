---
title: "NFS导出OverlayFS"
date: 2020-04-20T13:07:25+08:00
categories: ["基础架构"]
tags: []
---

需要Linux内核版本 > 4.16， 内核开启选项`OVERLAY_FS_NFS_EXPORT`，同时`mount -t overlay`时指定`-o nfs_export=on`。
NFS使用`showmount -e localhost`查看挂载点。
使用`modinfo overlay | grep nfs`查看overlay是否支持nfs。

内核参数文档：
> config OVERLAY_FS_NFS_EXPORT
> 	bool "Overlayfs: turn on NFS export feature by default"
> 	depends on OVERLAY_FS
> 	depends on OVERLAY_FS_INDEX
> 	depends on !OVERLAY_FS_METACOPY
> 	help
> 	  If this config option is enabled then overlay filesystems will use
> 	  the index directory to decode overlay NFS file handles by default.
> 	  In this case, it is still possible to turn off NFS export support
> 	  globally with the "nfs_export=off" module option or on a filesystem
> 	  instance basis with the "nfs_export=off" mount option.
> 
> 	  The NFS export feature creates an index on copy up of every file and
> 	  directory.  This full index is used to detect overlay filesystems
> 	  inconsistencies on lookup, like redirect from multiple upper dirs to
> 	  the same lower dir.  The full index may incur some overhead on mount
> 	  time, especially when verifying that directory file handles are not
> 	  stale.
> 
> 	  Note, that the NFS export feature is not backward compatible.
> 	  That is, mounting an overlay which has a full index on a kernel
> 	  that doesn't support this feature will have unexpected results.
> 
> 	  Most users should say N here and enable this feature on a case-by-
> 	  case basis with the "nfs_export=on" mount option.
> 
> 	  Say N unless you fully understand the consequences.

Overlay nfs_export 文档：
[https://patchwork.kernel.org/patch/10012463/](https://patchwork.kernel.org/patch/10012463/)

Overlay内核文档：
[https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt](https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt)
