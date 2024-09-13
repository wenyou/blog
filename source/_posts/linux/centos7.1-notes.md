---
title: Centos7.1笔记
author: Zeeny
comments: false
date: 2015-09-06
updated: 2015-09-06
tags: [linux,centos]
categories: [linux,centos]
summary: Centos7.1笔记
---



# Centos7.1笔记

1. 修改主机名

```
vi /etc/hostname
```

2. 查看磁盘信息

```
fdisk -l 
df -h
```
	
3. 重启网络

```
sudo systemctl restart network

get status of network service:
sudo systemctl status network.service

or:
sudo systemctl status network
```
	
4. 防火墙

```
查看状态
	firewall-cmd --state
	systemctl status firewalld.service
	
	firewall-cmd --get-services 获取所有支持的服务


关闭
	systemctl stop firewalld.service


禁止开机启动
	systemctl disable firewalld.service
	
	
	zkServer.sh start /home/hadoop/zookeeper-3.4.6/conf/zoo-solr.cfg
	
添加开放21端口：
	firewall-cmd --query-port=21/tcp 查看端口是否开放
	firewall-cmd --zone=public --add-port=21/tcp --permanent
				[--zone #作用域  --permanent #永久生效，没有此参数重启后失效]
	
添加开放被动端口范围：
	firewall-cmd --zone=public --add-port=64000-65535/tcp --permanent


重启防火墙：
	firewall-cmd --reload 

	
	firewall-cmd --zone=public --add-forward-port=port=50070:proto=tcp:toaddr=172.16.222.55 --permanent



安装 iptables
	yum install iptables-services
```


5. 网络相关命令

```
ip route show  【查看路由表】
ip addr  【网卡详情】
	
	
安装ifconfig
yum provides ifconfig
yum install net-tools
```


6. FTP不能上传文件解决方案：

```
getsebool -a|grep ftp

setsebool -P ftp_home_dir  on
setsebool -P ftpd_full_access on

systemctl restart  vsftpd.service
```