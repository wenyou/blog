---
title: Mac Java JDK,MACBOOKPro Eclipse Maven Web配置
author: Zeeny
comments: false
date: 2014-12-03
updated: 2014-12-03
tags: [Mac,Java,JDK]
categories: [Mac,Java]
summary: Mac Java JDK使用,MACBOOKPro Eclipse Maven Web配置
---


# Mac Java JDK使用

## 一、Mac JDK1.7安装

* 下载：jdk-7u71-macosx-x64.dmg

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

# MACBOOKPro Eclipse Maven Web配置

## 1、maven安装

1. 下载maven包apache-maven-3.5.2-bin.zip http://maven.apache.org/download.cgi
    解压到系统目录，例如：/Users/youwenzhang/Applications/apache-maven-3.5.2


2. 配置系统环境变量

    终端执行：vi .bash_profile 
    结尾处键入：
      export MAVEN_HOME=/Users/youwenzhang/Applications/apache-maven-3.5.2
      export PATH=${PATH}:${MAVEN_HOME}/bin

    :wq保存并退出，并使环境变量立即生效
      source ~/.bash_profile

     查看maven是否安装成功
      mvn -v

## 2. Eclipse的maven插件安装

1. 打开Eclipse，选择Help->Install New SoftWare


2. 点击Add...
   http://m2eclipse.sonatype.org/sites/m2e    


3. 点击OK，等待安装完成


4. 安装验证：
    重启eclipse， Help --> About Eclipse --> Installation Details
    在Installed Software标签中检查刚才选择的模块是否在这个列表中

    检查eclipse是否已经支持创建Maven项目：
    File --> New --> Other ，找到Maven一项，如果展开一切正常，说明m2eclipse已经正确安装了。


## 3. eclipse配置maven

1. eclipse -> prefrences -> maven -> user setting

`/Users/youwenzhang/Applications/apache-maven-3.5.2/conf/settings.xml`

2. eclipse -> prefrences -> maven -> installations -> Add...

    `apache-maven-3.5.2`

## 4. 配置tomcat的manager, 配置tomcat和maven

1. 进入/usr/local/apache-tomcat-7/conf/tomcat_users.xml，修改如下：

```
<role rolename="admin-gui"/>
<role rolename="admin-script"/>
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username="admin" password="admin" roles="manager-gui,manager-script,manager-jmx,manager-status,admin-script,admin-gui"/>
```

2. 进入/Users/youwenzhang/Applications/apache-maven-3.5.2/conf/settings.xml，修改如下：

* 在 servers节点中增加:
```
<server>  
  <id>tomcat7</id>  
  <username>admin</username>  
  <password>admin</password>  
</server> 

如果是tomcat8:
<server>  
  <id>tomcat8</id>  
  <username>admin</username>  
  <password>admin</password>  
</server> 
```

## 5. 部署maven web到tomcat，使用tomcat7-maven-plugin启动web项目

1. 在tomcat服务器的conf/Catalina/localhost/目录下创建一个manager.xml文件 

* 新建manager.xml：
![alt text](/images/mac-maven-image.png)

2. 配置项目中的 pom.xml:
![alt text](/images/mac-pom-image.png)

* 如下配置开发机（mac eclipse maven tomcat8)生效，上面不生效。
![alt text](/images/mac-pom-image-2.png)

* 正式上线，发布(部署)项目（AKBD.war）的时候，只需要保留如下这些配置就行了：
![alt text](/images/mac-pom-image-3.png)

* 同时在tomcat的 conf目录下修改配置文件server.xml，加上如下配置：
![alt text](/images/mac-server-xml-image.png)

<p>就可以通过http://ip:8081/打开系统了。</p>


3. 右击项目：
![alt text](/images/mac-eclipse-image.png)

* 第一次部署deploy：
![alt text](/images/mac-eclipse-deploy-image.png)

* 重复部署redeploy：
![alt text](/images/mac-eclipse-redeploy-image.png)

<p>
然后进行部署，如果是第一次部署，运行mvn tomcat7:deploy进行自动部署(对于tomcat8,9，也是使用tomcat7命令)，如果是更新了代码后重新部署更新，运行mvn tomcat7:redeploy，如果第一次部署使用mvn tomcat7:redeploy，则只会执行上传war文件，服务器不会自动解压部署。
</p>
<p>
如果路径在tomcat服务器中已存在并且使用mvn tomcat7:deploy命令的话，上面的配置中一定要配置<update>true</update>，不然会报错。
</p>

<p>如果IDE是eclipse,就在runas->run configurations中配置一个maven build，intellij类似。</p>


* 先把Tomcat启动起来之后，再执行上图中的 run，进行项目的部署。

4. 其他： 使用命令部署：
```
curl -i -X PUT -u admin:admin http://localhost:8080/manager/text/deploy?path=/AKBD --upload-file /Users/youwenzhang/USVN/ZXOA-BDSA/Allknow/web/AKBD/target/AKBD.war
```

5. 出错处理，如果出现如下错误：
```
[ERROR] Failed to execute goal org.apache.tomcat.maven:tomcat7-maven-plugin:2.2:redeploy (default-cli) on project AKBD: Cannot invoke Tomcat manager: Broken pipe (Write failed) -> [Help 1]
```
![alt text](/images/mac-war-error-image.png)

* 需要在项目的POM.xml文件中加入如下配置：
![alt text](/images/mac-war-pom-image.png)

