---
title: Jboss服务管理、Mac安装maven
author: Zeeny
comments: false
date: 2014-11-27
updated: 2014-11-27
tags: [Jboss,Mac,Maven]
categories: Java
summary: Jboss服务管理、Mac安装maven
---


# Jboss-AS-7.1.1.Final的管理

## Jboss服务管理

__1、启动standalone__

* /home/jboss-as-7.1.1.Final/bin/standalone.sh

```
/home/jboss-as-7.1.1.Final/bin/standalone.sh
/home/jboss-as-7.1.1.Final/bin/standalone.sh &
/home/jboss-as-7.1.1.Final/bin/standalone.sh >/dev/null &
```

## Eclipse luna Jboss插件

* jbossTools - http://download.jboss.org/jbosstools/updates/development/luna/


## 安装Jboss

```
1) 下载 jboss-as-7.1.1.Final.tar.gz

2) tar zxf jboss-as-7.1.1.Final.tar.gz
	    mv jboss-as-7.1.1.Final /home/

3) vi /etc/profile
		JBOSS_HOME=/home/jboss-as-7.1.1.Final

4) 修改standalone/configuration/standalone.xml配置文件
	找到：
		<interface name="public">
			<inet-address value="${jboss.bind.address:127.0.0.1}"/>
		</interface>
		把127.0.0.1 改成	所在机器的IP地址（172.16.37.27），使用局域网能够访问，否则在局域网内不能通过172.16.37.27:8080打开网页。
        
5) 关闭防火墙或开放8080端口
	/sbin/iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
    	
6) 创建管理员用户
		cd /home/jboss-as-7.1.1.Final/bin
		./add-user.sh
			出现如下提示：
				What type of user do you wish to add? 
				a) Management User (mgmt-users.properties) 
				b) Application User (application-users.properties)
				(a): 
    		直接回车
    		又出现如下提示：
					Enter the details of the new user to add.
					Realm (ManagementRealm) : #直接回车
					Username : admin       #输入管理员用户名，默认admin
					Password : zxjboss #输入管理员密码
					Re-enter Password : zxjboss
    	
    	管理员控制台访问：
        http://127.0.0.1:9990/console
    	如果也想在其他局域网机器上访问管理员控制台，防火墙需要开放端口9990，修改standalone.xml
    		vi standalone/configuration/standalone.xml
    		找到：
					<interface name="management">
						<inet-address value="${jboss.bind.address.management:127.0.0.1}"/>
					</interface>
        把127.0.0.1改为jboss所在机器（本机）的IP地址172.16.37.27。
		
7) 启动服务
	/home/jboss-as-7.1.1.Final/bin/standalone.sh
	
8) 设置虚拟目录
	8.1）修改${JBOSS_HOME}/standalone/configuration/standalone.xml文件
		找到：<subsystem xmlns="urn:jboss:domain:deployment-scanner:1.1"> 节点处：
			<subsystem xmlns="urn:jboss:domain:deployment-scanner:1.1">
				<deployment-scanner path="deployments" relative-to="jboss.server.base.dir" scan-interval="5000"/>
				<deployment-scanner name="demo" path="/home/web/deployments" scan-enabled="true" scan-interval="5000" auto-deploy-zipped="false" auto-deploy-exploded="false" deployment-timeout="60" />
			</subsystem>
        
			#<deployment-scanner name="demo" 部份就是新增的虚拟目录新位置。
			/home/web/deployments 必须是存在的目录，其有可写权限。
		
	8.2）在新的虚拟目录位置/home/web/deployments放置一个demo.war目录，和一个文件demo.war.deployed。
		其中demo.war里面放置你的应用，demo.war.deployed是一个空文件。
		
	8.3）修改${JBOSS_HOME}/standalone/configuration/standalone.xml文件
		找到：<subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="false"> 节点处：
		内容如下：
			<subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="false">
				<connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"/>
				<virtual-server name="default-host" enable-welcome-root="true">
					<alias name="localhost"/>
					<alias name="example.com"/>
				</virtual-server>
				##增加
				<virtual-server name="/demo" enable-welcome-root="false" default-web-module="demo.war">
					<alias name="localhost"/>
				</virtual-server>
			</subsystem>
		
			就可以通过http://localhost:8080/demo/访问了。
	
		
9) JBoss AS 7 虚拟主机配置
	新建zxweb.war WEB应用程序：/home/jboss-as-7.1.1.Final/standalone/deployments/zxweb.war
	新建zxweb.war.dodeploy：touch /home/jboss-as-7.1.1.Final/standalone/deployments/zxweb.war.dodeploy,内容为空即可。
		9.1) 在应用程序（如zxweb）的WEB-INF下新建jboss-web.xml文件
			vi /home/jboss-as-7.1.1.Final/standalone/deployments/zxweb.war/WEB-INF/jboss-web.xml
	    文件内容:
	    	<!DOCTYPE jboss-web PUBLIC "-//JBoss//DTD Web Application 5.0//EN" "http://www.jboss.org/j2ee/dtd/jboss-web_5_0.dtd">
				<jboss-web>
        	<context-root>/</context-root>
       		<!-- For session copy -->
        	<replication-config>
						<cache-name>standard-session-cache</cache-name>
        	</replication-config>
        	<!-- For session time -->
        	<max-active-sessions>20</max-active-sessions>
        	<passivation-config>
						<use-session-passivation>true</use-session-passivation>
						<passivation-min-idle-time>60</passivation-min-idle-time>
						<passivation-max-idle-time>600</passivation-max-idle-time>
        	</passivation-config>
				</jboss-web>
		
		9.2) 修改${JBOSS_HOME}/standalone/configuration/standalone.xml文件
			找到：<subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="false"> 节点处：
			内容如下：
				<subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="zxweb.com" native="false">
					<connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"/>
					<virtual-server name="default-host" enable-welcome-root="true">
						<alias name="localhost"/>
					</virtual-server>
					<virtual-server name="zxweb.com" enable-welcome-root="false" default-web-module="zxweb.war">
						<alias name="www.zxweb.com"/>
					</virtual-server>
        </subsystem>
        修改default-virtual-server的值为你新配置虚拟主机的name,由于jboss-web.xml配置了<context-root>/</context-root>，所以这里zxweb.com中应配置enable-welcome-root="false"，以使用zxweb.war项目的主页作为服务器欢迎页。
        
    9.3) 最后修改host文件，以standalone.sh启动JBoss服务器即可通过www.zxweb.com访问了。
        
        
        
10) 使用jboss7.1.1Final版本部署web项目，怎么样配置能够在访问时不用带上项目名?
	法一、修改配置文件,把Jboss 默认启动的项目替换成自己的。在自己的应用里面的WEB-INF文件夹下增加jboss-web.xml文件。文件内容是：
		<?xml version="1.0" encoding="UTF-8"?>
		<jboss-web>
			<context-root>/</context-root>
		</jboss-web>
		
		在JBoss standalone下找到配置文件
		修改 standalone.xml  把enable-welcome-root="true"  改成 enable-welcome-root="false"
			<subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="false">
				<connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"/>
				<virtual-server name="default-host" enable-welcome-root="false">
					<alias name="localhost"/>
					<alias name="example.com"/>
				</virtual-server>
			</subsystem>
        
	法二、
		把项目改名成welcome-content.war，这个就跟tomcat一样的。
		修改 standalone.xml，把enable-welcome-root="true"  改成 enable-welcome-root="false"。
```

