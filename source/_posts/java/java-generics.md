---
title: Java 泛型
author: Zeeny
comments: false
date: 2017-02-07
updated: 2017-02-07
tags: [Java]
categories: Java
summary: Java 泛型
---


# 一、有限制的通配符

## 1.无界通配符

* `<?>` 允许所有泛型的引用调用。称为 无界通配符。


## 2.上界通配符

* `<? extends XXX>` 上限 extends ：使用时指定的类型必须是继承某个类(XXX)，或者实现了某个接口(XxX)。 即 <= 。
* 比如:

```
//(无穷小, Person] 只允许泛型为 Person 以及 Person 子类的引用调用
<? extends Person>;  

// 只允许泛型为实现 Comparable 接口的实现类的引用调用
<? extends Comparable>;  
```


## 3.下界通配符

* `<? super XXX>` 下限 super：使用时指定的类型不能小于操作的类 XXX，即 >=。
* 比如:

```
//[Person, 无穷大] 只允许泛型为 Person 及  Person父类的引用调用
<? super Person>;
```
