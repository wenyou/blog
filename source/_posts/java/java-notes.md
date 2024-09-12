---
title: Java笔记
author: Zeeny
comments: false
date: 2014-08-06
updated: 2014-08-06
tags: [Java]
categories: Java
summary: Java笔记
---


# Java笔记 

### 1. java war 打包、解压命令 : 

```
//将当前目录打包成war包
jar  -cvf  temp.war  *

命令格式：
	java cvf 打包文件名称 打包当前目录中的所有文件

解压自然就是:
jar xvf temp.war
```

### 2. Java中访问文件内容的4种方法：

* RandomAccessFile，随机读取数据，此种访问速度较慢。
* FileInputStream，文件输入流，使用此种方式访问速度较慢。
* 缓冲读取（例 BufferedReader）,使用此种方式访问速度较快。
* 内存映射（MappedByteBuffer）,使用此种方式访问速度最快。


## Java习惯用法总结

#### 1. 实现：equals()
```
class Person {
	String name;
	int birthYear;
	byte[] raw;
	
	public boolean equals(Object obj) {
			if (!obj instanceof Person)
					return false;

			Person other = (Person)obj;
			return name.equals(other.name) && birthYear == other.birthYear && Arrays.equals(raw, other.raw);
		}
			
		public int hashCode() { ... }
}
```
*   参数必须是Object类型，不能是外围类。
*   foo.equals(null) 必须返回false，不能抛NullPointerException。（注意，null instanceof 任意类 总是返回false，因此上面的代码可以运行。）
*   基本类型域（比如，int）的比较使用 == ，基本类型数组域的比较使用Arrays.equals()。
*   覆盖equals()时，记得要相应地覆盖 hashCode()，与 equals() 保持一致。
*   参考： java.lang.Object.equals(Object)。


#### 2. 实现hashCode()
```
class Person {
	String a;
	Object b;
	byte c;
	int[] d;
	
	public int hashCode() {
			return a.hashCode() + b.hashCode() + c + Arrays.hashCode(d);
		}
		
		public boolean equals(Object o) { ... }
}
```
*   当x和y两个对象具有x.equals(y) == true ，你必须要确保x.hashCode() == y.hashCode()。
*   根据逆反命题，如果x.hashCode() != y.hashCode()，那么x.equals(y) == false 必定成立。
*   你不需要保证，当x.equals(y) == false时，x.hashCode() != y.hashCode()。但是，如果你可以尽可能地使它成立的话，这会提高哈希表的性能。
*   hashCode()最简单的合法实现就是简单地return 0；虽然这个实现是正确的，但是这会导致HashMap这些数据结构运行得很慢。
*   参考：java.lang.Object.hashCode()。