### 常见问题

__1__ 
	Did not receive a response to the deployment operation within the allowed timeout period [60 seconds]. Check the server configuration file and the server logs to find more about the status of the deployment.

```
加 deployment-timeout="1200"
vi standalone.xml
	
	<subsystem xmlns="urn:jboss:domain:deployment-scanner:1.1">
		<deployment-scanner path="deployments" relative-to="jboss.server.base.dir" scan-interval="5000" deployment-timeout="1200" />
	</subsystem>
```

__2__ Boss7如何设置URI编码，JBOSS7中文乱码解决

```
standalone\configuration的standalone.xml文件中 <extensions> </extensions> 节点之后插入如下：
	<system-properties>
		<property name="org.apache.catalina.connector.URI_ENCODING" value="UTF-8"/>
		<property name="org.apache.catalina.connector.USE_BODY_ENCODING_FOR_QUERY_STRING" value="true"/>
	</system-properties>
``` 


```
tomcat 设置URI编码  解决中文乱码
	
	修改server.xml文件，加入 URIEncoding="UTF-8" ,如下所示:
		<Connector URIEncoding="UTF-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
	
		<Service name="Catalina2">
    <Connector URIEncoding="UTF-8" connectionTimeout="20000" port="8088" protocol="HTTP/1.1" redirectPort="8444"/>
```

