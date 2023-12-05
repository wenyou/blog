---
title: PHP扩展开发笔记(三)
author: Zeeny
comments: false
date: 2014-07-22
updated: 2014-07-22
tags: [php,php源码,php扩展]
categories: php
summary: PHP扩展开发
---

# PHP扩展开发笔记(三)

## 一、PHP内核变量访问宏

### 1. Z_TYPE(zval): 
​	&emsp;&emsp;对应zval结构体的实体
### 2. Z_TYPE_P(&zval) :

​	&emsp;&emsp;对应zval结构体的指针

###  Z_TYPE_PP(&&zval) : 

​	&emsp;&emsp;对应zval结构体的二级指针

### 4. Z_TYPE(zval) = IS_LONG; 

​	&emsp;&emsp;设置变量的类型

### 5. if(Z_TYPE(zval) == IS_LONG){...} 

​	&emsp;&emsp;访问变量的类型

### 6. 变量值对应的访问宏定义：

```

	类型     访问宏
	
	整数		Z_LVAL(zval)
			Z_LVAL_P(&zval)
			Z_LVAL_PP(&&zval)
			
	浮点		Z_DVAL(zval)
			Z_DVAL_P(&zval)
			Z_DVAL_PP(&&zval)
			
	布尔		Z_BVAL(zval)
			Z_BVAL_P(&zval)
			Z_BVAL_PP(&&zval)
	
	字符串	取得值
			Z_STRVAL(zval)
			Z_STRVAL_P(&zval)
			Z_STRVAL_PP(&&zval)
			取得长度
			Z_STRLEN(zval)
			Z_STRLEN_P(&zval)
			Z_STRLEN_PP(&&zval)
	
	数组		Z_ARRVAL(zval)
			Z_ARRVAL_P(&zval)
			Z_ARRVAL_PP(&&zval)
			
	资源		Z_RESVAL(zval)
			Z_RESVAL_P(&zval)
			Z_RESVAL_PP(&&zval)
	
```

### 7. zval_dtor 与zval_ptr_dtor的区别

​	&emsp;&emsp;两者的工作都与释放zval有关，但又有很大的区别,比如我们有一个zval *tmp，而且我们已经对它进行了MAKE_STD_ZVAL等一系列操作了；

​	&emsp;&emsp;如果我们对它使用zval_dtor， zval_dtor会直接把我们的tmp的value部分即tmp->value所指的内存释放掉，当然也可能不是直接释放掉，而是重新交给ZendMM，由ZendMM进行重新分配或者释放。

​	&emsp;&emsp;如果我们对它使用zval_ptr_dtor，zval_ptr_dtor首先会将它的refcount减一，如果减一后refcount为0了，便会再调用zval_dtor把tmp->value给释放掉，然后再调用efree_rel()函数把自己tmp所指的zval类型结构体所占的内存空间给释放掉。

​	&emsp;&emsp;如果减一后不为0呢？那zval_ptr_dtor便不会释放tmp->value和tmp本身，而是通知一下GC垃圾回收器，然后返回而已。

### 8. zend_read_property

​	&emsp;&emsp;zend_read_property函数用于读取对象的属性，而zend_read_static_property则用于读取静态属性。可以看出，静态属性是直接保存在类上的，用具体的对象无关。

### 9. zend_update_property

​	&emsp;&emsp;zend_update_property用来更新对象的属性，zend_update_static_property用来更新类的静态属性。如果对象或者类中没有相关的属性，函数将自动的添加上。

### 10. zend_hash_find()

​	&emsp;&emsp;zend_hash_find() 函数来找到当前某个作用域下用户已经定义好的变量。zend_hash_find()函数是内核提供的操作HashTable的API之一。http://www.walu.cc/phpbook/2.5.md

### 11. zend_hash_copy

​	&emsp;&emsp;zend_hash_copy, 使用HashTable与{数组},  http://www.walu.cc/phpbook/8.2.md

### 12. php_error_docref() 

​	&emsp;&emsp;php_error_docref()函数就相当与php语言里的trigger_error()函数。
它的第一个参数是一个将被添加到docref的可选的文档引用第三个参数可以是任何我们熟悉的 E_\*家族常量，用于指示错误的严重程度。后面的两个参数就像printf()风格的格式化和变量参数列表式样。

### 13. estrndup

​	&emsp;&emsp;void \*estrndup(void \*ptr，int len); 该函数能够分配len+1个字节的内存并且从ptr处复制len个字节到最新分配的块。

### 14. php_str_to_str

​	&emsp;&emsp;字符串替换

### 15. PHP扩展之smart_str分析

​	&emsp;&emsp;mart_str：smart_str_appends_ex，smart_str_appendc，smart_str_appendl...,smart_str_appends_ex(dest, src, what) 在一个smart_str变量后面添加一个字符串，它分为两种空间分配方法，当what为真是，它使用__zend_realloc函数，此函数为永久分配内存，在无法分配内存时会报Out of memory的，当what为假时，它使用标准空间分配，即ZendMM所我有的分配函数erealloc(ptr, size)。

