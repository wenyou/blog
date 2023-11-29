---
title: PHP问题与错误处理，出错解决方案 --（系列2，续篇）
author: Zeeny
comments: false
date: 2014-03-14
updated: 2015-07-21
tags: [php,错误处理,PGP加密,php函数]
categories: [php,错误处理,加密]
summary: PHP问题与错误处理，出错解决方案。PHP调用gpg加密命令不执行；解决PHP里fopen、file_get_content、get_headers不能解析域名问题
---

# PHP调用gpg加密命令不执行
​	&emsp;&emsp;PHP调用gpg加密命令不执行，在命令行下可以执行。exec返回值是2。

	php:
		exec('gpg --recipient zhxxxx@cnzxsoft.com --output /tmp/test.en.txt --encrypt /tmp/test.txt', $output, $resultval);


​	&emsp;&emsp;*$resultval* 返回值为：2，*$output* 为空，不能加密文件。

	#shell:
		gpg --recipient zhxxxx@cnzxsoft.com --output /tmp/test.en.txt --encrypt /tmp/test.txt

​	&emsp;&emsp;*shell* 命令行下可以执行。



​	&emsp;&emsp;**可能原因：**

​		&emsp;&emsp;&emsp;&emsp;1、php调用时找不到公钥存储的位置。

​		&emsp;&emsp;&emsp;&emsp;2、php以是非root权限运行的，不能成功执行gpg命令。



​	&emsp;&emsp;我用的**解决办法**（不一定是对的或最好的解决方式，但是暂时解决了问题）：

#### 方法如下：
​	&emsp;&emsp;**1、**新建C程序文件 gpgen.c ，代码如下：

	#include <stdio.h>
	#include <stdlib.h>
	#include <sys/types.h>
	#include <unistd.h>
	
	int main(int argc, char* argv[])	{
	    uid_t uid, euid;
	    uid = getuid();
	    euid = geteuid();
	    if(setreuid(euid, uid))
	            perror("setreuid");
	    
	    char s[256] = {0};
	    sprintf(s,"gpg --recipient %s --output %s --encrypt %s", argv[1], argv[2], argv[3]); 
	    
	    system(s);
	    
	    //system("whoami > /tmp/who.txt"); //查看一下用户是不是root
	    
	    return 0;
	}

​	&emsp;&emsp;**2、**把上面的文件生成可执行程序：

	#shell:
	gcc -o gpgen -Wall gpgen.c
	chmod u+s gpgen

​	&emsp;&emsp;**3、**再用PHP调用这个生成的可执行程序，代替直接调用gpg命令：

	php:
		putenv("GNUPGHOME=/root/.gnupg"); //这个也要加上，指定GNUPGHOME，要不gpg还是查不到公钥所在的位置。
		
	    exec("./gpgen zhxxxxx@cnzxsoft.com /tmp/test.en.txt /tmp/test.txt", $output, $resultval);

​	&emsp;&emsp;这样就可以解决问题了。



# 解决php里fopen、file_get_content、get_headers不能解析域名问题

## 		原因一：-with-curlwrapper PHP编译参数

&emsp;&emsp;1. [解决PHP里FOPEN不能解析域名](http://solf.me/php-fopen-domain-resolve-issue/)
&emsp;&emsp;2. [file_get_contents returns empty string](http://stackoverflow.com/questions/4172860/file-get-contents-returns-empty-string)
&emsp;&emsp;3. [php无法解析域名问题](http://bbs.vpser.net/thread-10771-1-1.html)
&emsp;&emsp;4. [php file_get_contents 不能获得内容](http://segmentfault.com/q/1010000000339984)

```
I fixed this issue on my server (running PHP 5.3.3 on Fedora 14) by removing the --with-curlwrapper from the PHP configuration and rebuilding it.
```
​	&emsp;&emsp;**解决办法：**
​		&emsp;&emsp;&emsp;&emsp;重新编译PHP，去掉 *-with-curlwrapper* 参数，编译前记得先执行 *make clean*。




## 		原因二：/etc/resolv.conf  PHP用户没有这个文件的访问权限

​	&emsp;&emsp;给 */etc/resolv.conf* 设置访问权限：
```
chmod 755 /etc/resolv.conf
```

​	&emsp;&emsp;执行后重启 nginx、php。
