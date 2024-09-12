---
title: Spring-framework-4.1.2使用
author: Zeeny
comments: false
date: 2014-12-02
updated: 2014-12-02
tags: [Java,Spring]
categories: [Java,Spring]
summary: Spring-framework-4.1.2使用
---


# Spring-framework-4.1.2使用

```
Spring-framework source code
git url: git clone git://github.com/SpringSource/spring-framework.git

导入IDE
	Run ./import-into-eclipse.sh or read import-into-idea.md as appropriate.
	启动 ./import-into-eclipse.sh 时，出现如下错误：
		Unrecognized VM option ‘MaxMetaspaceSize=1024m’
		Error: Could not create the Java Virtual Machine.
		Error: A fatal exception has occurred. Program will exit.

	这是因为”MaxMetaspaceSize=1024m” 这个参数配置只出现在jdk 8中，默认情况下是在master分支下，猜测是基于jdk 8开发。

所以有三个选择：
	1. 切换到其他分支，如3.2.X
	2. 安装jdk8
	3. 编辑gradlew(gradlew.bat windows)，移去VM option MaxMetaspaceSize.
```


## 编译Spring-framework

```
cd spring-framework-4.1.2.RELEASE
./gradlew build
```

## Spring-framework源码导入到Eclise

```
cd spring-framework-4.1.2.RELEASE
./import-into-eclipse.sh
```
* 在Eclipse菜单File->import选择General下面的Exsiting projects into workspace，这样就导入完成了。
* 需要安装jdk1.8版本

