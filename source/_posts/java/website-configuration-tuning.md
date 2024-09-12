---
title: 网站最大并发连接数设置
author: Zeeny
comments: false
date: 2015-11-21
updated: 2015-11-21
tags: [Java,Web,Tomcat]
categories: [Java,Web]
summary: 网站最大并发连接数、新建数配置调优
---


# 网站最大并发连接数、新建数配置调优

### Tomcat修改最大并发连接数

```
vi conf/server.xml
	两者的默认值分别是200和100，要调整Tomcat的默认最大连接数，可以增加这两个属性的值，并且使acceptCount大于等于maxThreads：
		<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" acceptCount="2000" maxThreads="2000" />  


vi bin/catalina.sh 
	JAVA_OPTS='-Xms1024m -Xmx2048m'
	JAVA_OPTS="$JAVA_OPTS -server -XX:PermSize=128M -XX:MaxPermSize=512m"
    
    
查看tomcat的连接数
  netstat -na | grep ESTAB | grep 80 | wc -l
  netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

### Linux并发

```
使用ulimit命令查看系統允許當前用戶進程打開的文件數限制：
	ulimit -n

调整文件数：
	linux系统优化完网络必须调高系统允许打开的文件数才能支持大的并发，默认1024是远远不够的。

执行命令：
	Shell代码
	echo ulimit -HSn 65536 >> /etc/rc.local
	echo ulimit -HSn 65536 >>/root/.bash_profile
	ulimit -HSn 65536
	
	
优化Linux内核参数:
	vi /etc/sysctl.conf
		在末尾增加以下内容：
		# Add
		net.ipv4.tcp_max_syn_backlog = 65536
		net.core.netdev_max_backlog =  32768
		net.core.somaxconn = 32768

		net.core.wmem_default = 8388608
		net.core.rmem_default = 8388608
		net.core.rmem_max = 16777216
		net.core.wmem_max = 16777216

		net.ipv4.tcp_timestamps = 0
		net.ipv4.tcp_synack_retries = 2
		net.ipv4.tcp_syn_retries = 2

		net.ipv4.tcp_tw_recycle = 1
		#net.ipv4.tcp_tw_len = 1
		net.ipv4.tcp_tw_reuse = 1

		net.ipv4.tcp_mem = 94500000 915000000 927000000
		net.ipv4.tcp_max_orphans = 3276800

		#net.ipv4.tcp_fin_timeout = 30
		#net.ipv4.tcp_keepalive_time = 120
		net.ipv4.ip_local_port_range = 1024  65535
		
使配置立即生效：
	/sbin/sysctl -p
```