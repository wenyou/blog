---
title: 装好系统后增加两块500G硬盘扩展存储空间
author: Zeeny
comments: false
date: 2016-03-28
updated: 2016-03-28
tags: [linux,centos]
categories: [linux]
summary: 装好系统后增加两块500G硬盘扩展/home目录空间
---


# 装好系统后增加两块500G硬盘扩展/home目录空间


### 操作步骤

#### 一、CentOS下添加新硬盘

```
1、查看新硬盘 
	新添加的硬盘的编号为/dev/sda   /dev/sdb
	
2、硬盘分区 
	1) 进入fdisk模式 
		#/sbin/fdisk /dev/sda

	2) 输入n进行分区 

	3) 选择分区类型 
		选择的p    (p: 主分区	linux上主分区最多能有4个 )

	4) 选择分区个数
			1

	5) 设置柱面，这里选择默认值就可以 

	7) 输入w，写入分区表，进行分区
	  
	  
	  分区结束后，查看/dev目录
	  #ls -l /dev
	  
	  可以看到刚刚生成的新分区sda1   
	  
3、格式化分区 
	将新分区格式化为ext4文件系统 
	#mkfs -t ext4 /dev/sda1 
	
	此时就可以用新创建的分区了，另一块盘也这样操作。
	
	如果不使用lvm，直接把新硬盘分区挂载给一个目录使用的话，使用下面的步骤。

4、挂载硬盘 
	1) 创建挂载点 
		#mkdir /storage 

	2) 将/dev/sda1挂载到/storage下 
		#mount /dev/sda1 /storage 
	   
5、设置开机启动自动挂载 
	新创建的分区不能开机自动挂载，每次重启机器都要手动挂载。 
	设置开机自动挂载需要修改/etc/fstab文件 
	#vi /etc/fstab
    在文件的最后增加一行 
    /dev/sda1 /storage ext4 defaults 1 2 
```
* 如果使用lvm，把新增的硬盘用来扩充现有目录的空间，参考下面的步骤。


#### 二、lvm扩容

```
1.sda分区,只分一个sda1

2.在sda1上创建pv
	#pvcreate /dev/sda1
	 
	显示下pv的情况
	# pvdisplay
	 
3.查看系统现在vg的情况
	# vgdisplay
		--- Volume group ---
		VG Name               vg_zxsoft
	  
4.扩容vg
	#vgextend vg_zxsoft /dev/sda1
	  
5.检查下扩容后vg的情况
	#vgdisplay
	
6.查看下系统lv的情况
	#lvdisplay

7.扩容lv
	  #lvextend /dev/vg_zxsoft/lv_home /dev/sda1

8.检查下扩容后的lv
	  #lvdisplay

9.现在系统的分区情况如下,/home没有扩容
	#df -h
	  
10. 将/home扩容。
	#resize2fs /dev/mapper/vg_zxsoft-lv_home
	 
11.扩容后分区的情况
	#df -h
	看到分区已经成功增加了。  
```