__3__ Spring MVC 字符编码过滤器  解决中文乱码

```
web.xml 
	
	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
		<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```

__4__ 在out.print前添加response.setContentType 解决中文乱码

```
response.setContentType("text/html;charset=UTF-8");
        
PrintWriter out = response.getWriter();
    
out.print("我是中文");
```


# Mac安装maven

* 下载 apache-maven-3.2.3-bin.tar.gz

```
1) tar zxf apache-maven-3.2.3-bin.tar.gz

2) mv apache-maven-3.2.3 /usr/local/maven/maven-3.2.3

3) vi ~/.bash_profile

	追回如下内容：
		M3_HOME = /usr/local/maven/maven-3.2.3
		PATH = $PATH:$M3_HOME/bin
		export M3_HOME
		export PATH
	
4) 新打开一个终端窗口:
	通过
	echo $M3_HOME
	echo $PATH
	可查看刚设置的环境变量。
	
5) mvn -version
	
6) 重新设置本地Repository的位置
	vi /usr/local/maven/maven-3.2.3/conf/settings.xml
		在这一段：
		<!-- localRepository
			| The path to the local repository maven will use to store artifacts.
			|
			| Default: ${user.home}/.m2/repository
			<localRepository>/path/to/local/repo</localRepository>
			-->
			后面添加：
			<localRepository>/Users/zxsoft/USVN/Java</localRepository>
		
7) 创建一个简单的Java工程：
	 mvn archetype:create -DgroupId=com.cnzxsoft.myapp -DartifactId=myapp

8) 创建一个java的web工程
	 mvn archetype:create -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-webapp -DgroupId=com.cnzxsoft.vibap -DartifactId=zxweb
	 
	 打包：cd zxweb
	      mvn package
	 
9) 常用命令
	 	打包：mvn package
	 	编译：mvn compile
		编译测试程序：mvn test-compile
		清空：mvn clean
		运行测试：mvn test
		生成站点目录: mvn site
		生成站点目录并发布：mvn site-deploy
		安装当前工程的输出文件到本地仓库: mvn install
		安装指定文件到本地仓库：mvn install:install-file -DgroupId=<groupId> -DartifactId=<artifactId> -Dversion=1.0.0 -Dpackaging=jar -Dfile=<myfile.jar>
		查看实际pom信息: mvn help:effective-pom
		分析项目的依赖信息：mvn dependency:analyze 或 mvn dependency:tree
		跳过测试运行maven任务：    mvn -Dmaven.test.skip=true XXX
		生成eclipse项目文件: mvn eclipse:eclipse
		查看帮助信息：mvn help:help 或 mvn help:help -Ddetail=true
		查看插件的帮助信息：mvn <plug-in>:help，比如：mvn dependency:help 或 mvn ant:help 等等。
```
