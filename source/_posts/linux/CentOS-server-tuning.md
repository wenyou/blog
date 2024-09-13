---
title: CentOS服务器调优
author: Zeeny
comments: false
date: 2016-07-08
updated: 2016-07-08
tags: [linux,centos]
categories: [linux,centos]
summary: CentOS服务器调优
---


# CentOS服务器调优

### 1. centos ssh速度慢

```
ssh速度慢：
vi /etc/ssh/sshd_config
UseDNS no
GSSAPIAuthentication no
GSSAPIAuthentication yes

systemctl restart sshd.service 或 /etc/init.d/sshd restart
```

### 2. centos ulimit 文件描述符的设置

```
查看：
	ulimit -n
	ulimit -u
	ulimit -s
	ulimit -a
	
修改：
vi /etc/security/limits.conf
	添加：
	root soft nofile 65536
	root hard nofile 65536
	* soft nofile 65536
	* hard nofile 65536
	
	root soft nproc 32768
	root hard nproc 32768
	* soft nproc 32768
	* hard nproc 32768
	

vi /etc/security/limits.d/90-nproc.conf
	#*         soft    nproc     1024
	*          soft    nproc     32768
	
	
vi /etc/pam.d/login最后添加禁止调试文件
	session required /lib/security/pam_limits.so
	
或：
Shell命令：
	echo ulimit -HSn 65536 >> /etc/rc.local
	echo ulimit -HSn 65536 >>/root/.bash_profile
	ulimit -HSn 65536
```

### 3. centos 内核参数的设置

```
vi /etc/sysctl.conf
	添加：
	kernel.panic_on_oops = 1
	kernel.panic = 1
	kernel.panic_on_oops = 1
	kernel.softlockup_panic = 1
	kernel.hung_task_panic = 1
   	
	vm.swappiness = 0
	net.core.netdev_max_backlog = 4096
	net.core.somaxconn = 1024
	net.core.rmem_max = 16777216
	net.core.wmem_max = 16777216
	net.ipv4.tcp_rmem = 4096 87380 16777216
	net.ipv4.tcp_wmem = 4096 65536 16777216
	net.ipv4.tcp_max_syn_backlog = 8192
	
	net.ipv6.conf.all.disable_ipv6 = 1
	net.ipv6.conf.default.disable_ipv6 = 1
	net.ipv6.conf.lo.disable_ipv6 = 1
	
使生效：
	`/sbin/sysctl -p`
```

### 网站最大并发连接数、新建数配置调优.md
[text](../java/website-configuration-tuning.md)