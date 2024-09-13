---
title: LNAMP杂记
author: Zeeny
comments: false
date: 2014-09-29
updated: 2014-09-29
tags: [lnmp,lamp,php,nignx,apache,linux,centos]
categories: [linux,php]
summary: LNAMP杂记
---



# LNAMP杂记


__1. Linux下编译php 不编译mysql 让php支持mysql扩展__
 
```
--with-mysql=mysqlnd
--with-mysqli=mysqlnd
--with-pdo-mysql=mysqlnd

	mysqlnd是php5.3以后版本加入的，5.3以下版本必须还是要编译mysql。
```
 
* 之前编译php的时候，需要加个参数--with-mysql=/usr/local/mysql （mysql 安装路径），因为需要mysql头文件和库文件（mysql.h 等），只有安装mysql 客户端（mysql-devel）。

* 但是php5.3以上就不要安装mysql客户端了，“对于PHP 5.3.0或更新版本，mysqli默认使用Mysql Native Driver作为驱动。 这个驱动比libmysql会有一些优势，--with-mysql=mysqlnd。”

