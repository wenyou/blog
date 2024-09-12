---
title: Maven Nexus
author: Zeeny
comments: false
date: 2015-05-06
updated: 2015-05-06
tags: [Java,Maven]
categories: [Java]
summary: Maven Nexus
---


# Maven Nexus

* http://172.xx.xx.xx:8081/nexus

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