---
title: PHP问题与错误处理，出错解决方案
author: Zeeny
comments: false
date: 2012-05-30
updated: 2012-11-08
tags: [php,静态方法,json,错误处理,中文乱码,验证码]
categories: [php,错误处理,验证码]
summary: PHP问题与错误处理，出错解决方案。静态方法和实例方法的小漏洞-PHP Strict Standards解决方案
---

# 静态方法和实例方法的小漏洞-PHP Strict Standards解决方案

- php报错：Strict Standards: Non-static method request::set() should not be called statically。

​	&emsp;&emsp;原因：
​		&emsp;&emsp;&emsp;&emsp;**php5.3+** 开始执行严格错误检查，并且继承类必须在父类以后定义。

​	&emsp;&emsp;解决方法如下 :

	error_reporting(E_ALL & ~(E_STRICT | E_NOTICE));

​	&emsp;&emsp;静态方法和实例方法的小漏洞。

​	&emsp;&emsp;静态方法和实例方法都是保存在类结构体 zend_class_entry.function_table中，那这样的话， Zend引擎在调用的时候是怎么区分这两类方法的，比如我们静态调用实例方法或者实例调用静态方法会怎么样呢？

​	&emsp;&emsp;可能一般人不会这么做，不过笔者有一次错误的这样调用了，而代码没有出现任何问题，在review代码的时候意外发现笔者像实例方法那样调用的静态方法，而什么问题都没有发生(没有报错)。在理论上这种情况是不应发生的，类似这这样的情况在PHP中是非常的多的，例如前面提到的create_function方法返回的伪匿名方法，后面介绍访问控制时还会介绍访问控制的一些瑕疵，PHP在现实中通常采用Quick and Dirty的方式来实现功能和解决问题，这一点和Ruby完整的面向对象形成鲜明的对比。

​	&emsp;&emsp;我们先看一个例子：

	error_reporting(E_ALL);
	class A {
		public static function staticFunc() {
			echo "static";
		}
		public function instanceFunc() {
			echo "instance";
		}
	}
	A::instanceFunc(); // instance
	$a = new A();
	$a ->staticFunc(); // static

​	&emsp;&emsp;上面的代码静态的调用了实例方法，程序输出了instance，实例调用静态方法也会正确输出static，这说明这两种方法本质上并没有却别。唯一不同的是他们被调用的上下文环境，例如通过实例方法调用方法则上下文中将会有$this这个特殊变量，而在静态调用中将无法使用$this变量。

​	&emsp;&emsp;不过实际上Zend引擎是考虑过这个问题的，将error_reporting的级别增加E_STRICT，将会出出现E_STRICT错误:

	Strict Standards: Non-static method A::instanceFunc() should not be called statically

​	&emsp;&emsp;这只是不建议将实例方法静态调用，而对于实例调用静态方法没有出现E_STRICT错误，有人说：某些事情可以做并不代表我们要这样做。

​	&emsp;&emsp;PHP在实现新功能时通常采用渐进的方式，保证兼容性，在具体实现上通常采用打补丁的方式，这样就造成有些”边界“情况没有照顾到。



# echo json_encode数据json/javascript:unterminated string literal 解决


​	&emsp;&emsp;*unterminated string literal*，js输出数据库中的数据。

​	&emsp;&emsp;用Js输出从数据库中读取出的文本时，出错提示 *unterminated string literal*，主要是因为js里输出里包含了\n。解决办法是先将数据通过以下方式去掉\n，然后再传值给js。

​	&emsp;&emsp;例如：

	$audit_http_list = $this->dbo->query($sql);
	
	ob_start();
	include template(‘listhttp’,STK_M,”,false);
	$list = ob_get_contents();
	ob_end_clean();
	
	echo str_replace(array(‘\n’,'\t’,'\r’), ”,json_encode(array($pages,$list)));




# json_encode不能处理中文或中文乱码解决方案

​	&emsp;&emsp;php调用 *json_encode* 时，如果有中文，则为生成 *null*，中文无法转换，不能生成 *json* 格式的字符串。

