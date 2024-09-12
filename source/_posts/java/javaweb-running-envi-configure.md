---
title: Java Web运行环境配置
author: Zeeny
comments: false
date: 2014-10-10
updated: 2014-10-10
tags: [Java,Web,Tomcat]
categories: Java
summary: Java Web运行环境配置
---


# Java Web运行环境配置

## 一、centos 6.4 x86_64 下安装Tomcat 8.0

__1. 安装JDK__

* 安装jdk-6u45-linux-x64-rpm.bin

```
chmod +x jdk-6u45-linux-x64-rpm.bin 
./jdk-6u45-linux-x64-rpm.bin
rpm -ivh jdk-6u45-linux-amd64.rpm
	 
配置环境变量:
	vi /etc/profile
		向文件里面追加以下内容:
	 		JAVA_HOME=/usr/java/jdk1.6.0_45
	 		JRE_HOME=/usr/java/jdk1.6.0_45/jre
     	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
	 		CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
	 		export JAVA_HOME JRE_HOME PATH CLASSPATH
	 
	 使修改生效:
	 	source /etc/profile   //使修改立即生效 
	 	echo $PATH   //查看PATH值
```

__2.安装Tomcat__

* 安装前提：系统必须已经安装配置了JDK6+

```
wget http://mirrors.cnnic.cn/apache/tomcat/tomcat-8/v8.0.14/bin/apache-tomcat-8.0.14.tar.gz

	将  apache-tomcat-8.0.14.tar.gz 放到 /usr/local  中。

cd /usr/local
tar -zxvf ./apache-tomcat-8.0.14.tar.gz
mv apache-tomcat-8.0.14 tomcat
```

__3.启动Tomcat__

```
[root@localhost bin]# /usr/local/tomcat/bin/startup.sh
	
打印出如下信息表示启动成功：
	Using CATALINA_BASE:   /usr/local/tomcat
	Using CATALINA_HOME:   /usr/local/tomcat
	Using CATALINA_TMPDIR: /usr/local/tomcat/temp
	Using JRE_HOME:        /usr
	Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
	Tomcat started.
```

* 防火墙开放8080端口
* 通过地址查看tomcat是否运行正常：http://172.16.37.2:8080/
* 看到tomcat系统界面，安装成功。
* 停止Tomcat：执行 /usr/local/tomcat/bin/shutdown.sh


__4.服务器配置__

* 3.1 修改端口号：
```
vi /usr/local/tomcat/conf/server.xml

找到如下内容：
	<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
	将port定义的内容修改即可。
	修改后必须重启服务。
```

* 3.2 配置虚拟目录
	每一个虚拟目录都保存了一个完整的Web项目。
	首页建立一个自己的文件夹，并在此文件夹中建立一个WEB-INF的子文件夹，同时在WEB-INF文件夹中建立一个web.xml文件，此文件的格式如下：

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" 
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
			xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
			http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" 
			version="3.1" 
			metadata-complete="true">
	<display-name>Welcome to Tomcat</display-name>
	<description>
		Welcome to Tomcat
	</description>
</web-app>
```

* 建立完项目目录后，打开Tomcat安装目录的 conf/server.xml 文件:

```
vi /usr/local/tomcat/conf/server.xml
	在 server.xml文件最后的
		<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      			</Host>
    		</Engine>
    	</Service>
	</Server>
	
	代码中间加入如下代码：
		<Context path="/demo" docBase="/mnt/java/demo" />
	
	加入后效果如下：
		<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />  
							<Context path="/demo" docBase="/mnt/java/demo" />
      			</Host>
    		</Engine>
    	</Service>
	</Server>	
	
扩展:
	1.为了开发方便，添加reloadable="true"，避免每次修改JavaBean都要重启Tomcat服务器。
	<Context path="/demo" docBase="/mnt/java/demo" reloadable="true" />
	但是这样使用服务器始将处于监视状态，运行的性能是比较低的，所以当项目真正发布动行时，一定要将reloadable的内容设置为false。
```

* 以上代码中的 Context 是一个固定的标记，表示配置虚拟目录。
* path：表示浏览器上的访问虚拟路径名称，前面必须加上“/”.
* docBase: 表示此虚拟路径名称所代表的真实路径地址。
* 可以配置多个虚拟目录，但是path不能重名。
* 配置完成后，重启Tomcat服务。在浏览器中输入：http://172.16.37.2:8080/demo/
* 如果出现如下错误界面：

```
HTTP Status 404 - /demo/

type Status report

message /demo/

description The requested resource is not available.

Apache Tomcat/8.0.14	
```

* 产生原因1：是项目缺少首页，在/mnt/java/demo下新建一个index.jsp页面即可。
* 产生原因2：在Tomcat服务器上，默认情况下是不会允许出现文件列表页的。
* 那么只需要修改 conf/web.xml 文件，使服务器允许列出目录下的文件列表，找到此文件中如下代码位置：

```
vi /usr/local/tomcat/conf/web.xml
	<servlet>
		<servlet-name>default</servlet-name>
		<servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
		<init-param>
				<param-name>debug</param-name>
				<param-value>0</param-value>
		</init-param>
		<init-param>
				<param-name>listings</param-name>
				<param-value>false</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
  </servlet>
    
	将其中的 listings 中的 false 修改为 true 。
	重启服务。
    
	此时如果没有首页，则会显示如下界面：
    
	Directory Listing For /

	Filename	Size	Last Modified
	
	Apache Tomcat/8.0.14
```