---
layout: post
title: mysql开启InnoDB支持
description: mysql开启InnoDB支持
category: mysql
tags: [mysql,InnoDB]
---
#mysql开启InnoDB支持#

* 首先看看编译安装mysql时有没有编译InnoDb。

		mysql -uroot -pzxsoftdb123 登陆mysql shell
		show plugins;
		如果显示：InnoDB   | DISABLED   字样说明安装mysql时编译了InnoDb，只要启用InnoDb支持就行了。

* 如果安装mysql时编译了InnoDb，只要启用InnoDb支持就行了。

		/etc/init.d/mysql stop
		vi /etc/my.cnf
		注释掉#loose-skip-innodb
		保存，重启mysql。
		/etc/init.d/mysql start。
		就行了，再次show plugins;查看:
		InnoDB  | ACTIVE
	
* 如果show plugins列表中没有InnoDB 这一项，说明需要对mysql重新编译安装。

		./configure --prefix=/usr/local/mysql/ --with-plugins=innobase 
	
	
	​	
	
* [Linux启用MySQL的InnoDB引擎](http://blog.csdn.net/benben0503/article/details/8621015)



# MySQL针对单个表存储位置的定制方法

如要定制(自定义）单个数据库的存储位置，可以使用如下方法：

​	假如要定制的MySQL数据库名称为“stkitOaDb”，

​	那么可以在mysql安装的目录下(my.ini定义)的data文件夹中新建一个“stkitOaDb.sym”的文件，

​	stkitOaDb.sym里面的内容就是新的存储位置，如 “D:\newmysqldata\stkitdb”，

​	然后重启MySQL即可。

如果遇到问题，要检查下原来data目录下的“stkitOaDb”文件夹有没有删除。

