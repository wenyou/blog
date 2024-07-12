---
layout: post
title: Mac Java JDK
description: Mac Java JDK使用
category: Java
tags: [java,mac,jdk,jdk1.7]
---

#Mac Java JDK使用#

##一、Mac JDK1.7安装##

*下载：jdk-7u71-macosx-x64.dmg
* 双击安装，Oracle提供的Java都安装在/Library/Java/JavaVirtualMachines/jdk1.7.0_71.jdk目录下。
* vi ~/.bash_profile，添加如下代码

```

	JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_71.jdk/Contents/Home
	CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
	PATH=$JAVA_HOME/bin:$PATH:
	export JAVA_HOME
	export CLASSPATH
	export PATH
	
```