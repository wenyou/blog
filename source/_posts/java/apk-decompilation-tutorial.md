---
title: APK反编译源码教程
author: Zeeny
comments: false
date: 2015-11-14
updated: 2015-11-14
tags: [Java,Android,APK]
categories: [Java,Tomcat]
summary: APK反编译源码教程
---


# APK反编译源码教程

### 第一步

首先我们直接用一个解压apk（开发过android应该知道apk其实就是个压缩文件）,解压之后拷贝出里面classes.dex文件待用。

### 第二步

* 下载dex2jar工具，最新版下载链接[dex2jar下载](http://sourceforge.net/projects/dex2jar/),
* 解压之后，打开cmd，进入解压目录，运行命令：

```
d2j-dex2jar.bat classes.dex(上一步解压的) jarpath(反编译dex后的文件目录)
	
example:
	d2j-dex2jar.bat c:\user\qting\classes.dex c:\user\qting\*
	
	反编译之后，会得到一个classes-dex2jar.jar文件，待用。
```

### 第三步

* 下载JD-GUI(反编译jar神器)，最新版下载链接[JD-GUI下载](http://www.softpedia.com/get/Programming/Debuggers-Decompilers-Dissasemblers/JD-GUI.shtml)
* 解压之后，双击打开，直接把上一步得到的的classes-dex2jar.jar文件直接拖入JD-GUI里面，你就可以随意查看蜻蜓的源码了。
