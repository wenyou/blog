---
title: Java Web开发笔记
author: Zeeny
comments: false
date: 2014-10-31
updated: 2014-10-31
tags: [Java,Web]
categories: Java
summary: Java Web开发笔记
---


# Java Web开发笔记

## 一、常见问题

### 1.编码问题

* Mac终端下javac中文乱码
 
```
javac -d ./WebRoot/WEB-INF/classes/ ./src/cn/stkit/zwy/demo/Register.java

出现中文乱码。
	
解决方法：
	加 -encoding 编译项，如 -encoding utf8
	
	javac -encoding utf8 -d ./WebRoot/WEB-INF/classes/ ./src/cn/stkit/zwy/demo/Register.java 
```

* 连接mysql数据库中文乱码

```	
1.org.gjt.mm.mysql.Driver 连接mysql数据库中文乱码
	private static final String DBDRIVER = "org.gjt.mm.mysql.Driver";
  private static final String DBURL = "jdbc:mysql://127.0.0.1:3306/demodb";
    	
	出现中文乱码，解决方法，添加 ?useUnicode=true&characterEncoding=utf-8，如下：
    	private static final String DBURL = "jdbc:mysql://127.0.0.1:3306/demodb?useUnicode=true&characterEncoding=utf-8";
```


### 2.javac编译时提示软件包 xxx 不存在

```
1.
	javac -encoding utf8 -d ./WebRoot/WEB-INF/classes/ ./src/cn/stkit/zwy/dao/IEmpDAO.java
	
		出错提示：./src/cn/stkit/zwy/dao/IEmpDAO.java:6: 软件包 cn.stkit.zwy.vo 不存在
			import cn.stkit.zwy.vo.Emp;
			
	解决方法：
		编译时加 -classpath 类路径
		javac -classpath ./src/ -encoding utf8 -d ./WebRoot/WEB-INF/classes/ ./src/cn/stkit/zwy/dao/IEmpDAO.java
	
2.
	javac -classpath "./src/:/Users/zxsoft/USVN/Java/demo/WebRoot/WEB-INF/lib/commons-io-2.4.jar:/Users/zxsoft/USVN/Java/demo/WebRoot/WEB-INF/lib/commons-fileupload-1.3.1.jar" -encoding utf8 -d ./WebRoot/WEB-INF/classes/ ./src/cn/stkit/zwy/util/FileUploadTools.java
	
	出错提示：
		软件包 javax.servlet.http 不存在
				import javax.servlet.http.HttpServletRequest
	
	解决方法：
		把tomcat的lib目录下的“servlet-api.jar”加入到classPath中。
		javac -classpath "./src/:/usr/local/tomcat/lib/servlet-api.jar:/Users/zxsoft/USVN/Java/demo/WebRoot/WEB-INF/lib/commons-io-2.4.jar:/Users/zxsoft/USVN/Java/demo/WebRoot/WEB-INF/lib/commons-fileupload-1.3.1.jar" -encoding utf8 -d ./WebRoot/WEB-INF/classes/ ./src/cn/stkit/zwy/util/FileUploadTools.java
```

### 3.java命令运行程序时，提示java.lang.NoClassDefFoundError

```
java cn.stkit.zwy.dao.test.TestDAOInsert
	提示：Exception in thread "main" java.lang.NoClassDefFoundError: cn/stkit/zwy/dao/test/TestDAOInsert
	
	解决方案：
		方法1、进入 “cn.stkit.zwy.dao.test.TestDAOInsert” 所在目录，cd /Users/zxsoft/USVN/Java/demo/WebRoot/WEB-INF/classes
		再执行：java cn.stkit.zwy.dao.test.TestDAOInsert
	
	方法2：
		加 -classpath 
		java -classpath /Users/zxsoft/USVN/Java/demo/WebRoot/WEB-INF/classes/ cn.stkit.zwy.dao.test.TestDAOInsert	
	
	扩展，-classpath （-cp）运行时临时指定多个路径
		java -cp ".:/Users/zxsoft/USVN/Java/demo/WebRoot/WEB-INF/lib/mysql-connector-java-5.1.33-bin.jar" cn.stkit.zwy.dao.test.TestDAOInsert
	
		如上解决了：java.lang.ClassNotFoundException: org.gjt.mm.mysql.Driver 问题
	
	
	
其他提示：
	1. 不含包层次的HelloWorld.java
		public class HelloWorld {  
			public static void main(String args[]){  
				System.out.println("Hello World!");  
			}  
		} 
		保存在/home/javatest/下，使用javac命令编译:
		javac HelloWorld.java 
		过行：java HelloWorld 
	
	2.包含包层次的HelloWorld.java
		package cn.stkit.demo;
		public class HelloWorld {  
			public static void main(String args[]){  
				System.out.println("Hello World!");  
			}  
		} 
		保存在/home/javatest/下。
		编译时有两种方法:
			2.1) 直接编译: javac HelloWorld.java 
				此时在当前目录下输出HelloWorld.class。此时，运行不能使用上面相同的方法，如果使用了：java HelloWorld
	运行时，会出现如下错误：
					Exception in thread "main" java.lang.NoClassDefFoundError: HelloWorld (wrong name: cn/stkit/demo/HelloWorld)
				从上述错误信息可以看到，系统可以找到HelloWorld类（因为当前路径包含在CLASSPATH中），但这个类属于cn.stkit.demo包。所以，要做的就是按照上述包层次，相应的创建目录层次，把上面生成的HelloWorld.class放到/home/javatest/cn/stkit/demo/目录下。HelloWorld.java在/home/javatest/目录下。运行：
					java cn.stkit.demo.HelloWorld 
				注意：这儿要注意的是，不能使用java cn\stkit\demo\HelloWorld来运行，此时同样会出现上面的错误。
	
			2.2) 使用 -d <directory>编译选项
					使用-d <directory>编译选项在当前路径（或任意指定的路径）下生成包层次。
					javac -d . HelloWorld.java
					此时，在当前目录(/home/javatest/)下就生成了一个cn/stkit/demo目录(/home/javatest/cn/stkit/demo)，并且输出的.class文件也在里面。运行：
						java org.myorg.HelloWorld
```

