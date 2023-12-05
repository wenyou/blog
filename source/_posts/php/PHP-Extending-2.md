---
title: PHP扩展开发笔记(二)
author: Zeeny
comments: false
date: 2014-06-10
updated: 2014-06-10
tags: [php,php源码,php扩展]
categories: php
summary: PHP扩展开发
---

# PHP扩展开发笔记(二)

## 参考经验

* 常用的通用功能已经封装好了，在如zen_API.h 头文件中，不用费力查看内部细节，浪费时间。（参考：Extending and Embedding PHP 的附录A）
* 在terminal中运行测试程序，可以看到扩展的内部错误输出，这一点对于解决内存泄漏问题尤其重要。（编译一个debug 的 lib）
* 开发过程中修改Makefile中的“CFLAGS = -g -O2”，去掉优化选项，增加-Wall和-pedantic，便于调试和显示编译警告；
* 某zval\*，但其strval非拷贝的，不可用zval_ptr_dtor(zval\*\*)，要用efree(void\*)。
* terminal中的$_SERVER['PWD']有值，但是无法通过zend_getenv()取得，原因应该是该值无意义或不可靠。
* 调用“导出函数”，可利用INTERNAL_FUNCTION_PARAM_PASSTHRU传参；声明的非导出函数可通过INTERNAL_FUNCTION_PARAM使用“导出函数”的参数。
* 注意：RETURN_TYPE用在选择分之和循环等处时，最好置于花括号中，
  或者不用分号，因为：

  ```c
  #define RETURN_BOOL(b) { RETVAL_BOOL(b); return; }
  ```
* 如果函数的参数是引用的，且非标量，要先析构，以防内存泄露。
* 抛出异常前最好判断EG(exception)中是否已经存在异常，否则会造成内存泄露。
* 当Web服务器API是ISAPI (IIS)的时候，zend_getenv函数是不起作用的。
* 向zend_stack_push()传入数据指针，实际存储(copy)的是该指针指向的数据，换句话说，传入的应该是要存储的数据的指针。
   
   ```c
   ZEND_API int zend_stack_push(zend_stack \*stack, void \*element, int size);
   ZEND_API int zend_stack_top(zend_stack \*stack, void \*\*element);
   ```
   
   其中，size == sizeof(\*element);
   类似地，zend_hash 也是如此，比较 zend_hash_update 和 zend_hash_find。
* 使用 add_assoc_zval(HashTable\*, const char\*, zval\*) 存储的是zval\*，而非zval，因此，存储用户传入的参数时候，要先拷贝一份新的zval，否则会发生不可预料的事情。
* zval_dtor(zval\*)释放变量及其内部的引用内存，zval_ptr_dtor(zval\*\*)先检查refcount,再决定是否调用zval_dtor(zval\*)，zval_copy_dtor(zval\*)仅执行深层的拷贝，即只拷贝其内部引用的内存，而不拷贝zval；
* 如使用VC编译win的动态链接库，而且代码中调用了zend函数，如zend_getenv，在zend.h中定义为：

  ```c
  extern "C" {
  extern ZEND_API char \*(\*zend_getenv)(char \*name, size_t name_len TSRMLS_DC);
  }
  ```

  &emsp;&emsp;需要引入该函数，如要使用ZEND_API，需要事先取消LIBZEND_EXPORTS（包括VC“设置”中的预处理定义），或者使用ZEND_DLIMPORT，

```c
ZEND_DLIMPORT char \*(\*zend_getenv)(char \*name, size_t name_len TSRMLS_DC);
```



__下面取自：__

​	&emsp;&emsp;**zend_config.w32.h**

```c

	#ifdef LIBZEND_EXPORTS
	#define ZEND_API __declspec(dllexport)
	#else
	#define ZEND_API __declspec(dllimport)
	#endif

	#define ZEND_DLEXPORT        __declspec(dllexport)
	#define ZEND_DLIMPORT        __declspec(dllimport)
```
​	&emsp;&emsp;*executor_globals_id* 也需要作如下声明：

```c
ZEND_DLIMPORT int executor_globals_id；
```

​	&emsp;&emsp;这个比较有用，如果你要手工编译某些扩展的时候，比如我在编译 *sqlite3* 这个扩展的时候，就遇到这个问题。

