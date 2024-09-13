---
title: Ubuntu笔记
author: Zeeny
comments: false
date: 2015-09-01
updated: 2015-09-01
tags: [linux,Ubuntu]
categories: [linux,Ubuntu]
summary: Ubuntu笔记
---



# Ubuntu笔记

## 一、常用命令

* 开启root帐号的方法，命令：sudo passwd root，给root设置密码。

* 切换到root用户的命令：su

* 当你使用完毕后屏蔽root帐号使用：sudo passwd -l root，这个将锁住root帐号。

* 在终端模式下切换到root身份：sudo -s -H

* root用户关机、重启命令：halt、poweroff 、reboot
```
关机命令： 
1、halt   立刻关机 
2、poweroff 立刻关机 
3、shutdown -h now 立刻关机(root用户使用) 
4、shutdown -h 10 10分钟后自动关机 

重启命令： 
1、reboot 
2、shutdown -r now 立刻重启(root用户使用) 
3、shutdown -r 10 过10分钟自动重启(root用户使用) 
4、shutdown -r 20:35 在时间为20:35时候重启(root用户使用)
```


## 二、配置网络

### 1. 修改IP地址

```
sudo vi /etc/network/interfaces

  auto eth0                   #设置自动启动eth0接口
	iface eth0 inet static      #配置静态IP
	address 172.16.37.13        #IP地址
	netmask 255.255.0.0         #子网掩码
	gateway 172.16.18.1         #默认网关
```


### 2. 修改DNS

```
sudo vi /etc/resolv.conf      重启后丢失

nameserver 127.0.0.1 
nameserver 223.5.5.5

sudo vi /etc/resolvconf/resolv.conf.d/head ＃重启后不丢失
nameserver 127.0.0.1 
nameserver 223.5.5.5
```

### 3. 重启网络，使配置生效

```
sudo /etc/init.d/networking restart


ifconfig eth0 down
ifconfig eth0 up
```

## 三、安装SSH

```
sudo apt-get install openssh-server
```

## 四、安装MySQL

```
首先查看有没有已安装：sudo netstat -tap | grep mysql
    
更新一下源:
sudo apt-get update    
  (有些软件使用apt-get install安装不上，提示Err http://security.ubuntu.com/ubuntu/ trusty-security/main mysql-server-core-5.5 amd64 5.5.47-0ubuntu0.14.04.1 404  Not Found [IP: 91.189.88.152 80] E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing? 这类的找不到错误，使用sudo apt-get update更新一下系统即可。)

方法一：
sudo apt-get install mysql-server
sudo apt-get install mysql-client
sudo apt-get install libmysqlclient-dev
	
	
启动MySQL服务： sudo start mysql
停止MySQL服务： sudo stop mysql
修改 MySQL 的管理员密码： sudo mysqladmin -u root password newpassword
```

## 五、安装Apache

