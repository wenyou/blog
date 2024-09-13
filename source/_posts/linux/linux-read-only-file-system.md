---
title: Read-only file system
author: Zeeny
comments: false
date: 2015-09-07
updated: 2015-09-07
tags: [linux,centos]
categories: [linux]
summary: 出现只读文件系统问题
---

# Read-only file system

1. 正常运行中的网站，忽然间出现session目录不可写，连接服务器一看，任何关于写硬盘的命令都不能用，提示Read-only file system，使用一条命令即可搞定此问题：

```
#以读写方式重新挂载文件系统
mount -o remount rw /  
```

2. 如果是文件系统有问题，那就需要在umount状态下执行fsck命令来检查文件系统并修复文件系统中的错误。

```
nohup fsck -y /dev/VolGroup00/LogVol00 > /dev/shm/fscklog &  

#检查好后重启  
reboot
```

* 你可以去CTRL+H 看一下是一组RIAD 还是两组RAID。看一下硬盘的状态。
