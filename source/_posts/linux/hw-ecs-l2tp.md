---
title: 华为ecs云服务器l2tp配置
author: Zeeny
comments: false
date: 2022-03-27
updated: 2022-03-27
tags: [linux,centos,ecs,云主机,vpn]
categories: [linux,centos]
summary: 华为ecs云服务器 l2tp 配置
---

# 华为ecs云服务器 l2tp 配置

```
192.168.0.90 | 119.8.167.90
```

### step 1:

```
yum install -y make gcc gmp-devel xmlto bison flex xmlto libpcap-devel lsof vim-enhanced man
yum install -y xl2tpd 
yum install -y libreswan
```

### step 2:

```
vim /etc/ipsec.conf

在config setup中添加一句：nat_traversal=yes，充许传透nat建立l2tp连接
```

### step 3:

* vi /etc/ipsec.d/l2tp_psk.conf

```
conn L2TP-PSK-NAT
    rightsubnet=vhost:%priv
    also=L2TP-PSK-noNAT
conn L2TP-PSK-noNAT
    authby=secret
    pfs=no
    auto=add
    keyingtries=3
    dpddelay=30
    dpdtimeout=120
    dpdaction=clear
    rekey=no
    ikelifetime=8h
    keylife=1h
    type=transport
    left=192.168.0.90
    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any
```

### step 4:

* vim /etc/ipsec.d/ipsec.secrets

```
192.168.0.90 %any: PSK "zeeny-vpn"
```

### step 5:

* vim /etc/sysctl.conf

```
vm.swappiness = 0
net.ipv4.neigh.default.gc_stale_time=120
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.all.arp_announce=2
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 1024
net.ipv4.tcp_synack_retries = 2
net.ipv4.conf.lo.arp_announce=2

net.ipv4.ip_forward = 1
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.default.accept_source_route = 0
```

* `sysctl -p` 
```
sysctl -p
```

### step 6:

* 检验IPSEC服务配置
```
systemctl restart ipsec

ipsec verify
```

* 出现如下问题：
```
Checking rp_filter                                      [ENABLED]
/proc/sys/net/ipv4/conf/eth0/rp_filter                 [ENABLED]
```
* 解决：
```
echo 0 > /proc/sys/net/ipv4/conf/eth0/rp_filter

systemctl restart ipsec
ipsec verify

#检查进程状态
service ipsec status 
```

### step 7:

* 修改L2TP配置文件
* vim /etc/xl2tpd/xl2tpd.conf

```
[global]
listen-addr = 192.168.0.90
ipsec saref = yes

[lns default]
ip range = 192.168.0.128-192.168.0.254
local ip = 192.168.0.1
```

### step 8:

* 修改XL2TPD配置文件

* vim /etc/ppp/options.xl2tpd

```
第一行添加上：require-mschap-v2

DNS服务器写俩就行
```


### step 9:

* 设置连接此VPN服务器的账户与密码
* vim /etc/ppp/chap-secrets

```
zeeny  *       123456  *
olly   *       123456  *
```


### step 10:

* 防护墙与地址转发规则

```
systemctl stop firewalld
systemctl mask firewalld

yum install -y iptables
yum install -y iptables-services

iptables -P INPUT ACCEPT
iptables -F
iptables -X
iptables -Z


iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
iptables -I FORWARD -s 192.168.0.0/24 -j ACCEPT
iptables -I FORWARD -d 192.168.0.0/24 -j ACCEPT

iptables -A INPUT -p udp -m policy --dir in --pol ipsec -m udp --dport 1701 -j ACCEPT
iptables -A INPUT -p udp -m udp --dport 1701 -j ACCEPT
iptables -A INPUT -p udp -m udp --dport 500 -j ACCEPT
iptables -A INPUT -p udp -m udp --dport 4500 -j ACCEPT

iptables -A INPUT -p esp -j ACCEPT
iptables -A INPUT -m policy --dir in --pol ipsec -j ACCEPT

iptables -A FORWARD -i ppp+ -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT


iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to-source 119.8.167.90


service iptables save
systemctl restart iptables
```




### step 11:

* 服务启动检查

```
systemctl status iptables
systemctl status ipsec
systemctl status xl2tpd


# 重启
systemctl restart iptables
systemctl restart ipsec
systemctl restart xl2tpd

# 启动
systemctl start iptables
systemctl start ipsec
systemctl start xl2tpd


#设置 ipsec与xl2tpd 服务开机自启动:
systemctl enable ipsec
systemctl enable xl2tpd


#查看规则：
iptables -L -n
iptables -t nat -nL


#查看状态时
systemctl status xl2tpd
```

* 出现问题：
`Cannot determine ethernet address for proxy ARP`
<p>解决：</p>

```
添加：
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to-source 119.8.167.90

service iptables save
systemctl restart iptables
```

* Windows 11 L2TP连接尝试失败，因为安全层在初始化与远程计算机的协商时遇到了一个处理错误:

<p>查看服务是否开启:</p>
  `windows+r 运行 输入 services.msc`
<p>查找ipsec policy agent</p>

<p>重启后还没有解决</p>
<p>然后在注册表添加两条信息：</p>

    1.单击开始，单击运行，键入regedit，然后单击确定

    2.在注册表编辑器中，找到并单击以下注册表子项︰HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Rasman\Parameters

    3.在编辑菜单上，指向新建，然后单击DWORD 值。

    4. 键入ProhibitIpSec，然后按 enter 键。

    5. 在编辑菜单上，单击修改。

    6. 在数值数据框中，键入1，然后单击确定。

    7. 点击allowL2TPweakcryphto　　修改值为1点击文件 退出

    8. 重启计算机就可以解决了


### Others

* 删除FORWARD 规则：
```
iptables -nL FORWARD --line-number

iptables -D FORWARD 1

删除一条nat 规则  删除SNAT规则
iptables -t nat  -D POSTROUTING  1
```