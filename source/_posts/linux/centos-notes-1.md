---
title: Centos笔记一
author: Zeeny
comments: false
date: 2015-07-10
updated: 2015-07-10
tags: [linux,centos]
categories: [linux,centos]
summary: Centos笔记一
---


# Centos笔记一

1. yum update  升级补丁。

2. 查看cpu信息（核数、型号等）：`cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c`，查看是几颗物理cpu：`cat /proc/cpuinfo | grep physical | uniq -c`

3. 当前CPU运行在32bit还是64bit模式下: `getconf LONG_BIT` 

4. `cat /proc/cpuinfo | grep flags | grep ' lm ' | wc -l` 结果大于0, 说明支持64bit计算. lm指long mode, 支持lm则是64bit


```
EG:
yum install --downloadonly vsftpd
	
默认的，包会被报存在/var/cache/yum/{RepositoryName}/packages/目录中。在这里 {RepositoryName}是rhel-i386-server-5【我的是fedora】。指定yum的参数--downloaddir，和 --downloadonly一并使用，来指定另外的目录来存放下载的包。 
	
#yum install --downloadonly --downloaddir=/tmp vsftpd
	
使用前先安装：yum install yum-downloadonly
```


#### Centos 内存占满 释放内存

```
top，然后按下shift+m，也就是按内存占用百分比排序，发现排在第一的进程，才占用0.9%，那是什么占用的呢？谷歌了一下，据说是centos为了提高效率，把部分使用过的文件缓存到了内存里。如果是这样的话，我又不需要这样的文件性能，那就可以释放。如下两个命令就可以：
#sync
#echo 3 > /proc/sys/vm/drop_caches

#free -m
```