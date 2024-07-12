---
layout: post
title: Centos安装Samba服务
description: Centos安装Samba服务
category: centos
tags: [linux,centos,Samba]
---
#Centos安装Samba服务#

##1.安装Samba服务##

```
	
	# yum install samba samba-client samba-swat
	
	查看安装状况:
	rpm -qa|grep samba
	
	amba服务器安装完毕，会生成配置文件目录/etc/samba和其它一些samba可执行命令工具，/etc/samba/smb.conf是samba的核心配置文件，/etc/init.d/smb是samba的启动/关闭文件
	
	启动Samba服务器:
	可以通过/etc/init.d/smb start/stop/restart来启动、关闭、重启Samba服务
	
	查看samba的服务启动情况：
	service smb status
	
	设置开机自启动:
	chkconfig --level 35 smb on
	
```

配置Samba服务

```
   
	设置共享：
	
	1.创建需要共享的目录 /var/zxoapublicfiles/share
	修改该目录权限: chmod 777 share
	
	2.vi /etc/samba/smb.conf
	搜索security=user
	加入
	security = user
	username map = /etc/samba/sambausers
	
    文件最后加入:允许访问用户为 zxsoftoa 共享目录/var/zxoapublicfiles/share
    [zxsoftoa]
	comment = Share Folder with username and password
	path = /var/zxoapublicfiles/share
	public = yes
	writable = yes
	vaild users = zxsoftoa
	create mask = 0700
	directory mask = 0700
	;force user = nobody
	;force group = nogroup
	available = yes
	browerable = yes
	
	
	global workgroup = MSHOME 改为 WORKGROUP
    
    3. 增加网络访问用户 zxsoftoa
    sudo useradd zxsoftoa
    删除用户:sudo userdel -r zxsoftoa
    
    4.新增网络使用者的帐号
    vi /etc/samba/smbusers
    新增一行：
    zxsoftoa = network username
    
    
    5. 更改zxsoftoa的网络访问密码
    sudo smbpasswd -a zxsoftoa
    删除网络使用者的帐号的命令把上面的 -a 改成 -x 
    
    6. 配置防火墙策略
    vi /etc/sysconfig/iptables 
    加入samba的端口
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 139 -j ACCEPT  
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 445 -j ACCEPT
	
    重启防火墙服务
    service iptables restart  
    
    6.重启
    /etc/init.d/smb restart
    
    7. 然后在window运行\\172.16.10.18 访问share 用户名为zxsoftoa 密码输入已设置的密码
```