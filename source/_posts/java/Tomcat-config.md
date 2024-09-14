---
title: Tomcat配置
author: Zeeny
comments: false
date: 2015-10-16
updated: 2015-10-16
tags: [Java,Tomcat]
categories: [Java]
summary: Tomcat配置
---

# Tomcat配置

## 1、Tomcat配置Java运行虚拟机内存

```
配置文件：catalina.bat 或 startup.bat (windows)

配置内容：JAVA_OPTS="-server -Xms256m -Xmx512m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize:256m"
```

## 2、Tomcat初始化配置

### 2.1 关闭服务器端口

* 在server.xml文件里
```
<Server port="8005" shutdown="SHUTDOWN"> 处：
修改8005端口号和shutdown命令，改成不易猜测的命令，防止别人通过telnel命令远程关闭tomcat服务。
```

### 2.2 隐藏版本信息

*  版本信息在 tomcat/lib 目录下的 catalina.jar 包文件，位置在catalina.jar/org/apache/catalina/util/ServerInfo.properties文件中。

### 2.3 禁用Tomcat管理页面

*  法1：改名或者删除webapps目录下的ROOT目录，新建一个空的ROOT目录。

### 2.4 自定义错误页面

* 新建自己的错误页面，配置web.xml文件，写入内容如下：

```
<error-page>
  <error-code>400</error-code>
  <location>/error.html</location>
</error-page>

<error-page>
  <error-code>404</error-code>
  <location>/error.html</location>
</error-page>

<error-page>
  <error-code>500</error-code>
  <location>/error.html</location>
</error-page>
```


### 2.5 AJP端口管理   AJP协议

* 如果tomcat前端用的是apache服务器的时候就会使用AJP连接器。

* 如果没有使用apache进行 tomcat服务连接，就把AJP端口注释掉。

* 在server.xml文件中注释。

### 2.6 启用cookie的HttpOnly

*  配置conf/context.xml 文件
```
在<Context>...</Context>处加上useHttpOnly="true",
变成：<Context useHttpOnly="true">...</Context>。
```

## 3、安全规范

### 3.1 账号管理、认证授权
* 共享、无关账号
    conf/tomcat-users.xml

* 口令密码
    复杂，定期修改

* 用户权限
    最小权限

### 3.2 日志配置操作
* conf/server.xml

```
此处：
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
prefix="localhost_access_log" suffix=".txt"
pattern="%h %l %u %t &quot;%r&quot; %s %b" />
```

## 4、优化配置

### 4.1 缓存优化(nginx、gzip)

### 4.2 运行模式
* BIO：tomcat7以下默认模式。
  效率比较低。


* NIO：基于缓存区、非阻塞的I/O。
  java一般选择这个模式。


* APR：tomcat7及以上默认模式。