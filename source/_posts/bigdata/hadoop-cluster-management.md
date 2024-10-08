---
title: Hadoop集群的管理
author: Zeeny
comments: false
date: 2014-11-27
updated: 2014-11-27
tags: [Hadoop,大数据]
categories: [大数据]
summary: Hadoop集群的管理
---


# Hadoop集群的管理

## 一、服务管理

__1、开启hadoop集群__

* hadoop->zookeeper->hbase

1.1）、起动hadoop：

* /home/hadoop/hadoop-1.2.1/bin/start-dfs.sh （master-NameNode节点）
* /home/hadoop/hadoop-1.2.1/bin/start-mapred.sh （master-JobTracker节点）

1.2）、开启zookeeper （所有节点）

* ssh到所有节点，zkServer.sh start
* /home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/bin/zkServer.sh start

1.3）、起动hbase (hbase master节点):

* ssh root@master-NameNode
* /home/hadoop/hadoop-1.2.1/hbase-0.98.8/bin/start-hbase.sh

__2、关闭hadoop集群__

* hbase->zookeeper->hadoop

2.1）、停止hbase(master节点):

* ssh root@master-NameNode
* /home/hadoop/hadoop-1.2.1/hbase-0.98.8/bin/stop-hbase.sh

2.2）、停止zookeeper（所有节点）：

* ssh到所有节点，zkServer.sh stop
* /home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/bin/zkServer.sh stop

2.3）、停止hadoop:
* /home/hadoop/hadoop-1.2.1/bin/stop-mapred.sh （master-JobTracker节点）
* /home/hadoop/hadoop-1.2.1/bin/stop-dfs.sh （master-NameNode节点）


__3、其他服务管理__


3.1)、单独在要启动的RegionServer上执行以下命令

```
./hbase-0.98.8/bin/hbase-daemon.sh start regionserver
	
```

3.2)、linux设置时间服务器

* 一般来说，对一个机房内的设备，可以设置一台时间服务器，由它定期从一个标准的时间服务器上获取时间。其他的服务器可以通过内网的连接从这台服务器进行同步。

```
1. 确定ntp包已经安装
	 #rpm -qf /usr/sbin/ntpd
	 
2. 启动服务器，如果ntpd已经安装，则可以直接启动：
	#service ntpd start
	
3. 设置ntpd自启动：
	#find  /etc/rc.d/ -name "*ntpd"
	#/sbin/chkconfig --level 345 ntpd on
	# !find
		结果：
		find  /etc/rc.d/ -name "*ntpd"
		/etc/rc.d/rc4.d/S58ntpd
		/etc/rc.d/rc2.d/K74ntpd
		/etc/rc.d/rc5.d/S58ntpd
		/etc/rc.d/init.d/ntpd
		/etc/rc.d/rc3.d/S58ntpd
		/etc/rc.d/rc6.d/K74ntpd
		/etc/rc.d/rc1.d/K74ntpd
		/etc/rc.d/rc0.d/K74ntpd
		
		说明在3，4，5三个级别已经可以自启动。
		
4. 检查防火墙
	 # iptables -L
	 对比较严格的防火墙，应该对ntp端口123进行配置：
	 # iptables -A INPUT -p udp --dport 123 -j ACCEPT
	 
5. 客户端配置
	 客户端采用ntpdate来更新，配置在crontab中。根据需要决定频率。在每一台需要同步时间的设备上设置crontab
	 # crontab -e
	 
	 00 00 * * * /usr/sbin/ntpdate 172.16.37.21
	 
	 crontab 设置的是每天0点同步时间。
	 
	 为了保证时间服务器可用，将命令先在命令行下执行一下。
	 
	 #ntpdate 172.16.37.21
```

3.3)、开启、恢复重建hdfs中的回收站

```
在每个节点(不仅仅是主节点)上添加配置 /home/hadoop/hadoop-1.2.1/conf/core-site.xml,增加如下内容:
	<property>
		<name>fs.trash.interval</name>
		<value>1440</value>
	</property>
    
重启hadoop服务。	
    
删除一个hdfs文件测试一下，就可以了。
```

__4、mysql管理__

```
启动mysql:
	/etc/init.d/mysql start
	
	/etc/init.d/mysql restart
	
	/etc/init.d/mysql stop
	
登陆mysql shell:	
	mysql -uroot -p123456 
```

### Mysql系统：

* zxsoft-hadoop-MysqlServer
* user/pwd: root/123456
* /etc/init.d/mysql start/restart/stop
* mysql -uroot -p123456

```
#mysql -uroot -p123456
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'172.16.37.21' IDENTIFIED BY '123456' WITH GRANT OPTION;
mysql> flush privileges;
	

设置mysql开机启动：
	# chkconfig --level 345 mysql on
	# /etc/init.d/mysql start
```

### 其他配置信息
```
172.16.37.21 -  172.16.37.27
hadoop安装目录均为：/home/hadoop/...    ,通常情况下不要动。
hadoop hdfs namenode 在 172.16.37.21  上，状态查看：http://172.16.37.21:50070/。
secondary-namenode 在172.16.37.22上。
job-tracker  在172.16.37.23上，状态查看：http://172.16.37.23:50030/。
172.16.37.24 - 27 是datanode  和 tasktracker

hbase master 目前也放在了 172.16.37.21，状态查看：http://172.16.37.21:60010/master-status

hive 目前也放在了 172.16.37.21 上，hive web界面查看：http://172.16.37.21:9999/hwi/

172.16.37.25  读取webservice,发送udp包

172.16.37.26  接收udp包，存入hdfs系统。
```