```
安装gcc
  sudo apt-get install build-essential
    
编译安装APR
  wget http://mirrors.hust.edu.cn/apache//apr/apr-1.5.2.tar.gz
  sudo mv apr-1.5.2.tar.gz /usr/local/src/
  sudo tar -zxvf apr-1.5.2.tar.gz
  cd apr-1.5.2/
  sudo ./configure -prefix=/usr/local/apr
  sudo make
  sudo make install
    
再编译安装apr-util
  sudo wget http://www.apache.org/dist/apr/apr-util-1.5.4.tar.bz2
  sudo tar -jxvf apr-util-1.5.4.tar.bz2 
  cd apr-util-1.5.4/
  sudo ./configure -prefix=/usr/local/apr-util --with-apr=/usr/local/apr
  sudo make
  sudo make install
    
再编译安装pcre
  sudo wget http://exim.mirror.fr/pcre/pcre-8.35.tar.gz
  sudo tar -zxvf pcre-8.35.tar.gz
  cd pcre-8.35/
  sudo ./configure -prefix=/usr/local/pcre
  sudo make
  sudo make install
    
安装zlib-devel
  sudo apt-get install zlib1g-dev
  
  wget http://mirrors.hust.edu.cn/apache//httpd/httpd-2.4.20.tar.bz2
  tar xvf httpd-2.4.20.tar.bz2
  cd httpd-2.4.20/
  sudo ./configure -prefix=/usr/local/apache2 --enable-deflate --enable-expires --enable-headers --enable-modules=most --enable-so --with-mpm=worker --enable-rewrite --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util --with-pcre=/usr/local/pcre
  echo $? [检查是否有错误,返回0说明没问题了]
  sudo make
  sudo make install


启动apache
  sudo vi /usr/local/apache2/conf/httpd.conf
  #ServerName www.example.com:80 => ServerName localhost:80
  sudo /usr/local/apache2/bin/apachectl start
    
    
安装php前
  sudo apt-get install openssl
  sudo apt-get install libssl-dev
  sudo apt-get install curl
  sudo apt-get install libcurl4-gnutls-dev
  sudo apt-get install libjpeg-dev
  sudo apt-get install libpng-dev
  sudo apt-get install libmcrypt4
  sudo apt-get install libmcrypt-dev
  sudo apt-get install libreadline6 libreadline6-dev
  sudo apt-get install libxml2
  sudo apt-get install libxml2-dev
  sudo apt-get -y install libfreetype6-dev
    
  scp /Users/youwenzhang/USVN/MyRepositories/IKPCP-M/部署\&升级/ccsinstall/src/libmcrypt-2.5.8.tar.gz youwenzhang@172.16.37.14:/home/youwenzhang
  sudo wget http://vorboss.dl.sourceforge.net/project/mhash/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz
  sudo wget http://heanet.dl.sourceforge.net/project/mcrypt/MCrypt/2.6.8/mcrypt-2.6.8.tar.gz
  sudo tar -zxvf libmcrypt-2.5.8.tar.gz
  cd libmcrypt-2.5.8
  sudo ./configure
  sudo make
  sudo make install
    
  sudo tar -zxvf mhash-0.9.9.9.tar.gz
  cd mhash-0.9.9.9/
  sudo ./configure
  sudo make
  sudo make install
    
  sudo tar -zxvf mcrypt-2.6.8.tar.gz
  cd mcrypt-2.6.8/
  sudo LD_LIBRARY_PATH=/usr/local/lib ./configure
  sudo make
  sudo make install


安装php
  tar -jxvf php-7.0.6.tar.bz2
  cd php-7.0.6/
  sudo ./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2/bin/apxs --with-config-file-path=/usr/local/php/etc --enable-fpm --with-fpm-user=www --with-fpm-group=www --enable-shared --enable-opcache  --with-mysql --with-mysqli --with-mysql-sock  --enable-pdo --with-pdo-mysql --with-iconv-dir --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --disable-rpath --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curl --enable-mbregex --enable-mbstring --with-mcrypt --enable-ftp --with-gd --enable-gd-native-ttf --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --disable-fileinfo --enable-maintainer-zts
    
  sudo make
  sudo make install
    
  sudo cp php.ini-development /usr/local/php/etc/php.ini


编辑 httpd.conf 文件以调用 PHP 模块
  sudo vi /usr/local/apache2/conf/httpd.conf
    LoadModule php7_module        modules/libphp7.so
  
    告知 Apache 将特定的扩展名解析成 PHP
    <FilesMatch \.php$>
      SetHandler application/x-httpd-php
    </FilesMatch>    

  sudo /usr/local/apache2/bin/apachectl stop
  sudo /usr/local/apache2/bin/apachectl start



开机启动：
	sudo cp /usr/local/apache2/bin/apachectl /etc/init.d/httpd
	sudo ln -s /etc/init.d/httpd /etc/rc2.d/S80httpd 
	
	sudo service httpd start
	sudo service httpd stop
	
删除：
	sudo update-rc.d -f httpd remove  
```