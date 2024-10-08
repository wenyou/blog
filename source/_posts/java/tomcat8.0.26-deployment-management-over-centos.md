---
title: Tomcat8.0.26 over Centos部署与管理
author: Zeeny
comments: false
date: 2015-09-01
updated: 2015-09-01
tags: [Tomcat,Centos]
categories: [Java,Tomcat]
summary: Tomcat8.0.26 over Centos部署与管理
---


# Tomcat8.0.26 over Centos部署与管理

## 一、安装部署

1. 安全起见，本着最小权限原则，生产系统决不允许使用root账户来运行tomcat。为此，建立新账户tomcat，并设定登录密码。

```
useradd zxtomcat
passwd zxtomcat   [123456]
```

2. 下载tomcat8.0.26: wget http://apache.fayea.com/tomcat/tomcat-8/v8.0.26/bin/apache-tomcat-8.0.26.tar.gz

3. 解压：tar zxvf apache-tomcat-8.0.26.tar.gz

4. 修改server.xml,加上 URIEncoding="UTF-8"，防止URL中文乱码。

```
<Connector URIEncoding="UTF-8" port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />	
```

5. 开放80端口：

```
/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
/etc/rc.d/init.d/iptables save
/etc/rc.d/init.d/iptables restart
```

6. 部署war包：

```
把IFEPA.war放在 apache-tomcat-8.0.26/webapps/ 目录下。
	修改server.xml：
	添加：
		<Service name="Catalina2">
			<Connector URIEncoding="UTF-8" connectionTimeout="20000" port="80" protocol="HTTP/1.1" redirectPort="8444"/>
			<Connector port="8089" protocol="AJP/1.3" redirectPort="8444"/>
			<Engine defaultHost="localhost" name="Catalina2">
				<Host appBase="" autoDeploy="true" name="localhost" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
						<Context docBase="/home/apache-tomcat-8.0.26/webapps/VIBAP-WEB" path="" reloadable="true" workDir="/home/apache-tomcat-8.0.26/webapps/VIBAP-WEB"/>
					</Host>
			</Engine>
		</Service>
```

7. 启动、关闭tomcat

```
apache-tomcat-8.0.26/bin/startup.sh 
apache-tomcat-8.0.26/bin/shutdown.sh
```