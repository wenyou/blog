---
layout: post
title: iptables配置NAT转发
description: iptables配置NAT转发
category: linux
tags: [linux,centos,iptables,nat]
---
#iptables配置NAT转发#

###iptables 实现centos内网机器访问外网###

* 环境：一台带外网和内网的机器，另一台只有内网，默认不能上网。两台机器都是centos系统。
* 带外网机器的外网ip为15.124.225.102， 内网ip为172.16.37.39
* 内网机器的内网ip为172.16.37.32

* 设置方法

```

	1. 在带外网的机器上设置iptables规则：
		iptables -t nat -A POSTROUTING -s 172.16.37.32 -j SNAT --to 15.124.225.102
	
		#如果想让整个内网的机器全部上网，只需要把 -s 172.16.37.32 换成-s 172.16.37.0/255.255.0.0 即可。
	
	2.在带外网机器上打开转发：
	  #首先查看是否已经打开
	  sysctl -a |grep 'net.ipv4.ip_forward'
	  #如果值为1，则说明已经打开，否则需要修改配置文件 /etc/sysctl.conf
	  #打开该配置文件，找到该参数，使其变为
	  net.ipv4.ip_forward = 1
	  #然后运行 
	  sysctl -p
	  
	3. 在内网机器上，设置其网关为 172.16.37.39
	  vi /etc/sysconfig/network-scripts/ifcfg-eth0
	  GATEWAY=172.16.37.39
	  #重启网络服务。
	  
	4. 测试内网机器是否可以上网。
```


###centos iptables实现外网访问内网机器上的端口上的服务###

* 环境：一台带外网和内网的机器，另外几台只有内网，默认不能上网。机器都是centos系统。
* 带外网机器的外网ip为15.124.225.101， 内网ip为172.16.37.1
* 内网机器的内网ip为172.16.37.30，172.16.37.33，172.16.37.34,....

* 设置方法

```
	
	1. 所有的内网机器上，设置其网关为 172.16.37.1
	
	2. 在带外网的机器上设置iptables规则：
		iptables -t nat -A PREROUTING  -i eth0 -d 15.124.225.101  -p tcp --dport 80 -j DNAT --to-destination 172.16.37.30
		service iptables save
		/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
	3. 在带外网机器上打开转发
	   vi /etc/sysctl.conf
	   net.ipv4.ip_forward = 1
	   sysctl -p #使生效。
	   
	   
	   
	   
	centos7.1的DNAT（上面的第二步命令是centos6.5中的）： 
	配置转发（centos7.1的DNAT）
	firewall-cmd --zone=public --add-forward-port=port=50070:proto=tcp:toaddr=172.16.222.55 --permanent
	firewall-cmd --zone=public --add-forward-port=port=50030:proto=tcp:toaddr=172.16.222.55 --permanent
	firewall-cmd --zone=public --add-forward-port=port=60010:proto=tcp:toaddr=172.16.222.58 --permanent
	firewall-cmd --zone=public --add-forward-port=port=8983:proto=tcp:toaddr=172.16.222.58 --permanent
	firewall-cmd --zone=public --add-forward-port=port=50075:proto=tcp:toaddr=172.16.222.62 --permanent
	firewall-cmd --reload #重启防火墙
	
	开放端口：
	firewall-cmd --zone=public --add-port=50070/tcp --permanent
	firewall-cmd --zone=public --add-port=50030/tcp --permanent
	firewall-cmd --zone=public --add-port=60010/tcp --permanent
	firewall-cmd --zone=public --add-port=8983/tcp --permanent
	firewall-cmd --zone=public --add-port=50075/tcp --permanent
	firewall-cmd --zone=public --add-port=3306/tcp --permanent
	firewall-cmd --zone=public --add-port=11000/tcp --permanent
	firewall-cmd --reload #重启防火墙
	
```