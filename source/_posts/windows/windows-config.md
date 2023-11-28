---
title: Windows常用配置、开发配置
author: Zeeny
comments: false
date: 2015-05-15 22:47:05
updated: 2015-05-15 22:47:05
tags: [windows,cmd,route]
categories: windows
summary: Windows常用配置、开发配置
---

# 一、windows 转发

**1、添加端口转发策略** 

​	&emsp;&emsp;以下是cmd命令，请用管理员模式执行：

```
netsh interface portproxy show all

netsh interface portproxy add v4tov4 13306 127.0.0.1 3306 192.168.1.101
```

​	&emsp;&emsp;解释：netsh interface portproxy add v4tov4 [监听端口] [转发地址] [转发端口] [监听地址]

**2、删除已有的netsh端口转发策略**

```
netsh interface portproxy delete v4tov4 listenport=xxx

EG：netsh interface portproxy add v4tov4 9002 10.0.101.84 9002 127.0.0.1
```



# 二、cmd静态路由设置方法

__问题抛出：__

​	&emsp;&emsp;现在是这种情况，我的windows电脑，IP地址如下所示：

![2015-05-15-cmd-route-1.png](/images/2015-05-15-cmd-route-1.png)

​	&emsp;&emsp;现在我要访问 `172.37.37.27` 这个地址，如果要访问这个地址的话，windows电脑的IP地址，也就是网关必须设成 `172.16.18.1` 才可以访问，如下图所示：

![2015-05-15-cmd-route-2.png](/images/2015-05-15-cmd-route-2.png)

​	&emsp;&emsp;而把网关设为 `172.16.18.1` 的话，我就不能访问外网了。如何设置才能既可以访问`172.37.37.27` 又可以访问外网呢？

__解决方案：__

​	&emsp;&emsp;打开windows下的cmd窗口：


	#输入：
	route add 0.0.0.0 mask 0.0.0.0 172.16.18.1
	# 执行即可。
	#测试, 看是否能通?
	ping 172.37.37.27 
	#也可执行,命令查看路由表:
	route print 


​	

**案例：**

​	&emsp;&emsp;让192.168.3.x网段地址的电脑可以透过网关172.16.1.1访问另外一台可以把网关设为172.16.1.1也能上网的电脑。解决的问题是，我在三楼会议室笔记本分配的ip是192.168.3.15，网关是192.168.3.1，我想访问我在四楼的电脑(ip:172.16.37.11， gateway:172.16.18.1)，这台电脑也可以把172.16.1.1当网关，现在的需求是需要它既能继续使用172.16.18.1网关，又能通过172.16.1.1让192.168.3.15访问。

```
#打开172.16.37.11的cmd窗口：
#输入：
route add 192.168.3.0 mask 255.255.255.0 172.16.1.1
# 执行即可。
#测试: 
ping 192.168.3.15 
# 看是否能通。
#也可执行：
route print 
# 命令查看路由表。
```