### 4.web.xml中，listener， filter， servlet加载顺序

* 加载配置顺序：listener， filter， servlet
* Listener配置信息必须在Filter和Servlet配置之前，Listener的初始化（ServletContextListener初始化）比Servlet和Filter都优先，而销毁比Servlet和Filter都慢。
* web.xml 的加载顺序是：context-param -> listener -> filter -> servlet
* 加载顺序与它们在 web.xml 文件中的先后顺序无关。


### 5. Server Tomcat v7.0 Server at localhost was unable to start within 45 seconds

```
修改 workspace/.metadata/.plugins/org.eclipse.wst.server.core/servers.xml
把其中的start-timeout="45" 改为  start-timeout="100" 或者更长。
最后重启eclipse就可以了。
```


## 二、常用命令

### 1、jar命令

* 打包生成.jar文件

```
jar -cvf wordcount.jar -C WordCount/ .
	将当前目录下的 WordCount目录下的所有文件（通常是.class文件）打包生成 wordcount.jar 文件，并放在当前目录下。
	
jar -cvf classes.jar Foo.class Bar.class
	将两个类文件归档到一个名为 classes.jar 的归档文件中。
```


## 三、web.xml 配置中classpath: 与classpath*:的区别？ 

* classpath是指 WEB-INF文件夹下的classes目录
	
* classes含义：
	
		1. 存放各种资源配置文件 eg.init.properties log4j.properties struts.xml
		2. 存放模板文件 eg.actionerror.ftl
		3. 存放class文件 对应的是项目开发时的src目录编译文件
		4. 总结：这是一个定位资源的入口
		
* lib和classes下文件访问优先级的问题: lib>classes
	
* classpath 和 classpath* 区别：
		
* classpath：只会到你的class路径中查找找文件;
		 
* classpath*：不仅包含class路径，还包括jar文件中(class路径)进行查找.


### 启动tomcat后出现红色警告:[SetPropertiesRule]{Server/Service/Engine/Host/Context} Setting property 'source' to 'org.eclipse.js ....' did not find a matching property 问题？

* 这是因为我们在eclipse下，通过tomcat部署web工程时，tomcat的配置文件server.xml中会自动生成一个关于该web工程的配置信息，类似于下面的东西：
	  
		`<Context docBase="webPoject" path="/stkit" reloadable="true" source="org.eclipse.jst.jee.server:stkit"/>`
		
* 而默认情况下，server.xml的 Context元素不支持名称为source的属性，所以会发出警告。

* 解决办法是：关闭tomcat，双击eclipse下tomcat服务器，在出来的Tomcat server at localhost页面中找到server options选项，选中其中的选项”Publish modual contexts to separat XML files“，ctr+s，启动tomcat。
	
	
### asm（spring-asm-3.1.4.RELEASE.jar）导致的org.springframework.beans.factory.BeanDefinitionStoreException错误。

		五月 21, 2015 7:02:05 下午 org.springframework.web.servlet.DispatcherServlet initServletBean
		严重: Context initialization failed
		org.springframework.beans.factory.BeanDefinitionStoreException: Failed to read candidate component class: file [/usr/local/tomcat/webapps/STKIT/WEB-INF/classes/cn/stkit/web/cp/LoginController.class]; nested exception is java.lang.IncompatibleClassChangeError: class org.springframework.core.type.classreading.ClassMetadataReadingVisitor has interface org.springframework.asm.ClassVisitor as super class
	
		aused by: java.lang.IncompatibleClassChangeError: class org.springframework.core.type.classreading.ClassMetadataReadingVisitor has interface org.springframework.asm.ClassVisitor as super class
		at java.lang.ClassLoader.defineClass1(Native Method)
		
		
* 解决方案：删除 spring-asm-3.1.4.RELEASE.jar 包。

* 在 spring-core-4.1.6 的jar包中已经包含了asm，所以这时候再导入spring-asm-xxxx.jar 就会出现这个问题，开发时需注意版本之间的差别，jar包最好套用同一个版本。
	