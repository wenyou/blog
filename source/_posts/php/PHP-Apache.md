---
title: PHP Apache配置 泛域名应用 URL重写
author: Zeeny
comments: false
date: 2009-08-03
updated: 2009-08-18
tags: [url重写,apache,php,泛域名,web]
categories: [php,apache,web]
summary: PHP Apache配置 泛域名应用 Apache php URL重写
---
# PHP Apache  URL 重写

&emsp;&emsp;1、LoadModule rewrite_module modules/mod_rewrite.so 启动（将前面的#去了）
&emsp;&emsp;2、修改：

	<Directory />
		Options FollowSymLinks
		AllowOverride All
		Order deny,allow
		Deny from all
	</Directory>

​	&emsp;&emsp;AllowOverride none 改为 All



# PHP Apache泛域名应用

​	&emsp;&emsp; **以Windows开发环境**
&emsp;&emsp;1、windows 下的 hosts文件

	127.0.0.1    asia.t
	127.0.0.1    *.asia.t
	127.0.0.1    www.asia.t
	127.0.0.1    coca.asia.t
&emsp;&emsp;2、apache 下的 httpd-vhosts.conf文件

	<VirtualHost *:80>
		DocumentRoot E:\www\asia\www
		ServerName *.asia.t
		ServerAlias *.asia.t
	</VirtualHost>
&emsp;&emsp;3、php处理

	/*
	 代理商子网站处理
	 DOMAIN代理商二级域名前缀 GE: coca
	 BASEDOMAIN 网站真实域名，不带www.  EG: asia.com
	 SITEDOMAIN 访问时网站地址，可能是二级域名，可以为： *.asia.com或www.asia.com或asia.com
	 代理商子网站二级域名组合形式如下： coca.asia.com
	*/
	$site_url = $_SERVER['HTTP_HOST'];
	$site_url = explode('.',$site_url);
	if(count($site_url)<3)
	{
		define('DOMAIN','www');
		define('SITEDOMAIN',$_SERVER['HTTP_HOST']);
		define('BASEDOMAIN',SITEDOMAIN);
	}
	else
	{
		define('DOMAIN',$site_url[0]);
		if(DOMAIN == 'www')
		{
			define('SITEDOMAIN',$_SERVER['HTTP_HOST']);
			define('BASEDOMAIN',str_replace('www.','',SITEDOMAIN));
		}
		else
		{
			define('BASEDOMAIN',str_replace(array(DOMAIN.'.',DOMAIN),array('',''),$_SERVER['HTTP_HOST']));
			define('SITEDOMAIN',DOMAIN.'.'.BASEDOMAIN);
		}
	}