---
title: Hadoop使用笔记
author: Zeeny
comments: false
date: 2014-11-21
updated: 2014-11-21
tags: [Hadoop,大数据]
categories: [大数据]
summary: Hadoop使用笔记
---


# Hadoop使用笔记

## 一、运行问题

### 1.Snappy native library not loaded

```
执行：
	../hadoop-1.2.1/bin/hadoop jar wordcount.jar WordCount input output

出现如下警告：
	WARN snappy.LoadSnappy: Snappy native library not loaded
	
解决：
	wget https://snappy.googlecode.com/files/snappy-1.1.1.tar.gz
	tar zxvf snappy-1.1.1.tar.gz
	cd snappy-1.1.1
	./configure
	make
	cd .libs/
	cp ./libsnappy.so* /home/hadoop/hadoop-1.2.1/lib/native/Linux-amd64-64/
	make install
```

## 二、常用方方法##

### 1.运行测试程序

```
1.1) 建立测试目录
	mkdir /home/hadoop/test
	cd /home/hadoop/test
	
1.2) 编写测试程序
	cd /home/hadoop/test
	WordCount.java
	
1.3) 编译WordCount.java
	cd /home/hadoop/test
	mkdir WordCount
	javac -cp /home/hadoop/hadoop-1.2.1/hadoop-core-1.2.1.jar -d ./WordCount ./WordCount.java 
	jar -cvf wordcount.jar -C WordCount/ .
	
1.4) 建立（准备）输入文件
	cd /home/hadoop/test
	echo "Hello World Bye World" > file01
	echo "Hello Hadoop GoodBye Hadoop" > file02
	
1.5) 上传输入文件(file01 file02)
	cd /home/hadoop/test
	../hadoop-1.2.1/bin/hadoop dfs -mkdir input
	../hadoop-1.2.1/bin/hadoop dfs -put ./file0* input
     #将本地文件系统上的./file0*（file01, file02）拷贝到HDFS的根目录下，目录名为input。
     
     
1.6) 运行 wordcount.jar
	cd /home/hadoop/test
	../hadoop-1.2.1/bin/hadoop jar wordcount.jar WordCount input output
		#执行测试任务，输出到output。
    
1.7）查看输出文件
	cd /home/hadoop/test
	../hadoop-1.2.1/bin/hadoop fs -ls output

	查看结果:
	../hadoop-1.2.1/bin/hadoop fs -cat output/*
	../hadoop-1.2.1/bin/hadoop fs -cat output/part-00000
    
1.8）将结果从HDFS复制到本地再查看
	../hadoop-1.2.1/bin/hadoop fs -get output ./output
    
    
1.9) 备注：
	../hadoop-1.2.1/bin/hadoop fs –help 可以了解各种 HDFS命令的使用。 
```

### 2.HDFS的Shell操作

```
2.1) hadoop fs -ls /     查看文件根目录
	hadoop fs -ls /home   查看指定文件目录
	hadoop fs -lsr /  递归查看根目录
		 
2.2) 在hdfs上创建文件夹
	hadoop fs -mkdir /demo1

2.3) 上传文件
	hadoop fs -put /root/test.txt /demo1   把/root/test.txt 文件上传到hdfs上的/demo1目录下
	如果/demo1目录不存在的话，表示的是上传后的文件名称。
    	
2.4) 下载文件
	hadoop fs -get /demo1/text.txt ./    下载到当前目录下
        
2.5) 查看hdfs中的文件
	hadoop fs -text <hdfs文件>
    
2.6) 删除hdfs 中的文件
	hadoop fs -rm /demo1/text.txt
	hadoop fs -rmr /demo1 删除目录
         
2.7) hadoop fs -help
	hadoop fs -help cp
	hadoop fs -help ls  
```