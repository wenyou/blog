---
title: 华为ecs云服务器pptp配置
author: Zeeny
comments: false
date: 2022-03-26
updated: 2022-03-26
tags: [linux,centos,ecs,云主机,vpn]
categories: [linux,centos]
summary: 华为ecs云服务器 pptp 配置
---


# 华为ecs云服务器 pptp 配置

### Step 1:

```
yum install -y epel*
yum install -y ppp pptpd
```

### Step 2:

* vim /etc/ppp/options.pptpd

```
ms-dns 8.8.8.8
ms-dns 114.114.114.114
```

### Step 3:

* vim /etc/pptpd.conf

```
ppp /usr/sbin/pppd   #查找ppp选项，解除本行注释


#查找localip选项和remoteip选项
#其中localip代表虚拟专用网络的本机地址，remoteip代表虚拟专用网络客户端的地址池

localip 192.168.0.2
remoteip 192.168.0.100-127
```

### Step 4:

* vim /etc/ppp/chap-secrets

```
zeenypptp     *       123456  *
```

### Step 5:

* 启动服务

```
systemctl restart pptpd
systemctl status pptpd
```

### Step 6:

* 关闭防火墙

```
setenforce 0
systemctl stop firewalld.service
systemctl disable firewalld.service
firewall-cmd --state
```

### Step 7:

* 开启Linux的路由功能

```
iptables -A FORWARD -i eth0 -j ACCEPT    #让发送至内网网卡的数据全部通过
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE   #修改数据报头信息
#iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to-source 119.8.167.90

service iptables save
systemctl restart iptables

echo 1 > /proc/sys/net/ipv4/ip_forward        #开启Linux的路由功能
cat /proc/sys/net/ipv4/ip_forward             #查看是否启用了路由功能，1代表启用，0代表禁用
```

### Step 8：

* 设置开机自启
<p>添加pptpd服务开机自启</p>

```
systemctl enable pptpd
```