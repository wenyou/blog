---
layout: post
title: Maven&Nexus
description: Maven&Nexus
category: Maven
tags: [Maven,Nexus]
---
#Maven&Nexus#

* [Maven 的classifier的作用](http://blog.csdn.net/lovingprince/article/details/5894459)
* [Maven入门指南⑤：使用Nexus搭建Maven私服](http://www.cnblogs.com/luotaoyeah/p/3791966.html)
* [Nexus私服使Maven更加强大](http://blog.csdn.net/liujiahan629629/article/details/39272321)
* http://172.16.10.18:8081/nexus   admin  / zxsoftoa

```
	
		cd nexus-2.11.1-01-bundle/bin
		./nexus start
		
		ln -s /home/nexus-2.11.1-01-bundle/nexus-2.11.1-01/bin/nexus /etc/init.d/nexus
		
		chmod +x /etc/init.d/nexus
		
		service nexus stop
		service nexus start
		
		chkconfig --add nexus
		chkconfig nexus on
		chkconfig --level 345 nexus on
		
```