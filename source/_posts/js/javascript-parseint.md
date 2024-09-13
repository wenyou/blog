---
title: js中parseInt用法和parseInt("08")问题
author: Zeeny
comments: false
date: 2012-08-03
updated: 2012-08-03
tags: [Javascript,JS]
categories: [Javascript]
summary: js中parseInt用法和parseInt("08")问题
---


# js中parseInt用法和parseInt("08")问题

* parseInt() 函数可解析一个字符串，并返回一个整数。

* 语法

	parseInt(string, radix)
	参数	描述
	string	必需。要被解析的字符串。
	radix	可选。表示要解析的数字的基数。该值介于 2 ~ 36 之间。
	如果省略该参数或其值为 0，则数字将以 10 为基础来解析。如果它以 “0x” 或 “0X” 开头，将以 16 为基数。

	如果该参数小于 2 或者大于 36，则 parseInt() 将返回 NaN。

	返回值

	返回解析后的数字。

* 说明

	当参数 radix 的值为 0，或没有设置该参数时，parseInt() 会根据 string 来判断数字的基数。

	举例，如果 string 以 “0x” 开头，parseInt() 会把 string 的其余部分解析为十六进制的整数。如果 string 以 0 开头，那么 ECMAScript v3 允许 parseInt() 的一个实现把其后的字符解析为八进制或十六进制的数字。如果 string 以 1 ~ 9 的数字开头，parseInt() 将把它解析为十进制的整数。

* 提示和注释

注释：只有字符串中的第一个数字会被返回。

注释：开头和结尾的空格是允许的。

提示：如果字符串的第一个字符不能被转换为数字，那么 parseFloat() 会返回 NaN。

* 实例

在本例中，我们将使用 parseInt() 来解析不同的字符串：
```
parseInt("10");			//返回 10
parseInt("19",10);		//返回 19 (10+9)
parseInt("11",2);		//返回 3 (2+1)
parseInt("17",8);		//返回 15 (8+7)
parseInt("1f",16);		//返回 31 (16+15)
parseInt("010");		//未定：返回 10 或 8
```

	parseInt("08")问题
	对于parseInt("01")到parseInt("07");都能得到正确的结果，但如果是parseInt("08") 或parseInt("09")则返回0；
	首先看parseInt语法：parseInt(string, radix);
	其中string为要转换的字符串，radix为二进制，八进制，十六进制或十进制。
	在默认不指定radix时，当以0x开关时，为十六进制；如果以0开关且第二位不为x,则让为是八进制，（因为八进制不能有8，9所以报错返回0）。
	所以，在我们用时还是明确指定进位制，以防出错。
	如我们平时都用十进制位，我们就 parseInt("08", 10);
	parseInt(number,type)这个函数后面如果不跟第2个参数来表示进制的话，默认是10进制。

	比如说parseInt("010",10)就是10进制的结果：10，parseInt("010",2)就是2进制的结果：2，parseInt("010",8)就是8进制的结果：8，parseInt("010",16)就是2进制的结果：16。

* 下面我来说说没有指定进制单位的时候，默认是10进制，但：如果是里面的Number是0开头的就认为是8进制的，如果是0x开头的就认为是16进制的。
```
parseInt("10")==>parseInt("010",10)===>10

parseInt("010")==>parseInt("010",8)==>8

parseInt("0x10")==>parseInt("010",16)==>16.

```