---
title: PHP SPL笔记
author: Zeeny
comments: false
date: 2014-04-11
updated: 2014-04-11
tags: [php,SPL]
categories: php
summary: PHP SPL笔记
---

# PHP SPL

* SPL是Standard PHP Library（PHP标准库）的缩写。
* [PHP SPL笔记](http://www.ruanyifeng.com/blog/2008/07/php_spl_notes.html)

## 实例
&emsp;&emsp;1. 使用 SPL 的迭代器寻找指定目录中的图片文件的例子。

```php
<?php
	class RecursiveFileFilterIterator extends FilterIterator {
    	// 满足条件的扩展名
    	protected $ext = array('jpg','gif');
 
	    /**
	     * 提供 $path 并生成对应的目录迭代器
	     */
	    public function __construct($path) {
	        parent::__construct(new RecursiveIteratorIterator(new RecursiveDirectoryIterator($path)));
	    }
 
	    /**
	     * 检查文件扩展名是否满足条件
	     */
	    public function accept() {
	        $item = $this->getInnerIterator();
	        if ($item->isFile() &&
	                in_array(pathinfo($item->getFilename(), PATHINFO_EXTENSION), $this->ext)) {
	            return TRUE;
	        }
	    }
	}
 
	// 实例化
	foreach (new RecursiveFileFilterIterator('/home/images') as $item) {
    	echo $item . PHP_EOL;
	}
	?>
```


​	

