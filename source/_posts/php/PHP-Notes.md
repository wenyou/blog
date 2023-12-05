---
title: PHP笔记
author: Zeeny
comments: false
date: 2009-07-31 12:48:15
updated: 2016-04-13 12:48:15
tags: [php,php函数,笔记,测试,PHP框架,防XSS,数组排序]
categories: php
summary: PHP笔记
---
# PHP笔记

### PHP框架

1. [Phalcon](https://phalconphp.com/zh/)
2. [C开发的PHP框架Phalcon性能有多高](http://www.dedecms.com/knowledge/program/php/2012/0827/13423.html)
3. Zephir： http://www.zephir-lang.com
4. Laravel：https://laravel.com/
5. NativePHP：https://nativephp.com/



### 压力测试

1. http_load
2. ab
3. siege

```shell
ab -n 1000 -c 5 http://172.37.37.12/login

http_load -p 100 -s 10 http://172.37.37.12/login

#运行命令行
http_load -parallel 10 -fetches 1000 urls.txt 
http_load -rate 5 -seconds 300 urls.txt 
# 可缩写为：
http_load -p 10 -f 1000 urls.txt 
http_load -r 5 -s 300 urls.txt
```

&emsp;&emsp;*参数介绍*
	&emsp;&emsp;&emsp;&emsp;-p 并发访问进程数
	&emsp;&emsp;&emsp;&emsp;-f 总的访问次数
	&emsp;&emsp;&emsp;&emsp;-r 每秒的访问频率，限制大于1小于1000
	&emsp;&emsp;&emsp;&emsp;-s 总的访问时间
	&emsp;&emsp;&emsp;&emsp;通常参数组合：-p –f；-r -s
	&emsp;&emsp;&emsp;&emsp;urls.txt 是你要访问的网址名，参数可以是单个的网址也可以是包含网址的文件。



```shell
siege -c 300 -r 100 -f url.txt
```

​	&emsp;&emsp;*说明：* -c是并发量，-r是重复次数。url.txt就是一个文本文件，每行都是一个url，它会从里面随机访问的。



### PHP知识

1. spl_autoload_register     — 注册给定的函数作为 __autoload 的实现。

2. php use，php使用命名空间：https://www.php.net/manual/zh/language.namespaces.php ，https://www.php.net/manual/zh/language.namespaces.importing.php

   > PHP 从 5.3 开始引入了命名空间的概念。



### PHP一些函数

#### chr, ord

```php
function addbom($str) {
	return chr(239).chr(187).chr(191).$str;
}
```

#### ini_set函数使用方法

​	&emsp;&emsp;ini_set是php自带的用来设置php.ini配置文件的函数。

​	&emsp;&emsp;该函数用时最好放到php的脚本最头部。

语法:

​	&emsp;&emsp;ini_set(“选项”,”值”)

​	&emsp;&emsp;比如:

```php
ini_set(”max_execution_time”, ”180”); //设置php的脚本超时时间为180秒 
ini_set(“asp_tags”,”On”); //打开asp脚本标记的支持 比如：<% echo “aaa”%>
ini_set(“display_errors”,”On”); //打印脚本错误信息
```

#### PHP Filter 函数 – 严格控制输入的数据格式 – 防XSS

- PHP 过滤器用于对来自非安全来源的数据（比如用户输入）进行验证和过滤。
- filter 函数是 PHP 核心的组成部分，无需安装即可使用这些函数。
- 把表单里的值封装成严格的数据格式匹配入库，从输入就杜绝XSS，举个例子：表单里只让填EMAIL格式的字符串，什么时候允许让进< >‘ “这样的字符了。

#### array_multisort - 对多个数组或多维数组进行排序 - 二维数组按元素值排序

​	&emsp;&emsp;array_multisort()php函数库自带的方法。用于对多个数组或多维数组进行排序。
​	&emsp;&emsp;php官方网址为：http://cn.php.net/manual/zh/function.array-multisort.php

​	&emsp;&emsp;我的应用为，有一个二维数组，我要按照其中二个元素的值的大小，把这个二维数组中的第一维从大到小排序。

```php
$all_list = array_merge((array)$this->dbo->query($sql_1),(array)$this->dbo->query($sql_2));

foreach ($all_list as $key => $row) {
	$all_list[$key]['packets'] = round($row['ps_orgin']+$row['ps_reply'],0);
	$all_list[$key]['bs_orgin'] = $row['bs_orgin'] + $row['bs_reply'];
	$packets[$key] = $all_list[$key]['packets'];
	$bs_orgin[$key] = $all_list[$key]['bs_orgin'];
	unset($all_list[$key]['bs_reply']);
	unset($all_list[$key]['ps_orgin']);
	unset($all_list[$key]['ps_reply']);
}

array_multisort($bs_orgin, SORT_DESC, $packets, SORT_DESC, $all_list);
```

​	&emsp;&emsp;更多用法见：http://cn.php.net/manual/zh/function.array-multisort.php



### linux下php在后台执行命令

​	&emsp;&emsp;启动命令后马上返回，得到此命令的pid:

```php
exec("/opt/monitor.sh > /dev/null 2>&1 & echo $!", $output, $retval);
```

​	&emsp;&emsp;要把输出重定向到其它设备，如果有错误输出，也要重定向到其它设备。

​	&emsp;&emsp;“echo $!” 不能少，否则exec会一直等待命令的执行后的返回值。



​	&emsp;&emsp;启动命令，命令发出exit指令后就返回:

```php
exec("/opt/monitor.sh > /dev/null 2>&1", $output, $retval);
```

​	&emsp;&emsp;适用于脚本中有调用其它子命令，且脚本执行完后子命令不结束。此时 $retval 即为 /opt/monitor.sh 的 exit 值。