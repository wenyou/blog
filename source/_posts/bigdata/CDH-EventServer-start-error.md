---
title: CDH Cloudera Management Service启动出现Event Server启动失败
author: Zeeny
comments: false
date: 2024-11-08 13:00
updated: 2024-11-08 13:00
tags: [CDH,大数据]
categories: [大数据]
summary: CDH Cloudera Management Service启动出现Event Server启动失败,Event Server启动错误解决
---

# CDH Cloudera Management Service启动出现Event Server启动失败,Event Server启动错误解决

### CDH Cloudera Management Service启动出现Event Server启动失败
* mgmt/mgmt.sh ["eventserver"] 失败！

* 对角色 Event Server (xxx-cdh-developenv) 执行命令 启动此 Event Server 启动角色失败。

* 启动此 Event Server  启动角色失败。
![启动角色失败](/images/cdh-eventserver-error-1.jpg)
![启动角色失败](/images/cdh-eventserver-error-2.jpg)

### 查看错误日志文件
* 查看错误日志文件  /home/cloudera/logs/cloudera-scm-eventserver/mgmt-cmf-mgmt-EVENTSERVER-zxsoft-cdh-developenv.log.out

> 2024-11-07 17:59:15,442 INFO com.cloudera.cmf.eventcatcher.server.EventCatcherService: Successfully initialized non-built-in monitoring types
> 2024-11-07 17:59:15,448 INFO com.cloudera.cmf.PollingScmProxy: Polling interval is 1000 ms. Impala authentication cache validity is 120000 ms. Zookeeper authentication cache validity is 120000ms. Fetching metric schema is: false
> 2024-11-07 17:59:15,560 INFO com.cloudera.cmf.eventcatcher.upgrade.IndexVersionManager: Creating IndexVersionManager for /home/cloudera/datas/cloudera-scm-eventserver
> 2024-11-07 17:59:15,560 INFO com.cloudera.cmf.eventcatcher.upgrade.IndexVersionManager: Checking for version directory at /home/cloudera/datas/cloudera-scm-eventserver/version
> 2024-11-07 17:59:15,560 INFO com.cloudera.cmf.eventcatcher.upgrade.IndexVersionManager: Checking for version file at /home/cloudera/datas/cloudera-scm-eventserver/version/event_schema.version
> 2024-11-07 17:59:15,564 INFO com.cloudera.cmf.eventcatcher.upgrade.IndexVersionManager: Version from file is 3
> 2024-11-07 17:59:15,564 INFO com.cloudera.cmf.eventcatcher.upgrade.IndexVersionManager: No events index upgrade is required.
> 2024-11-07 17:59:15,564 INFO com.cloudera.enterprise.EnterpriseService: Starting com.cloudera.cmf.PollingScmProxy
> 2024-11-07 17:59:15,632 INFO com.cloudera.cmf.BasicScmProxy: Using encrypted credentials for SCM
> 2024-11-07 17:59:15,635 INFO com.cloudera.cmf.BasicScmProxy: Authenticated to SCM.
> 2024-11-07 17:59:19,704 ERROR com.cloudera.cmf.eventcatcher.server.EventCatcherService: Error starting EventServer
> java.io.IOException: read past EOF
> at org.apache.lucene.store.MMapDirectory$MMapIndexInput.readByte(MMapDirectory.java:278)
> at org.apache.lucene.store.ChecksumIndexInput.readByte(ChecksumIndexInput.java:40)
> at org.apache.lucene.store.DataInput.readInt(DataInput.java:84)
> at org.apache.lucene.index.SegmentInfos.read(SegmentInfos.java:272)
> at org.apache.lucene.index.IndexFileDeleter.<init>(IndexFileDeleter.java:177)
> at org.apache.lucene.index.IndexWriter.<init>(IndexWriter.java:1182)
> at com.cloudera.cmf.eventcatcher.server.SingleIndexManager.makeIndexWriter(SingleIndexManager.java:139)
> at com.cloudera.cmf.eventcatcher.server.SingleIndexManager.<init>(SingleIndexManager.java:112)
> at com.cloudera.cmf.eventcatcher.server.EventCatcherService.<init>(EventCatcherService.java:282)
> at com.cloudera.cmf.eventcatcher.server.EventCatcherService.main(EventCatcherService.java:148)

### 错误原因：
* 创建索引失败，由日志错误信息可知在“/home/cloudera/datas/cloudera-scm-eventserver” 目录下创建索引失败。

### 解决方法：
* 进入CDH管理界面，打开 Cloudera Management Service 管理，查看配置，查看Event Server配置，Event Server 索引目录 选项，发现配置值与上述日志信息一样都是：/home/cloudera/datas/cloudera-scm-eventserver。

![Event Server配置](/images/cdh-eventserver-error-3.jpg)

1. 先关闭CDH所有服务和停止Cloudera Management Service。
2. 进入服务器 /home/cloudera/datas/cloudera-scm-eventserver 目录，删除目录下所有文件：rm -rf ./*
3. 再次开启Cloudera Management Service服务，启动 Event Server。
4. 观察 Event Server 日志（/home/cloudera/logs/cloudera-scm-eventserver），查看有无错误，确认是否成功。

![Event Server 日志配置](/images/cdh-eventserver-error-4.jpg)

5. 看到如下界面，证明解决成功。
![Event Server状态](/images/cdh-eventserver-error-5.jpg)