---
title: NodeJS安装
author: Zeeny
comments: false
date: 2015-08-19
updated: 2015-08-19
tags: [Javascript,nodejs,js,Node.js]
categories: [Javascript]
summary: Node.js安装
---


# NodeJS(Node.js)安装


1. 下载的文件不同，对应的安装方式不同。

2. 下载的是编译好的文件，node-v0.12.6-linux-x64.tar.gz：

```
tar -zxvf node-v0.12.6-linux-x64.tar.gz 
cd node-v0.12.6-linux-x64/bin
ls
./node -v
ln -s /usr/local/node-v0.12.6-linux-x64/bin/node /usr/local/bin/node
ln -s /usr/local/node-v0.12.6-linux-x64/bin/npm /usr/local/bin/npm
```

3. 下载的是源码，则需要通过源码编译：

```
tar xvf node-v0.10.28.tar.gz
cd node-v0.10.28 
./configure
make
make install
cp /usr/local/bin/node /usr/sbin
```

4. 查看当前安装的Node的版本：

```  		
node -v
```

5. 运行程序：

```
node ./webAPI.js &
```  		
   		

6. POI程序部署

```
/home/POI/
1. vi dbClient.js 

2. vi /etc/profile
export FOREVER_HOME=/home/POI/node_modules/forever
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$JBOSS_HOME/bin:$FOREVER_HOME/bin

3.cd /home/POI/server
forever start -l forever.log -o out.log webAPI.js

4. forever list  查看启动起来没有。

5. 通过url访问看看poi程序是否可用：http://172.37.37.27:8081/search/v1/queryByName/新华书店

6.以上是使用node_modules自带的forever模块，通常是使用安装的forever模块，安装forever需要联网，命令如下：

node环境装好后，安装forever模块， 命令： npm install forever -gd
```