​	&emsp;&emsp;smart_str_appends(dest, src) 在一个smart_str变量后面添加一个原生的字符串。

​	&emsp;&emsp;smart_str_appendc(dest, c) 在一个smart_str变量后面添加一个字符。smart_str_appendl(dest, src, len) 在一个smart_str变量后面添加一个长度为len的字符串。

​	&emsp;&emsp;smart_str_free(s) 消除smart_str变量，释放所占内存。

​	&emsp;&emsp;smart_str_append(dest, src) 在一个smart_str变量后面添加一个smart_str字符串。

​	&emsp;&emsp;[参考：PHP扩展之smart_str分析](http://www.phppan.com/2009/10/php-smart_str/)

​	

​	**PHP扩展之smart_str分析**
​	&emsp;&emsp;*php_smart_str_public.h*
​	&emsp;&emsp;在php_smart_str_public.h文件中我们找到了如下定义：


```c
	typedef struct {
              char *c;
              size_t len;
              size_t a;
	} smart_str;
```

​	&emsp;&emsp;*其中：*

​	&emsp;&emsp;&emsp;&emsp;size_t的定义为：typedef unsigned int size_t;
​	&emsp;&emsp;&emsp;&emsp;char *c 表示整个字符串
​	&emsp;&emsp;&emsp;&emsp;size_t len 表示整个字符串的实际长度
​	&emsp;&emsp;&emsp;&emsp;size_t a 表示这个字符的总长度（smart_str的字符串总长度总是以128字节的倍数增加）



**从php_smart_str.h文件所在的源码中我们可以看出它的空间分配具体有如下特点：**
	&emsp;&emsp;1、在初始化时，如果所给字符串的长度小于SMART_STR_START_SIZE（定义为78字节），则初始smart_str的总长度为78字节（即a值），并给它分配空间；如果所给字符串的长度大于SMART_STR_START_SIZE，则将此长度上加上SMART_STR_PREALLOC（定义为128字节）作为smart_str的总长度，并为之分配空间。
	&emsp;&emsp;2、在已有字符串后添加时，如果添加后的总长度小于当前smart_str->a的值，则无需分配空间，否则，在smart_str新的长度的基础上再加上128字节的空间。这样有些类似于操作系统中内存以页为单位分配，它的好处是对齐内存地址，提高访问速度。
	**在php_smart_str.h文件中，我们可以看到一些针对smart_str类型的宏定义的操作方法，方法列表如下：**

	smart_str_0 如果字符串存在，给字符串的最后添加’\0’;
	smart_str_alloc 在初始化和添加字符串时进行空间的分配；
	smart_str_appends_ex(dest, src, what) 在一个smart_str变量后面添加一个字符串，它分为两种空间分配方法，当what为真是，它使用__zend_realloc函数，此函数为永久分配内存，在无法分配内存时会报Out of memory的，当what为假时，它使用标准空间分配，即ZendMM所我有的分配函数erealloc(ptr, size)。
	smart_str_appends(dest, src) 在一个smart_str变量后面添加一个原生的字符串
	smart_str_appendc(dest, c) 在一个smart_str变量后面添加一个字符
	smart_str_free(s) 消除smart_str变量，释放所占内存
	smart_str_appendl(dest, src, len) 在一个smart_str变量后面添加一个长度为len的字符串
	smart_str_append(dest, src) 在一个smart_str变量后面添加一个smart_str字符串
	smart_str_append_long(dest, val) 将一个long的数字转化成字符串后添加到smart_str的后面	smart_str_append_unsigned(dest, val) 将一个unsigned long的数字转化成字符串后添加到smart_str的后面
	smart_str_appendc_ex(dest, ch, what) 与smart_str_appends_ex类似，所不同的是它添加的是一个字符。
	smart_str_setl(dest, src, nlen) 将一个长为len的原生的字符串强制转化为一个smart_str
	smart_str_sets(dest, src) 将一个原生的字符串强制转化为一个smart_str

__【经典代码】__
	**数字转字符串：**

```c
	char __b[32];             
	int __num = 28790;
	char *__p;
	__p = __b + sizeof(__b) - 1;
	*__p= '\0';                                                                                                                                                                                     
	do{                                                                                   
		 *--__p = (char) (__num % 10)+ '0';                                
		__num /=10;
	} while (__num > 0);
```



# 二、常用内置宏（方法）解释

- 1. __ZEND_STRL__

     

# 三、错误提示

 &emsp;&emsp;1. 警告：初始化时将整数赋给指针，未作类型转换。如果编译时提示这个错误，检查当前文件是否包含了(#include)调用其他文件函数的头文件（声明“调用函数”的头文件）。（http://blogs.360.cn/360apps/2013/03/19/%E5%B0%8F%E5%BF%83c%E8%AF%AD%E8%A8%80%E7%9A%84%E5%AE%9A%E4%B9%89%E4%B8%8E%E5%A3%B0%E6%98%8E/）