​	&emsp;&emsp;如果确定所有的中文都用一个编码格式如：*gb2312*，则可以用以下解决方案：

	<?php
	    foreach ($data_list as $key=>$val) {
	        $data_list[$key]['name'] = urlencode(iconv(‘gb2312′,’utf-8′,		$val['name']));
	    }
	    
	    echo json_encode($data_list);
	?>

​	&emsp;&emsp;客户端 js 处理：

	var data=eval(json);
	var html = ‘<select>’;
	
	$.each(data, function(k) {
		html = ‘<option value=”‘+data[k]['id']+’”>’+ decodeURIComponent/decodeURI(json[k]['name'])+’</option>’;
	});
	
	html  =”</select>”;

​	&emsp;&emsp;就可以了。



​	&emsp;&emsp;但是如果，数据库/数组里的中文编码不统一，有多种编码 *utf-8*, *gbk*, *gb2312* 都有，或者原始数据本身就有乱码。上面的方法就行不通了。

​	&emsp;&emsp;可以用下面这种方案，例如我给公司做OA系统的方案：

	foreach($audit_http_list as $k=>$v){
		$audit_http_list[$k]['title'] = urlencode($v['title']);
		$audit_http_list[$k]['url'] = urlencode($v['url']);
	}

​	&emsp;&emsp;这里只需把含有中文的地方 *urlencode* 一下，不管它有没有乱码还是什么编码。



​	&emsp;&emsp;然后json_encode 一下：

	$data_json = json_encode(array($pages,$list))；

​	&emsp;&emsp;最后再 *urldecode* 一下：

	echo urldecode($data_json);

​	&emsp;&emsp;客户端，就不需要 *decodeURI* 了。

​	&emsp;&emsp;javascript:

	$.get(uri, function(json){
		var data = eval(json);
		$(‘#content_list’).html(data[1]);
		$(‘#pages’).html(data[0]);
	});



# 验证码显示不出来 – 多半由于图片前有输出造成的

​	&emsp;&emsp;验证码显示不出来，多半由于图片前有输出造成的。

​	&emsp;&emsp;有一种根绝的办法，就是在验证码图片输出前加 “**ob_clean();**”，在应用程序入口文件开头加上 “**ob_start();**”。

​	&emsp;&emsp;例如：

​	&emsp;&emsp;*checkcode.php* 显示验证码图片文件的代码如下：

	require dirname(__FILE__).’/includes/common.inc.php’;
	//这里省略N多生成图片的代码。。。。
	//这里省略N多生成图片的代码。。。。
	//output to browser
	ob_clean();
	header(“content-type:image/png\r\n”);
	imagepng($im);
	imagedestroy($im);

​	&emsp;&emsp;在 **common.inc.php** 文件的开头（或入口文件的开头）加上 “**ob_start();**”。
​	&emsp;&emsp;common.inc.php 代码如下：

	ob_start();
	
	define(‘IN_SCF’, TRUE);
	error_reporting(E_ALL);//E_ALL
	define(‘DS’,DIRECTORY_SEPARATOR);
	define(‘S_ROOT’, substr(__FILE__,0,-24).DS);
	$_CONFIG = $_SGLOBAL =array();
	$_CONFIG = parse_ini_file(S_ROOT.’config.ini’);
	date_default_timezone_set($_CONFIG['timezone_set']);
	$_start = explode(” “, microtime());
	$_SGLOBAL['timestamp'] =$timestamp = $_start[1];
	$_SGLOBAL['app_starttime'] = $_SGLOBAL['timestamp'] + $_start[0];
	set_include_path(S_ROOT.’includes’.DS);
	require_once ‘global.func.php’;
	ini_set(“magic_quotes_runtime”,0);
	define(‘DATA_PATH’,S_ROOT.’data’.DS);
	define(‘MAGIC_QUOTES_GPC’, get_magic_quotes_gpc());
	!defined(‘CURSCRIPT’) && define(‘CURSCRIPT’, ”);
	….
	….

​	&emsp;&emsp;这样处理后，验证码就会正常显示了。