---
title: Hadoop添加或删除节点
author: Zeeny
comments: false
date: 2015-12-05
updated: 2015-12-05
tags: [Hadoop,大数据]
categories: [大数据]
summary: Hadoop添加或删除节点
---



# Hadoop添加或删除节点

## 删除或下线节点

```
ssh zxsoft-hadoop-master-NameNode
cd /home/hadoop/hadoop-1.2.1/conf/

1. 确定要下架的机器,创建exclude文件
	vi exclude

	添加需要删除的节点，一行一个：
	zxsoft-hadoop-slave-DataNode-TaskTracker-07
	zxsoft-hadoop-slave-DataNode-TaskTracker-08
	
	
2. 修改conf/hdfs-site.xml文件
	vi hdfs-site.xml
		添加：
			<property> 
				<name>dfs.hosts.exclude</name> 
				<value>/home/hadoop/hadoop-1.2.1/conf/exclude</value> 
				<description>Names a file that contains a list of hosts that are not permitted to connect to the namenode. The full pathname of the file must be specified. If the value is empty, no hosts are excluded.</description> 
			</property>
	
	修改core-site.xml
	vi core-site.xml
		添加：
			<property> 
				<name>dfs.hosts.exclude</name> 
				<value>/home/hadoop/hadoop-1.2.1/conf/exclude</value> 
				<description>Names a file that contains a list of hosts that are not permitted to connect to the namenode. The full pathname of the file must be specified. If the value is empty, no hosts are excluded.</description> 
			</property>
	

3. 降低备份系数
	如果下架后，机器不足了，就需要降低备份系数。

	在我的测试环境中，目前节点为4台，备份系数为3，如果去掉一台的话备份系数就与节点数相同了，而Hadoop是不允许的。通常备份系数不需要太高，可以是服务器总量的1/3左右即可，Hadoop默认的数值是3。

	下面，我们将备份系数从3降低到2。

	3.1 在所有的Hadoop服务器上更新以下配置：
		vi hdfs-site.xml
	    <property>
	   		<name>dfs.replication</name>
	   		<value>2</value>
	    </property>

	3.2 登陆zxsoft-hadoop-master-NameNode：修改hdfs文件备份系数，把/ 目录下所有文件备份系数设置为2。
		sudo -u hdfs hadoop dfs -setrep -w 2 -R /
	       
4. 动态刷新配置(强制重新加载配置)
	ssh zxsoft-hadoop-master-NameNode
	./bin/hadoop dfsadmin -refreshNodes


5. 检查节点的处理状态
	通过WEB管理界面查看
	Decommissioning Nodes (退役中节点)
	Dead Nodes （已经下线）
	  
	  
6.检查进程状态 
		这时我们查看进程状态，可以发现datanode进程已经被自动中止了
		jps

		而Tasktracker进程还在，需要我们手动中止
		jps
		./hadoop-daemon.sh stop TaskTracker
		
		此时，即使我们手动启动datanode，也是不能成功的，日志中会显示UnregisteredDatanodeException的错误。
		./hadoop-daemon.sh start DataNode

			显示错误
			此，对Hadoop集群节点的动态删除也已经成功完成了。
	
7.关闭节点
	  等待刚刚的操作结束后，需要下架的机器就可以安全的关闭了。

8. 再次编辑exclude文件 
	   一旦完成了机器下架，它们就可以从exclude文件移除了。
	
需要注意的是：
	在删除节点时一定要停止所有Hadoop的Job，否则程序还会向要删除的节点同步数据，这样也会导致Decommission的过程一直无法完成。	
```


```
start-all.sh 启动所有的Hadoop守护进程。包括NameNode、 Secondary NameNode、DataNode、JobTracker、 TaskTrack
stop-all.sh 停止所有的Hadoop守护进程。包括NameNode、 Secondary NameNode、DataNode、JobTracker、 TaskTrack
start-dfs.sh 启动Hadoop HDFS守护进程NameNode、SecondaryNameNode和DataNode
stop-dfs.sh 停止Hadoop HDFS守护进程NameNode、SecondaryNameNode和DataNode
hadoop-daemon.sh start namenode 单独启动NameNode守护进程
hadoop-daemon.sh stop namenode 单独停止NameNode守护进程
hadoop-daemon.sh start datanode 单独启动DataNode守护进程
hadoop-daemon.sh stop datanode 单独停止DataNode守护进程
hadoop-daemon.sh start secondarynamenode 单独启动SecondaryNameNode守护进程
hadoop-daemon.sh stop secondarynamenode 单独停止SecondaryNameNode守护进程

start-mapred.sh 启动Hadoop MapReduce守护进程JobTracker和TaskTracker
stop-mapred.sh 停止Hadoop MapReduce守护进程JobTracker和TaskTracker
hadoop-daemon.sh start jobtracker 单独启动JobTracker守护进程
hadoop-daemon.sh stop jobtracker 单独停止JobTracker守护进程
hadoop-daemon.sh start tasktracker 单独启动TaskTracker守护进程
hadoop-daemon.sh stop tasktracker 单独启动TaskTracker守护进程
```