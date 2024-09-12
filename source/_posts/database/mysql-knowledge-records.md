---
title: Mysql使用知识点记录
author: Zeeny
comments: false
date: 2015-03-05
updated: 2015-03-05
tags: [Mysql]
categories: 数据库
summary: Mysql使用知识点记录
---


# Mysql使用知识点记录

## 一、命令行相关操作


### 1.1. 查看 mysql 数据库编码的命令

```
查看某个数据库的编码:
	mysql> show create database dbname;

	或
	
	mysql> use dbname;
	mysql> status;
```

## 二、数据库优化

* 把长连接改为短连接,长连接只在访问远程数据库（比如你的机房里有独立的数据库服务器）才能发挥优势。

* 可是mysql查询的瓶颈还是没有解决，通过查找相关资料，我使用mysql的慢查询功能，发现原来这是由于通用的论坛程序没有对大负载量情况考虑周到造成的。我将原来直接使用limit start,num的分页代码更改成使用子查询的select * from table where col>(select col from table limit start,1) limit num，将原来的单列索引更换成符合Sql查询语句的复合索引，将频繁调用的sql用php写操作缓存，将like搜索转换成中文全文搜索。经过这样的优化，mysql勉强可以支撑1000万PV的论坛了，但还是有点慢，于是我安装了memcache，并将论坛的sql类改写为支持memcache的语句。虽然使用memcache会有一些延时，但相对于等待漫长的sql查询，这样的延时对浏览网站的会员是更容易忍受的选择。


## 三、mysql开启InnoDB支持

* 首先看看编译安装mysql时有没有编译InnoDb。

```
登陆mysql shell:
	mysql -uroot -p123456 
	show plugins;

	如果显示：InnoDB   | DISABLED   字样说明安装mysql时编译了InnoDb，只要启用InnoDb支持就行了。
```


* 如果安装mysql时编译了InnoDb，只要启用InnoDb支持就行了。

```
/etc/init.d/mysql stop
vi /etc/my.cnf

	注释掉#loose-skip-innodb
	保存，重启mysql。

/etc/init.d/mysql start。
	就行了，再次 show plugins; 查看:
		InnoDB  | ACTIVE
```


* 如果 show plugins 列表中没有 InnoDB 这一项，说明需要对mysql重新编译安装。

```
./configure --prefix=/usr/local/mysql/ --with-plugins=innobase
```


## 四、MySQL针对单个表存储位置的定制方法

* 如要定制(自定义）单个数据库的存储位置，可以使用如下方法：

	假如要定制的MySQL数据库名称为“stkitOaDb”，那么可以在mysql安装的目录下(my.ini定义)的data文件夹中新建一个“stkitOaDb.sym” 的文件，stkitOaDb.sym 里面的内容就是新的存储位置，如 “D:\newmysqldata\stkitdb”，然后重启MySQL即可。

	如果遇到问题，要检查下原来data目录下的 “stkitOaDb” 文件夹有没有删除。

