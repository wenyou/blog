---
title: PHP扩展开发笔记(一)
author: Zeeny
comments: false
date: 2014-04-14
updated: 2014-04-14
tags: [php,php源码,php扩展]
categories: php
summary: PHP扩展开发
---

# PHP扩展开发笔记(一)

## 参考资源

* [用C/C++扩展你的PHP](http://www.laruence.com/2009/04/28/719.html)
* [关于做PHP扩展开发的一些资源](http://www.laruence.com/2011/09/13/2139.html)
* [扩展PHP[Extending PHP](一)](http://www.laruence.com/2008/08/16/301.html)
* [Yaf-一个PHP扩展实现的PHP框架](http://www.laruence.com/2010/09/04/1736.html)
* http://www.php.net/manual/book.yaf.php
* [Yaf - Yet Another Framework](http://www.yafdev.com)
* [深入浅出PHP(Exploring PHP)](http://www.laruence.com/2008/08/11/147.html)
* [深入理解PHP原理之函数(Introspecting PHP Function)](http://www.laruence.com/2008/08/12/164.html)
* [深入理解PHP原理之Opcodes](http://www.laruence.com/2008/06/18/221.html)
* [Yar – 并行的RPC框架(Concurrent RPC framework)](http://www.laruence.com/2012/09/15/2779.html)
* [关于Yaf的一些说明](http://www.laruence.com/2012/08/31/2742.html)
* [再一次, 不要使用(include/require)_once](http://www.laruence.com/2012/09/12/2765.html)
* [PHP的命名空间的实现](http://www.laruence.com/2010/10/12/1763.html)
* [Yaf的一些资源](http://www.laruence.com/2012/07/06/2649.html)
* [PHP扩展开发及内核应用](https://github.com/walu/phpbook/blob/master/preface.md)
* [TIPI 深入理解PHP内核](http://www.php-internals.com/)
* [深入理解Zend SAPIs(Zend SAPI Internals)](http://www.laruence.com/2008/08/12/180.html)
* [FreeBSD下安装mysql5.1.56+php5.2.17(FastCGI)+nginx1.0.1高性能Web服务器](http://www.92csz.com/12/542.html)
* http://my.oschina.net/20130614/blog/119198

## 安装php

* _安装php前的预装库：_
* 参考: lnmp1.1/centos.sh 安装所需要的依赖库。
* _安装php_

```shell
mkdir /usr/local/php
cd php-5.5.10

./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --enable-fpm --with-fpm-user=www --with-fpm-group=www --with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-iconv-dir --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --disable-rpath --enable-magic-quotes --enable-safe-mode --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curl --enable-mbregex --enable-mbstring --with-mcrypt --enable-ftp --with-gd --enable-gd-native-ttf --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --disable-fileinfo [ --enable-debug --enable-maintainer-zts --enable-embed ]

make ZEND_EXTRA_LIBS='-liconv'

make install
```

```shell
cp ./php.ini-development /usr/local/php/etc/php.ini
```

* 创建php-fpm.conf文件

  ```shell
  vi /usr/local/php/etc/php-fpm.conf
  ```

  &emsp;&emsp;php-fpm.conf文件内容：

```ini

	[global]
	pid = /usr/local/php/var/run/php-fpm.pid
	error_log = /usr/local/php/var/log/php-fpm.log
	log_level = notice
	
	[www]
	listen = /tmp/php-cgi.sock
	user = nobody
	group = nobody
	pm = dynamic
	pm.max_children = 10
	pm.start_servers = 2
	pm.min_spare_servers = 1
	pm.max_spare_servers = 6
	request_terminate_timeout = 100
```

* 启动php-fpm并加入启动脚本

  ```shell
  * /usr/local/php/sbin/php-fpm
  * echo "/usr/local/php/sbin/php-fpm" >> /etc/rc.d/start.sh
  ```

  

* vi nginx.conf

```ini

	server {
                listen       81;
                server_name www.zxoa.com;
                index index.html index.htm index.php;
                root  /home/test;

                location ~ .*\.(php|php5)?$
                        {
                                try_files $uri =404;
                                fastcgi_pass  unix:/tmp/php-cgi.sock;
                                fastcgi_index index.php;
                                include fcgi.conf; #改为fastcgi.conf,nginx1.4后
                        }

                location /status {
                        stub_status on;
                        access_log   off;
                }

                location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
                        {
                                expires      30d;
                        }

                location ~ .*\.(js|css)?$
                        {
                                expires      12h;
                        }
        }
```

* 启动nginx

  ```shell
  /opt/nginx/sbin/nginx
  echo "/opt/nginx/sbin/nginx" >> /etc/rc.d/start.sh
  ```

  

## 生成扩展骨架

* 进入php源码目录：

  ```sh
  cd /root/php-5.5.10/ext
  ```

  

* 执行：

  ```sh
  ./ext_skel --extname=stk
  ```

  ​	&emsp;&emsp;生成stk扩展骨架。

```sh

	To use your new extension, you will have to execute the following steps:

	$ cd ..
	$ vi ext/stk/config.m4
	$ ./buildconf
	$ ./configure --[with|enable]-stk
	$ make
	$ ./php -f ext/stk/stk.php
	$ vi ext/stk/stk.c
	$ make
	
	Repeat steps 3-6 until you are satisfied with ext/stk/config.m4 and
	step 6 confirms that your module is compiled into PHP. Then, start writing
	code and repeat the last two steps as often as necessary.

```

* _安装扩展_

  ```sh
  cd /root/php-5.5.10/ext/stk
  /usr/local/php/bin/phpize
  ./configure --with-php-config=/usr/local/php/bin/php-config
  make
  make install
  make clean all
  vi /usr/local/php/etc/php.ini
  ```

  

* php.ini添加：

  ```ini
  extension=/usr/local/php/lib/php/extensions/no-debug-non-zts-20121212/stk.so
  ```

  

* 重启php服务

  ```
  /usr/local/php/sbin/php-fpm [service php-fpm restart] 
  ```

  

* 测试

## 内存申请

```
	
	Traditional	  PHP/ZE内置的 Non-Persistent	  Prersistent
	
	malloc(count) 			 emalloc(count)        pemalloc(count, 1)*
	calloc(count, num)		 ecalloc(count, num)   pecalloc(count, num, 1)
	strdup(str)				 estrdup(str)          pestrdup(str, 1)
	strndup(str, len)		 estrndup(str, len)    pemalloc() & memcpy()
	free(ptr)				 efree(ptr)			    pefree(ptr, 1)
	realloc(ptr, newsize)   erealloc(ptr, newsize) perealloc(ptr, newsize, 1)
	malloc(count * num +extr) **safe_emalloc(count, num, extr) safe_pemalloc(count, num, extr)
```

## 常用的 PHP/ZE内置的方法

* RETURN_STRING(str, 1/0)
* RETURN_STRINGL(str, len, 1/0)
* RETURN_LONG(42)
* RETURN_DOUBLE(3.1415926)
* RETURN_BOOL(0/1)    可以使用 RETURN_TRUE 或 RETURN_FALSE代替
* RETURN_NULL()
* 使用php.ini文件中的值：INI_STR("stk.version") ，eg: RETURN_STRING(INI_STR("stk.version"),1)
* 使用 INI_STR() 来获取 stk.version 的当前值。除此之外，还有其他的一些函数来获取长整型、浮点型和布尔型值。在下表中列出了这些函数，以及它们相应的ORIG函数用来获取 INI 设置中原始值（在被 .htaccess 和 ini_set() 修改前的值）的函数：

```
	
	Current value     Original value           Type
	INI_STR(name)     INI_ORIG_STR(name)       char *(NULL terminated)
	INI_INT(name)     INI_ORIG_INT(name)       signed long
	INI_FLT(name)     INI_ORIG_FLT(name)       signed double
	INI_BOOL(name)    INI_ORIG_BOOL(name)      zend_bool 
```


* PHP_INI_ENTRY() 的第一个参数是包含 INI 条目名称的字符串。为了避免命名冲突，你应该遵守和函数名相同的惯例——使用扩展名作为命名的前缀，例如 stk.version。另外，用点号分隔扩展名和INI条目名也是个好习惯。
第二个参数是初始值，无论你的 INI 条目是数值还是字符串值，都需要传递一个相应的字符串值。这归因于 .ini 文件本身的文本性。当你在代码中使用 INI_INT(), INI_FLT(), INI_BOOL()的时候，它们会自动进行类型转换。
第三个参数指定访问模式修饰符。这是个用来确定该 INI 设置在何时何处能够被修改的掩码值。例如register_globals，在脚本中通过 ini_set() 修改它的值是毫无意义的，因为它在脚本执行之前的请求开始阶段起作用。另外，像allow_url_fopen，是管理类的设置，你不希望共享主机的用户能够通过 ini_set() 或者.htaccess 来改变它。这个参数的一个典型值是 PHP_INI_ALL，指示该值可以在任意地方被改变。PHP_INI_SYSTEM|PHP_INI_PERDIR指示该值只能在 php.ini 或者 .htaccess 中被改变，而不能通过ini_set() 改变。PHP_INI_SYSTEM 指示该值只能在 php.ini 文件中修改。


* int zend_parse_parameters(int num_args TSRMLS_DC, const char type_spec, ...)



```

	type_spec是格式化字符串，其常见的含义如下：
	参数   代表着的类型
	b   Boolean
	l   Integer 整型
	d   Floating point 浮点型
	s   String 字符串
	r   Resource 资源
	a   Array 数组
	o   Object instance 对象
	O   Object instance of a specified type 特定类型的对象
	z   Non-specific zval 任意类型～
	Z   zval**类型
	f   表示函数、方法名称，PHP5.1里貌似木有... ...
	
```


```

	参数  对应C里的数据类型
	b   zend_bool
	l   long
	d   double
	s   char*, int 前者接收指针，后者接收长度
	r   zval*
	a   zval*
	o   zval*
	O   zval*, zend_class_entry*
	z   zval*
	Z   zval**
```

* WRONG_PARAM_COUNT宏,它的功能是抛出一个E_WARNING级别的错误信息，并自动return。

* php_error_docref(NULL TSRMLS_CC, E_WARNING,"Expected at least 1 parameter.");

  __php_info_\*()系列的函数：__

* char \*php_info_html_esc(char *str TSRMLS_DC)

* void php_info_print_table_start(void)

* void php_info_print_table_end(void)

* void php_info_print_table_header(int cols, ...)

* void php_info_print_table_colspan_header(int cols, char \*header)

* void php_info_print_table_row(int cols, ...)

* void php_info_print_table_row_ex(int cols, char \*class, ...)

* void php_info_print_hr(void)

* _在内核中，会使用REGISTER _ \* _ CONSTANT()的 家族函数来使用常量_

* REGISTER_LONG_CONSTANT(char \*name, long lval, int flags)

* REGISTER_DOUBLE_CONSTANT(char \*name, double dval, int flags)

* REGISTER_STRING_CONSTANT(char \*name, char \*value, int flags)

* REGISTER_STRINGL_CONSTANT(char \*name,char \*value, int value_len, int flags)

## 工具

* gdb  Linux下C语言的调试器
* 安装 gdb， yum install gdb*

## 名词解释

* SAPI

> 我们平时接触的最多的是web模式下的php，当然你也肯定知道php还有个CLI模式。 其实无论哪种模式，PHP的工作原理都是一样的， 都是作为一种SAPI在运行（Server Application Programming Interface： the API used by PHP to interface with Web Servers）。当我们在终端敲入php这个命令时候，它使用的是"command line sapi"！它就像一个mini的web服务器一样来支持php完成这个请求，请求完成后再重新把控制权交给终端。
> 	
> 简单来说, SAPI就是PHP和外部环境的代理器。它把外部环境抽象后, 为内部的PHP提供一套固定的, 统一的接口, 使得PHP自身实现能够不受错综复杂的外部环境影响，保持一定的独立性
> 	
> 更多内容参看来自Laruence的博客对SAPI的介绍： 深入理解Zend SAPIs


* TSRM

> TSRM(Thread Safe Resource Management)
> TSRMLS(TSRM Local Storage)

