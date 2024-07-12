---
layout: post
title: Spring-framework-4.1.2使用
description: Spring-framework-4.1.2使用
category: Spring
tags: [Spring]
---

#Spring-framework-4.1.2使用#

##一、参考资料##

* [Resolving Spring artifacts](https://github.com/spring-projects/spring-framework/wiki/Downloading-Spring-artifacts#resolving-spring-artifacts)
* [The Central Repository Search Engine](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20v%3A%224.1.2.RELEASE%22)
* [Downloading Spring artifacts](https://github.com/spring-projects/spring-framework/wiki/Downloading-Spring-artifacts)
* [编译 Spring-framework的经验分享](http://www.th7.cn/Program/java/201410/301614.shtml)
* [Unrecognized VM option MaxMetaspaceSize=1024m](http://blog.sina.com.cn/s/blog_4abbf0ae0101ehy3.html)

```

	Spring-framework source code
	git url: git clone git://github.com/SpringSource/spring-framework.git

	导入IDE
	Run ./import-into-eclipse.sh or read import-into-idea.md as appropriate.
	启动 ./import-into-eclipse.sh 时，出现如下错误

	Unrecognized VM option ‘MaxMetaspaceSize=1024m’
	Error: Could not create the Java Virtual Machine.
	Error: A fatal exception has occurred. Program will exit.

	这是因为”MaxMetaspaceSize=1024m” 这个参数配置只出现在jdk 8中，默认情况下是在master分支下，猜测是基于jdk 8开发。

	所以有三个选择：
	1. 切换到其他分支，如3.2.X
	2. 安装jdk8
	3. 编辑gradlew(gradlew.bat windows)，移去VM option MaxMetaspaceSize.
	
```

* [spring libs release](http://repo.spring.io/libs-release/)


##二、编译Spring-framework##

* cd spring-framework-4.1.2.RELEASE
* ./gradlew build

##三、Spring-framework源码导入到Eclise##

* cd spring-framework-4.1.2.RELEASE
* ./import-into-eclipse.sh
* 在Eclipse菜单File->import选择General下面的Exsiting projects into workspace，这样就导入完成了。
* 需要安装jdk1.8版本
* [将spring framework源码导入Spring Tool Suite中](http://wind-bell27.iteye.com/blog/1969930)

##四、常见问题##

* [applicationContext.xml 配置文件的存放位置](http://www.cnblogs.com/wanggd/archive/2013/05/20/3087972.html)


