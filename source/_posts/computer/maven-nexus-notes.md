---
title: Maven Nexus，Maven常见问题，maven私服与eclipse配置
author: Zeeny
comments: false
date: 2015-05-06
updated: 2015-05-06
tags: [Nexus,Maven]
categories: [计算机,开发工具]
summary: Maven Nexus，Maven常见问题，maven私服与eclipse配置
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

# Maven常见问题

### 1. eclipse新建maven项目报错：

```
missing org.apache.maven.archetypes:maven-archetype-quickstart:pom。。。
```
* maven安装包下配置文件conf/settings.xml本地私服地址写错了。问题和下面一个问题一样。

### 2. 安装maven所见错误No plugin found for prefix 'help' in the current project and in the plugin

* 问题出在了镜像服务器地址上，下载不了jar包，把maven仓库的镜像地址修改了就行了；下面是几个常用的maven长仓库的镜像地址：
http://repo1.maven.org/maven2 （这个仓库最全，但有一点慢）
http://maven.apache.org/download.cgi
http://mvnrepository.com/ (这个仓库速度最快，国内有镜像服务器)

* 修改成以上地址后，检查maven的环境变量是否配置，然后再在cmd命令面板中输入mvn help:system命令检测是否成功。


# Maven私服与eclipse配置

## 1、安装jdk

```
配置windows系统环境变量：JAVA_HOME, PATH,CLASSPATH

CLASSPATH=.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar
```

## 2、安装eclipse

## 3、安装maven

* 下载apache-maven-3.5.2-bin.zip，解压放在c:\apache-maven-3.5.2下。

* 配置windows系统环境变量：MAVEN_HOME, PATH

```
MAVEN_HOME=c:\apache-maven-3.5.2

PATH追加：;%MAVEN_HOME%\bin
```

## 4.修改配置maven配置文件settings.xml

`c:\apache-maven-3.5.2\conf\settings.xml`

* A、在<mirrors>中添加私服地址：
```
<mirror>
  <id>public</id>
  <mirrorOf>*</mirrorOf>
  <name>central-mirror</name>
  <url>http://192.168.156.253:8081/nexus/content/groups/public/</url>
</mirror>
```

* B、在<profiles>添加私服地址：

```
<profile>
  <repositories>
    <repository>
      <id>central</id>
      <name>central</name>
      <url>http://192.168.156.253:8081/nexus/content/groups/public/</url>
      <layout>default</layout>
      <releases>
        <enabled>true</enabled>
      </releases>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    </repository>
  </repositories>
</profile>
```

* C、在配置文件后面</settings>结束符号之前添加激活项：

```
<activeProfiles>
  <activeProfile>central</activeProfile>
</activeProfiles>
```

## 5、配置eclipse与本地maven相结合

* A、eclipse->Window->Preferences->Maven->Archetypes->Add Remote Catalog
<p>打开Remote Archetype Catalog对话框，配置如下：</p>
```
Catalog file:http://192.168.156.253:8081/nexus/content/repositories/repo1/archetype-catalog.xml
Description: sifu-maven-catalog   随便写个。
```

* B、eclipse->Window->Preferences->Maven->Installations->Add
<p>打开New Maven Runtime对话框，配置如下：</p>
```
Installation home: c:\apache-maven-3.5.2
```

* C、eclipse->Window->Preferences->Maven->User Settings
<p>配置如下：</p>
<p>Global Settings:</p>
```
c:\apache-maven-3.5.2\conf\settings.xml
```
