---
title: PHP的迭代器笔记
author: Zeeny
comments: false
date: 2014-04-10
updated: 2014-04-10
tags: [php,迭代器,Iterator,PHP数据结构]
categories: php
summary: PHP的迭代器
---

# PHP的迭代器


### 实例
&emsp;&emsp;1.用PHP的迭代器来实现一个斐波纳契数列。

```php
class Fibonacci implements Iterator {
	private $pervious = 1;
	private $current = 0;
	private $key = 0;
	
	public function current() {
		return $this->current;
	}
	
	public function key() {
		return $this->key;
	}
	
	public function next() {
		//将当前值保存到 $newprevious
		$newprevious = $this->current;
		//将上一个值与当前值的和赋给当前值
		$this->current += $this->previous;
		//前一个当前值赋给上一个值
		$this->previous = $newprevious;
		$this->key++;
	}
	
	public function rewind() {
		$this->previous = 1;
		$this->current = 0;
		$this->key = 0;
	}
	
	public function valid() {
		return true;
	}
}
//use
$seq = new Fibonacci;
$i = 0;
foreach($seq as $f) {
	echo "$f ";
	if($i++ === 15) break;
}

//运行结果：
0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610
```


​	



