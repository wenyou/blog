---
title: HBase使用总结(二)
author: Zeeny
comments: false
date: 2014-12-16
updated: 2014-12-16
tags: [HBase,大数据]
categories: [大数据]
summary: HBase使用总结(二)
---


# HBase使用总结(二)

## 2.Hbase实践经验

### 2.1 合理设计rowKey & Pre-Sharding

* 避免仅操作集群中的少数几台机器；
* 根据数据量、region server数合理pre-sharding。

### 2.2 容量影响因素

* 开启压缩
* lzo

```
create table 't1',{NAME => 'cf1', COMPRESSION => 'lzo'}
```

### 2.3 写速度关键因素

* Table region分布均衡；
* 单台region server的region数；
* hbase.regionserver.handler.count
* hbase.regionserver.global.memstore.upperLimit
* hbase.hregion.memstore.block.multiplier
* hbase.hstore.blockingStoreFiles
* hbase.hregion.max.filesize

### 2.4 读速度关键因素

* 单台Region Server上的Region数；
* StoreFile数；
* bloomfilter；
* in-memory flag;
* blockcache设置；
* hfile.block.cache.size；

### 2.5 二级索引

* 合理使用三维有序
* 冗余
* 离线

## 3、知识点

* hadoop，hbase，需要在各自的集群中每个节点都安装，zookeeper根据需要安装，一般奇数个，数量越多，选举负担重，但数量越少，系统稳定性下降，使用时跟据实际情况选择方案，hive，oozie，sqoop只需要在需要执行客户端程序的机器上安装，只要能连上hadoop。

* zk的连接数要改的大一点，默认是30个，并且尽量与hadoop node节点分开，因为hadoop的暂时负担过重等异常会严重影响zk与hbase的正常工作，比如导致zk长时间选举不出leader，hbase 各节点会相继挂掉。