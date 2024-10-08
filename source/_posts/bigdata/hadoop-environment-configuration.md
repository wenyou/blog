---
title: Hadoop环境配置
author: Zeeny
comments: false
date: 2014-11-11
updated: 2014-11-11
tags: [Hadoop,大数据]
categories: [大数据]
summary: Hadoop环境配置
---


# Hadoop环境配置

## 配置案例

### 1、搭建一个有三台主机的小集群

```
为这三台机器分配ip地址及相应的角色:
	172.16.37.10 - master, NameNode, Jobtracker-master
	172.16.37.9  - slave,DataNode, tasktracker-slave1	172.16.37.8  - slave,DataNode, tasktracker-slave2
	
首先在三台主机上创建相同的用户：/home/zeeny

1) 在三台主机上安装JDK1.6，并设置环境变量。 

	2) 在这三台主机上安装OpenSSH，并配置SSH可以无密码登录。（无密码登录可配可不配，主要是为了有很多主机时启动方便。）

	   2.1) 在所有的Master和Slave机器上执行：ssh-keygen -t rsa  命令，执行此命令的时候，看到提示时只需要回车。然后就会在/root/.ssh/下面产生id_rsa.pub的证书文件。

	   2.2）通过scp将Master(172.16.37.21)机器上的这个文件拷贝到Slave上(记得修改名称)：scp ~/.ssh/id_rsa.pub root@172.16.37.22:/root/.ssh/id_rsa21.pub

	   2.3）然后在slave（172.16.37.22）上执行：cat ~/.ssh/id_rsa21.pub >> ~/.ssh/authorized_keys ，建立authorized_keys文件即可。

	   2.4）此时可以试验一下，从master(172.16.37.21)机上ssh到slave: ssh 172.16.37.22已经不需要密码了。由slave反向建立也是同样操作。
	   
		 2.5) 本机 SSH
	        在本机上将公钥~/.ssh/id_rsa.pub加入授权列表~/.ssh/authorized_keys中：
	        cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
	        chmod 755 ~/.ssh
	        chmod 600 ~/.ssh/authorized_keys
	        测试：ssh localhost 本机ssh登陆本机也无需要密码
	   
	
	3) 在三台主机上分别设置/etc/hosts 及 /etc/hostname

		/etc/hosts:
		127.0.0.1    localhost
		172.16.37.10 master-win7-VMware-centos-37-10
		172.16.37.9 slave1-win7-VMware-centos-37-9
		172.16.37.8 slave2-win7-VMware-centos-37-8
	
		/etc/hostname这个文件用于定义Ubuntu的主机名。
		/etc/hostname:
		你的主机名(如master, slave1等)
	
		而centos定义主机名，需要通过如下方式：
		172.16.37.10主机：
		vi /etc/sysconfig/network
		HOSTNAME=master-win7-VMware-centos-37-10
	
		172.16.37.9主机：
		vi /etc/sysconfig/network
		HOSTNAME=slave1-win7-VMware-centos-37-9
		
		172.16.37.8主机：
		vi /etc/sysconfig/network
		HOSTNAME=slave2-win7-VMware-centos-37-8
	

	4）配置三台主机的Hadop文件：
		cd /home/zeeny/hadoop-2.5.1
	
		4.1) 在/etc/profile最后添加：
	
			HADOOP_PREFIX=/home/zeeny/hadoop-2.5.1
			YARN_CONF_DIR=/home/zeeny/hadoop-2.5.1
		
			#PATH最后面增加$HADOOP_PREFIX/bin  这里要注意，和前面的属性要用:隔开
			#然后：
			export HADOOP_PREFIX  
			export YARN_CONF_DIR
			export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_PREFIX/lib/native
			export HADOOP_OPTS="-Djava.library.path=$HADOOP_PREFIX/lib"
	
			完整的配置如下：
			vi /etc/profile:
			JAVA_HOME=/usr/java/jdk1.6.0_45
			JRE_HOME=/usr/java/jdk1.6.0_45/jre

			HADOOP_PREFIX=/home/zeeny/hadoop-2.5.1
			YARN_CONF_DIR=/home/zeeny/hadoop-2.5.1

			PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_PREFIX/bin
			CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
			export JAVA_HOME JRE_HOME PATH CLASSPATH

			export HADOOP_PREFIX
			export YARN_CONF_DIR
			export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_PREFIX/lib/native
			export HADOOP_OPTS="-Djava.library.path=$HADOOP_PREFIX/lib"
	
			最后，source /etc/profile 使配置生效。
	
		4.2) hadoop-env.sh
			cd /home/zeeny/hadoop-2.5.1/etc/hadoop
			vi hadoop-env.sh :
			export JAVA_HOME="/usr/java/jdk1.6.0_45"  
	
		4.3) yarn-env.sh
			vi yarn-env.sh :
			export JAVA_HOME="/usr/java/jdk1.6.0_45"
	
		4.4) core-site.xml
			vi core-site.xml :
			<configuration>
				<property>
					<name>fs.default.name</name>
					<value>hdfs://master-win7-VMware-centos-37-10:9000</value> #hdfs:localhost:9000
				</property>
				<property>
					<name>Hadoop.tmp.dir</name>
					<value>/home/zeeny/HadoopData/tmp</value> #你希望Hadoop存储数据块的位置，此文件夹需要手动创建
				</property>
			</configuration>
	
		4.5) hdfs-site.xml

			vi hdfs-site.xml :
			<configuration>
				<property>
					<name>dfs.replication</name>
					<value>2</value>
				</property>
			</configuration>
	
		4.6) mapred-site.xml

			vi mapred-site.xml :
			<configuration>
				<property>
					<name>mapred.job.tracker</name>
					<value>master-win7-VMware-centos-37-10:9001</value>
				</property>
			</configuration>
	
		4.7) masters
			vi masters :
			master-win7-VMware-centos-37-10
	
		4.8) slaves
			vi slaves :
			slave1-win7-VMware-centos-37-9
			slave2-win7-VMware-centos-37-8
	
	5) 格式化
		cd /home/zeeny/hadoop-2.5.1
		bin/hdfs namenode -format
		//这句命令基本上就是把目前你的hadoop系统确定一下结构，我们的hadoop系统中，一般是一个namenode+多个datanode。namenode相当于顾客房间表，datanode相当于具体的房间。
	 
	6) 启动Hadoop
		cd /home/zeeny/hadoop-2.5.1
		sbin/start-dfs.sh 
		sbin/start-yarn.sh
	

		查看集群状态：
		网页方式：http://172.16.37.10:50070/
		命令方式：bin/hdfs dfsadmin -report
	
	
	7) 关闭Hadoop
		cd /home/zeeny/hadoop-2.5.1
		sbin/stop-dfs.sh 
		sbin/stop-yarn.sh
```

### 2、搭建一个多节点全分布模式的集群,三个主节点

```
三个主节点：
	一个NameNode
	一个SecondaryNameNode
	一个JobTracker
	   
	五个从节点（DataNode + TaskTracker）
	
	八台机配置如下：
		172.16.37.21
		172.16.37.22
		172.16.37.23
		172.16.37.24
		172.16.37.25
		172.16.37.26
		172.16.37.27
		172.16.37.10
	
步骤：
1、在所有节点机上安装常用支持软件:
		gcc :yum install -y gcc
		g++: yum install -y gcc-c++
		ntp: yum install -y ntp
					ntpdate -u pool.ntp.org/time.nist.gov
		scp/ssh: yum install openssl openssl-devel openssh-clients
		wget: yum install wget
		rsync: yum install -y rsync
		unzip: yum install -y unzip
	
2、安装jdk1.7
		rpm -ivh jdk-1.7.0_71-linux-x64.rpm
	
3、建立hadoop目录
		mkdir /home/hadoop
	
4、安装hadoop1.2.1
		tar zxvf hadoop-1.2.1-bin.tar.gz
		mv ./hadoop-1.2.1 /home/hadoop/
	
5、配置环境变量：
		vi /etc/profile   

			JAVA_HOME=/usr/java/jdk1.7.0_71
			JRE_HOME=/usr/java/jdk1.7.0_71/jre

			HADOOP_PREFIX=/home/hadoop/hadoop-1.2.1
			YARN_CONF_DIR=/home/hadoop/hadoop-1.2.1

			PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_PREFIX/bin
			CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
			export JAVA_HOME JRE_HOME PATH CLASSPATH

			export HADOOP_PREFIX
			export YARN_CONF_DIR
			export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_PREFIX/lib/native
			export HADOOP_OPTS="-Djava.library.path=$HADOOP_PREFIX/lib"

		执行：source /etc/profile 使生效。
        	
6、在所有节点主机上分别设置/etc/hosts 及 定义主机名
	/etc/hosts:
		127.0.0.1    localhost
		172.16.37.21 master-NameNode
		172.16.37.22 master-SecondaryNameNode
		172.16.37.23 master-JobTracker
		172.16.37.24 slave1-DataNode-TaskTracker
		172.16.37.25 slave2-DataNode-TaskTracker
		172.16.37.26 slave3-DataNode-TaskTracker
		172.16.37.27 slave4-DataNode-TaskTracker
		172.16.37.10 slave5-DataNode-TaskTracker
	
	定义主机名：
		172.16.37.21主机：
		vi /etc/sysconfig/network
			HOSTNAME=master-NameNode

		172.16.37.22主机：
		vi /etc/sysconfig/network
			HOSTNAME=master-SecondaryNameNode
		
		172.16.37.23主机：
		vi /etc/sysconfig/network
			HOSTNAME=master-JobTracker
		
		172.16.37.24主机：
		vi /etc/sysconfig/network
			HOSTNAME=slave1-DataNode-TaskTracker

		172.16.37.25主机：
		vi /etc/sysconfig/network
			HOSTNAME=slave2-DataNode-TaskTracker

		172.16.37.26主机：
		vi /etc/sysconfig/network
			HOSTNAME=slave3-DataNode-TaskTracker
		
		172.16.37.27主机：
		vi /etc/sysconfig/network
			HOSTNAME=slave4-DataNode-TaskTracker
		
		172.16.37.10主机：
		vi /etc/sysconfig/network
			HOSTNAME=slave5-DataNode-TaskTracker
	
7、配置 hadoop-env.sh （在所有节点主机上都需要这样配）
		cd /home/hadoop/hadoop-1.2.1/conf/
		vi hadoop-env.sh :
			export JAVA_HOME=/usr/java/jdk1.7.0_71
	
8、配置 core-site.xml
	cd /home/hadoop/hadoop-1.2.1/conf/
	vi core-site.xml :

	8.1）、master-NameNode主机的core-site.xml配置如下:
		<configuration>
			<property>
					<name>fs.default.name</name>
					<value>hdfs://master-NameNode:9000</value>
				</property>
				<property>
					<name>hadoop.tmp.dir</name>
					<value>/home/hadoop/hadoopData/tmp</value>
				</property>
				<property>
					<name>io.file.buffer.size</name>
					<value>65536</value>
				</property>
				<property>
					<name>hadoop.proxyuser.hadoop.hosts</name>
					<value>*</value>
				</property>
				<property>
					<name>hadoop.proxyuser.hadoop.groups</name>
					<value>*</value>
				</property>
			</configuration>

	8.2）、master-SecondaryNameNode主机的core-site.xml配置如下:
		<configuration>
			<property>
				<name>fs.default.name</name>
				<value>hdfs://master-NameNode:9000</value>
			</property>
			<property>
				<name>hadoop.tmp.dir</name>
				<value>/home/hadoop/hadoopData/tmp</value>
			</property>
			<property>
				<name>io.file.buffer.size</name>
				<value>65536</value>
			</property>
			<property>
				<name>hadoop.proxyuser.hadoop.hosts</name>
				<value>*</value>
			</property>   
			<property>
				<name>hadoop.proxyuser.hadoop.groups</name>
				<value>*</value>
			</property>
		</configuration>

	8.3）、master-JobTracker主机的core-site.xml配置如下:
		<configuration>
			<property>
				<name>hadoop.tmp.dir</name>
				<value>/home/hadoop/hadoopData/tmp</value>
			</property>
			<property>
				<name>io.file.buffer.size</name>
				<value>65536</value>
			</property>
			<property>
				<name>hadoop.proxyuser.hadoop.hosts</name>
				<value>*</value>
			</property>
			<property>
				<name>hadoop.proxyuser.hadoop.groups</name>
				<value>*</value>
			</property>
			<property>
				<name>fs.default.name</name>
				<value>hdfs://master-NameNode:9000</value>
			</property>
		</configuration>

	8.4）、slave1-DataNode-TaskTracker主机的core-site.xml配置如下:
		#其他slave节点主机配置相同。
		<configuration>
			<property>
				<name>fs.default.name</name>
				<value>hdfs://master-NameNode:9000</value>
			</property>
			<property>
				<name>hadoop.tmp.dir</name>
				<value>/home/hadoop/hadoopData/tmp</value>
				<final>true</final>
			</property>
			<property>
				<name>io.file.buffer.size</name>
				<value>65536</value>
			</property>
			<property>
				<name>io.compression.codecs</name>
				<value>org.apache.hadoop.io.compress.DefaultCodec,org.apache.hadoop.io.compress.GzipCodec,org.apache.hadoop.io.compress.BZip2Codec,com.hadoop.compression.lzo.LzoCodec,com.hadoop.compression.lzo.LzopCodec</value>
			</property>
			<property>
				<name>io.compression.codec.lzo.class</name>
				<value>com.hadoop.compression.lzo.LzopCodec</value>
			</property>
			<property>
				<name>hadoop.proxyuser.hadoop.hosts</name>
				<value>*</value>
			</property>
			<property>
				<name>hadoop.proxyuser.hadoop.groups</name>
				<value>*</value>
			</property>	
		</configuration>

		#############如果各节点使用相同配置的话，配置如下#########################
		<configuration>
			<property>
				<name>fs.default.name</name> #通过这里指定 NameNode 的位置。我们可以在这里手动指定端口号，也可以忽略以使用默认的端口号。
				<value>hdfs://master-NameNode:9000</value> #hdfs:172.16.37.21:9000
			</property>
			<property>
				<name>hadoop.tmp.dir</name>
				<value>/home/hadoop/hadoopData/tmp</value> #你希望Hadoop存储数据块的位置，此文件夹需要手动创建
			</property>
		</configuration>
	
9、配置 hdfs-site.xml
		cd /home/hadoop/hadoop-1.2.1/conf/
		vi hdfs-site.xml:

	9.1）、master-NameNode主机的hdfs-site.xml配置如下:
		<configuration>
			<property>
				<name>dfs.replication</name>
				<value>3</value>
			</property>
			<property>
				<name>dfs.permissions</name>
				<value>false</value>
			</property>
			<!--
			<property>
				<name>dfs.block.size</name>
				<value>134217728</value> #block越小namenode的压力越大，当然也不是block块越大越好。默认为64M,一般设为128M,256M也可以
			</property>
			-->
			<property>
				<name>dfs.name.dir</name>
				<value>/home/hadoop/hadoopData/hdfs/name,/var/hadoopData/hdfs/name</value> #冗余备份namenode hdfs fsimage, 也可以使用网络备份nfs系统
				<final>true</final>
			</property>
			<!--
			<property>
				<name>dfs.data.dir</name>
				<value>/home/hadoop/hadoopData/hdfs/data</value>
				<final>true</final>
			</property>
			-->
			<property>
				<name>dfs.secondary.http.address</name>
				<value>master-SecondaryNameNode:50090</value>
				<description>secondarynamenode path</description>
			</property>
			<property>
				<name>dfs.http.address</name>
				<value>master-NameNode:50070</value>
				<description>namenode path</description>
			</property>
		</configuration>
	
	9.2）、master-SecondaryNameNode 主机配置如下:
		<configuration>
			<property>
				<name>dfs.permissions</name> 
				<value>false</value>
			</property>
			<property>
				<name>dfs.name.dir</name>
				<value>/home/hadoop/hadoopData/hdfs/name,/var/hadoopData/hdfs/name</value>
				<final>true</final>
			</property>
			<property>
				<name>dfs.data.dir</name>
				<value>/home/hadoop/hadoopData/hdfs/data</value>
				<final>true</final>
			</property>
			<property>
				<name>dfs.replication</name>
				<value>3</value>
			</property>
			<!--
			<property>
				<name>dfs.block.size</name>
				<value>134217728</value>
			</property>
			-->
			<property>
				<name>dfs.secondary.http.address</name>
				<value>master-SecondaryNameNode:50090</value>
				<description>secondarynamenode path</description>
			</property>
			<property>
				<name>dfs.http.address</name>
				<value>master-NameNode:50070</value>
				<description>namenode path</description>
			</property>
		</configuration>
	
	9.3）、master-JobTracker主机的hdfs-site.xml配置如下:
		<configuration>
			<property>
				<name>dfs.permissions</name>
				<value>false</value>
			</property>
			<property>
				<name>dfs.name.dir</name>
				<value>/home/hadoop/hadoopData/hdfs/name,/var/hadoopData/hdfs/name</value>
				<final>true</final>
			</property>
			<property>
				<name>dfs.data.dir</name>
				<value>/home/hadoop/hadoopData/hdfs/data</value>
				<final>true</final>
			</property>
			<property>
				<name>dfs.replication</name>
				<value>3</value>
			</property>
			<!--
			<property>
				<name>dfs.block.size</name>
				<value>134217728</value>
			</property>
			-->
			<property>
				<name>dfs.secondary.http.address</name>
				<value>master-SecondaryNameNode:50090</value>
				<description>secondarynamenode path</description>
			</property>
			<property>
				<name>dfs.http.address</name>
				<value>master-NameNode:50070</value>
				<description>namenode path</description>
			</property>
		</configuration>
	
	9.4）、slave1-DataNode-TaskTracker主机的hdfs-site.xml配置如下:
		#其他slave节点主机配置相同。
		<configuration>
			<property>
				<name>dfs.permissions</name>
				<value>false</value>
			</property>
			<!--
			<property>
				<name>dfs.name.dir</name>
				<value>/home/hadoop/hadoopData/hdfs/name,/var/hadoopData/hdfs/name</value>
				<final>true</final>
			</property>
			-->
			<property>
				<name>dfs.data.dir</name>
				<value>/home/hadoop/hadoopData/hdfs/data</value>
				<final>true</final>
			</property>
			<property>
				<name>dfs.replication</name>
				<value>3</value>
			</property>
			<!--
			<property>
				<name>dfs.block.size</name>
				<value>134217728</value>
			</property>
			-->
			<property>
				<name>dfs.secondary.http.address</name>
				<value>master-SecondaryNameNode:50090</value>
				<description>secondarynamenode path</description>
			</property>
			<property>
				<name>dfs.http.address</name>
				<value>master-NameNode:50070</value>
				<description>namenode path</description>
			</property>
		</configuration>

		##########如果各节点使用相同配置的话，配置如下#######################
		<configuration>
			<property>
				<name>dfs.replication</name> #数据块默认副本份数，不配置默认是3份。
				<value>3</value>
			</property>
			<property>
				<name>dfs.secondary.http.address</name>
				<value>master-SecondaryNameNode:50090</value>##如果secondarynamenode为多个话可以设置为0.0.0.0:50090
				<description>secondarynamenode path</description>
			</property>
			<property>
				<name>dfs.http.address</name>
				<value>master-NameNode:50070</value>
				<description>namenode path</description>
			</property>
			<property>
				<name>dfs.name.dir</name>
				<value>/home/hadoop/hadoopData/hdfs/name,/var/hadoopData/hdfs/name</value>#${HADOOP_NN_DIR}
				<description>Determines where on the local filesystem the DFS name node should store the name table.  If this is a comma-delimited list of directories then the name table is replicated in all of the directories, for redundancy. </description>
				<final>true</final>
			</property>
			<property>
				<name>dfs.data.dir</name>
				<value>/home/hadoop/hadoopData/hdfs/data</value>#${HADOOP_DN_DIR},data node的数据目录,以,号隔开,hdfs会把数据存在这些目录下，一般这些目录是不同的块设备，不存在的目录会被忽略掉
				<description>Determines where on the local filesystem an DFS data node should store its blocks.  If this is a comma-delimited list of directories, then data will be stored in all named directories, typically on different devices.  Directories that do not exist are ignored.</description>
				<final>true</final>
				</property>
			</configuration>
	
10、配置 mapred-site.xml 
	cd /home/hadoop/hadoop-1.2.1/conf/
	vi mapred-site.xml :
	
	10.1）、master-NameNode 主机的mapred-site.xml配置如下:
		<configuration>
			<property>
				<name>mapred.child.java.opts</name>
				<value>-Xmx1024m</value>
			</property>
			<property>
				<name>io.sort.mb</name>
				<value>500m</value>
			</property>
			<property>
				<name>mapred.tasktracker.map.tasks.maximum</name>
				<value>4</value>
				<final>true</final>
			</property>
			<property>
				<name>mapred.tasktracker.reduce.tasks.maximum</name>
				<value>2</value>
				<final>true</final>
			</property>
			<property>
				<name>mapred.job.tracker</name>
				<value>master-JobTracker:9001</value>
				<final>true</final>
			</property>
		</configuration>
	
	10.2）、master-SecondaryNameNode 主机的mapred-site.xml配置如下:
			<configuration>
				<property>
					<name>mapred.child.java.opts</name>
					<value>-Xmx1024m</value>
				</property>
				<property>
					<name>io.sort.mb</name>
					<value>500m</value>
				</property>
				<property>
					<name>mapred.tasktracker.map.tasks.maximum</name>
					<value>4</value>
					<final>true</final>
				</property>
				<property>
					<name>mapred.tasktracker.reduce.tasks.maximum</name>
					<value>2</value>
					<final>true</final>
				</property>
				<property>
					<name>mapred.job.tracker</name>
					<value>master-JobTracker:9001</value>
					<final>true</final> 
				</property>
			</configuration>

	10.3）、master-JobTracker主机的mapred-site.xml配置如下:
			<configuration>
				<property>
					<name>mapred.child.java.opts</name>
					<value>-Xmx1024m</value>
				</property>
				<property>
					<name>io.sort.mb</name>
					<value>512m</value>
				</property>
				<property>
					<name>mapred.tasktracker.map.tasks.maximum</name>
					<value>4</value>
					<final>true</final>
				</property>
				<property>
					<name>mapred.tasktracker.reduce.tasks.maximum</name>
					<value>2</value>
					<final>true</final>
				</property>
				<property>
					<name>mapred.job.tracker</name>
					<value>master-JobTracker:9001</value>
					<final>true</final>
				</property>
				<!--
				<property>
					<name>mapred.task.timeout</name>
					<value>6000000</value>
				</property>  
				-->
				<!--设置map输出 -->
				<property>
					<name>mapred.compress.map.output</name>
					<value>false</value>
				</property>
				<property>
					<name>mapred.map.output.compression.codec</name>
					<value>com.hadoop.compression.lzo.LzopCodec</value>
				</property>
				<!--设置reduce输出 -->
				<property>
					<name>mapred.output.compress</name>
					<value>false</value>
				</property>
				<property>
					<name>mapred.output.compression.codec</name>
					<value>com.hadoop.compression.lzo.LzopCodec</value>
				</property>	
			</configuration>

	10.4）、slave1-DataNode-TaskTracker主机的mapred-site.xml配置如下:
			#其他slave节点主机配置相同。
			<configuration>
				<property>
					<name>mapred.child.java.opts</name>
					<value>-Xmx1024m</value>
				</property>
				<property>
					<name>io.sort.mb</name>
					<value>512m</value>
				</property>
				<property>
					<name>mapred.tasktracker.map.tasks.maximum</name>
					<value>4</value>
					<final>true</final>
				</property>
				<property>
					<name>mapred.tasktracker.reduce.tasks.maximum</name>
					<value>2</value>
					<final>true</final>
				</property>
				<property>
					<name>mapred.job.tracker</name>
					<value>master-JobTracker:9001</value>
					<final>true</final>  
				</property>
				<!--
				<property>
					<name>mapred.task.timeout</name>
					<value>6000000</value>
				</property>
				-->
			</configuration>

			##########如果各节点使用相同配置的话，配置如下##################
			<configuration>
				<property>
					<name>mapred.job.tracker</name>
					<value>master-JobTracker:9001</value>#172.16.37.23
				</property>
			</configuration>
	
11、配置 masters，指定 SecondaryNameNode

	11.1）、master-NameNode 主机配置如下:
		cd /home/hadoop/hadoop-1.2.1/conf/
		vi masters :
			# 输入secondnamenode的地址（可以输入主机名或者输入ip）
			master-SecondaryNameNode

	11.2）、master-SecondaryNameNode 主机无需配置masters。
	
	11.3）、master-JobTracker 主机无需配置masters。
	
	11.4）、slave节点主机无需配置masters。

		#虽然名为 masters 但并不是配置 NameNode 或者 JobTracker 的，相反，它内部包含的是 SecondaryNameNode 的主机名。默认这里面是 localhost 也就是本机，这也是为什么伪分布模式，我们可以看到 SecondaryNameNode 运行在本机上的原因。

		#这样将来启动 HDFS 的时候，NameNode 就会通过 SSH 连接 master-SecondaryNameNode 以启动 SecondaryNameNode了。

		#如果不需要集群中存在 SecondaryNameNode，只需要将 conf/masters 中的内容清空即可。

		# conf/masters 文件中指定的机器，并不是说jobtracker或者namenode进程要 运行在这台机器上，因为这些进程是运行在 launch bin/start-dfs.sh或者 bin/start-mapred.sh(start-all.sh)的机器上的。所以，masters这个文件名是非常的令人混淆的，应该叫做secondaries会比较合适。

		#将所有想要运行secondarynamenode进程的机器写到masters文件中，一行一台。
	

12、配置 slaves ，指定数据节点 （只需在namenode、jobtracker、SecondaryNameNode节点上配置，无需同步到各slave节点）

	12.1）、master-NameNode 主机配置如下:
		vi slaves :
			#输入各datanode节点的主机名或ip地址  
			slave1-DataNode-TaskTracker
			slave2-DataNode-TaskTracker
			slave3-DataNode-TaskTracker
			slave4-DataNode-TaskTracker
			slave5-DataNode-TaskTracker

	12.2）、master-SecondaryNameNode 主机无需配置slaves。
	
	12.3）、master-JobTracker 主机配置slaves如下：
		vi slaves :
			#输入各datanode节点的主机名或ip地址  
			slave1-DataNode-TaskTracker
			slave2-DataNode-TaskTracker
			slave3-DataNode-TaskTracker
			slave4-DataNode-TaskTracker
			slave5-DataNode-TaskTracker
	
	12.4）、slave节点主机无需配置slaves。
	
13、需要在所有节点主机上建立数据存储目录
		mkdir /home/hadoop/hadoopData
		mkdir /home/hadoop/hadoopData/tmp
		chmod -R 755 /home/hadoop/hadoopData
	
14、关闭每一个节点的防火墙：
		chkconfig iptables off
		service iptables stop
	
15、格式化 NameNode
		格式化一个新的分布式文件系统
		第一次启动 Hadoop 集群前，我们需要先格式化 NameNode。
		首先将各个节点主机启动起来(不是启动hadoop)，然后在 master-NameNode（172.16.37.21） 上面格式化 NameNode：
	
	15.1)
		cd /home/hadoop/hadoop-1.2.1/
		bin/hadoop namenode -format
	
		结果信息如下：
			14/11/20 12:33:41 INFO namenode.NameNode: STARTUP_MSG: 
			/************************************************************
			STARTUP_MSG: Starting NameNode
			STARTUP_MSG:   host = master-NameNode/172.16.37.21
			STARTUP_MSG:   args = [-format]
			STARTUP_MSG:   version = 1.2.1
			STARTUP_MSG:   build = https://svn.apache.org/repos/asf/hadoop/common/branches/branch-1.2 -r 1503152; compiled by 'mattf' on Mon Jul 22 15:23:09 PDT 2013
			STARTUP_MSG:   java = 1.7.0_71
			************************************************************/
			14/11/20 12:33:41 INFO util.GSet: Computing capacity for map BlocksMap
			14/11/20 12:33:41 INFO util.GSet: VM type       = 64-bit
			14/11/20 12:33:41 INFO util.GSet: 2.0% max memory = 932184064
			14/11/20 12:33:41 INFO util.GSet: capacity      = 2^21 = 2097152 entries
			14/11/20 12:33:41 INFO util.GSet: recommended=2097152, actual=2097152
			14/11/20 12:33:42 INFO namenode.FSNamesystem: fsOwner=root
			14/11/20 12:33:42 INFO namenode.FSNamesystem: supergroup=supergroup
			14/11/20 12:33:42 INFO namenode.FSNamesystem: isPermissionEnabled=true
			14/11/20 12:33:42 INFO namenode.FSNamesystem: dfs.block.invalidate.limit=100
			14/11/20 12:33:42 INFO namenode.FSNamesystem: isAccessTokenEnabled=false accessKeyUpdateInterval=0 min(s), accessTokenLifetime=0 min(s)
			14/11/20 12:33:42 INFO namenode.FSEditLog: dfs.namenode.edits.toleration.length = 0
			14/11/20 12:33:42 INFO namenode.NameNode: Caching file names occuring more than 10 times 
			14/11/20 12:33:43 INFO common.Storage: Image file /home/hadoop/hadoopData/hdfs/name/current/fsimage of size 110 bytes saved in 0 seconds.
			14/11/20 12:33:43 INFO namenode.FSEditLog: closing edit log: position=4, editlog=/home/hadoop/hadoopData/hdfs/name/current/edits
			14/11/20 12:33:43 INFO namenode.FSEditLog: close success: truncate to 4, editlog=/home/hadoop/hadoopData/hdfs/name/current/edits
			14/11/20 12:33:43 INFO common.Storage: Storage directory /home/hadoop/hadoopData/hdfs/name has been successfully formatted.
			14/11/20 12:33:43 INFO namenode.NameNode: SHUTDOWN_MSG: 
			/************************************************************
			SHUTDOWN_MSG: Shutting down NameNode at master-NameNode/172.16.37.21
			************************************************************/

		看到 INFO common.Storage: Storage directory /home/hadoop/hadoopData/hdfs/name has been successfully formatted. 说明格式化成功了。
	
	15.2) 重新格式化 NameNode
		如果需要重新格式化（修改了集群等），则需要将重新格式化 NameNode，为了防止出现 namespaceID 或 storageID 冲突的问题，我们要清空（conf/core-site.xml文件中配置的 hadoop.tmp.dir项所在的文件夹）/home/hadoop/hadoopData/tmp 文件夹，以确保 Hadoop 集群将处于一个全新的环境。如下命令：
			rm -rf /home/hadoop/hadoopData/tmp/*
			rm -rf /home/hadoop/hadoopData/hdfs
	
16、启动集群
		启动Hadoop集群需要启动HDFS集群和Map/Reduce集群。
		面对集群环境时，就不建议使用 start-all.sh（单机伪分布式时用） 了，应该分步启动，这样更便于各节点间的协调以及排障。
    
	16.1)、首先，应该启动 HDFS
		在分配的NameNode（172.16.37.21）上，运行下面的命令启动HDFS:
			cd /home/hadoop/hadoop-1.2.1/
			bin/start-dfs.sh

		#bin/start-dfs.sh脚本会参照NameNode上${HADOOP_CONF_DIR}/slaves文件的内容，在所有列出的slave上启动DataNode守护进程。
	
		#我们可以通过在各个节点执行 jps，判断是否已经启动了该节点对应的服务。比如，在 master-NameNode 上验证 NameNode；在 slave1-DataNode-TaskTracker 上验证 DataNode等等。

		#此外，我们还应该去 http://172.16.37.21:50070 去检查 HDFS 是否运行正常。

		#比如， Live Node 是否是3个；如果进入了 Safe Mode，是否在30秒后，自动退出了 Safe Mode 等。出现了任何问题，到相应的节点上的 /home/hadoop/hadoop-1.2.1/logs/ 目录下寻找对应于节点的日志，根据里面的错误进行排障。

		#去 http://172.16.37.22:50090/  查看SecondaryNameNode是否运行正常。
    
	16.2)、在确认 HDFS 启动正常后，启动 MapReduce
		在分配的JobTracker（172.16.37.23）上，运行下面的命令启动Map/Reduce：
			bin/start-mapred.sh
	
		#bin/start-mapred.sh脚本会参照JobTracker上${HADOOP_CONF_DIR}/slaves文件的内容，在所有列出的slave上启动TaskTracker守护进程。
	
		#同样需要到各个节点去验证 jps 是否 JobTracker 和 TaskTracker 都启动正常，以及通过 http://172.16.37.23:50030 判断 JobTracker 和 TaskTracker 是否正常。
	
	
17、停止Hadoop  
		停止的集群的顺序应该和启动集群顺序相反。
	
	17.1)、在分配的JobTracker（172.16.37.23）上，运行下面的命令停止Map/Reduce：
		bin/stop-mapred.sh 

		#bin/stop-mapred.sh脚本会参照JobTracker上${HADOOP_CONF_DIR}/slaves文件的内容，在所有列出的slave上停止TaskTracker守护进程。
	
	17.2)、在分配的NameNode（172.16.37.21）上，执行下面的命令停止HDFS：
		bin/stop-dfs.sh

		#bin/stop-dfs.sh脚本会参照NameNode上${HADOOP_CONF_DIR}/slaves文件的内容，在所有列出的slave上停止DataNode守护进程。
```

### 3、安装Hive 、 Hbase  之接上面

```
1、安装zookeeper-3.4.6，安装Hbase之前先安装zookeeper。
	安装hbase必须要有zookeeper来管理。

	1.1)、下载：zookeeper-3.4.6.tar.gz

	1.2)、tar zxvf zookeeper-3.4.6.tar.gz

	1.3)、mv zookeeper-3.4.6 /home/hadoop/hadoop-1.2.1/

	1.4)、mkdir /home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/data  #创建一个文件夹用于存储zookeeper数据

	1.5)、mkdir /home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/log #创建一个文件夹用于存储zookeeperlog

	1.6)、cd /home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/conf

	1.7)、cp ./zoo_sample.cfg ./zoo.cfg

	1.8)、vi zoo.cfg
		修改：/home/hadoop/zookeeper-3.4.6/data
				dataDir=/home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/data
		添加：
				dataLogDir=/home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/log
		最后在文件最后加如下几行：
			server.21=master-NameNode:2888:3888
			server.22=master-SecondaryNameNode:2888:3888
			server.23=master-JobTracker:2888:3888
			server.24=slave1-DataNode-TaskTracker:2888:3888
			server.25=slave2-DataNode-TaskTracker:2888:3888
			server.26=slave3-DataNode-TaskTracker:2888:3888
			server.27=slave4-DataNode-TaskTracker:2888:3888
			server.10=slave5-DataNode-TaskTracker:2888:3888
		
			参数含义：
			#initLimit: zookeeper集群中的包含多台server, 其中一台为leader, 集群中其余的server为follower. initLimit参数配置初始化连接时, follower和leader之间的最长心跳时间. 此时该参数设置为5, 说明时间限制为5倍tickTime, 即5*2000=10000ms=10s.

			#syncLimit: 该参数配置leader和follower之间发送消息, 请求和应答的最大时间长度. 此时该参数设置为2, 说明时间限制为2倍tickTime, 即4000ms.
			
			#server.X=A:B:C 其中X是一个数字, 表示这是第几号server. A是该server所在的IP地址. B配置该server和集群中的leader交换消息所使用的端口. C配置选举leader时所使用的端口.
	 
	1.9)、把配置好的zookeeper文件夹scp到其他节点机器，其实就是每台机器上都有这个zookeeper。

			scp -r ./zookeeper-3.4.6 root@master-SecondaryNameNode:/home/hadoop/hadoop-1.2.1/
			scp -r ./zookeeper-3.4.6 root@master-JobTracker:/home/hadoop/hadoop-1.2.1/
			scp -r ./zookeeper-3.4.6 root@slave1-DataNode-TaskTracker:/home/hadoop/hadoop-1.2.1/
			scp -r ./zookeeper-3.4.6 root@slave2-DataNode-TaskTracker:/home/hadoop/hadoop-1.2.1/
			scp -r ./zookeeper-3.4.6 root@slave3-DataNode-TaskTracker:/home/hadoop/hadoop-1.2.1/
			scp -r ./zookeeper-3.4.6 root@slave4-DataNode-TaskTracker:/home/hadoop/hadoop-1.2.1/
			scp -r ./zookeeper-3.4.6 root@slave5-DataNode-TaskTracker:/home/hadoop/hadoop-1.2.1/
	 
	1.20)、 在之前设置的dataDir中新建myid文件, 写入一个数字, 该数字表示这是第几号server. 该数字必须和zoo.cfg文件中的server.X中的X一一对应。
	 
			在master-NameNode节点主机上新建myid文件，输入内容: 21
				vi /home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/data/myid
					21
			
			在master-SecondaryNameNode节点主机上新建myid文件，输入内容: 22
				vi /home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/data/myid
					22
			
			在master-JobTracker节点主机上新建myid文件，输入内容: 23
				vi /home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/data/myid
					23
			
			在slave1-DataNode-TaskTracker节点主机上新建myid文件，输入内容: 24
				vi /home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/data/myid
					24
				
			其他节点主机依次新建myid文件。
	 
	1.21)、设置环境变量，在各个节点都要设置
		vi /etc/profile
			添加：
				ZOOKEEPER_HOME=/home/hadoop/hadoop-1.2.1/zookeeper-3.4.6
			修改：
				PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_PREFIX/bin:$ZOOKEEPER_HOME/bin
			添加：
				export ZOOKEEPER_HOME
	
		source /etc/profile
	 
	1.22)、配置完成，在每个节点上可以运行：
		../zookeeper-3.4.6/bin/zkServer.sh start

		jps查看如果有QuorumPeerMain这个进程就是对了

		停止zookeeper：zkServer.sh stop
	 
	 
2、安装hbase-0.98.8

	2.1）下载hbase-0.98.8-hadoop1-bin.tar.gz

	2.2) tar zxvf hbase-0.98.8-hadoop1-bin.tar.gz
		mv ./hbase-0.98.8-hadoop1 /home/hadoop/hadoop-1.2.1/hbase-0.98.8

	2.3) 配置hbase-env.sh文件
			vi /home/hadoop/hadoop-1.2.1/hbase-0.98.8/conf/hbase-env.sh
				修改：
					export JAVA_HOME=/usr/java/jdk1.7.0_71
					
				把export HBASE_MANAGES_ZK=true 改为 export HBASE_MANAGES_ZK=false  (因为单独安装了zookeeper，就不使用hbase自带的了)
	      
	2.4) 配置hbase-site.xml文件
	 	  vi /home/hadoop/hadoop-1.2.1/hbase-0.98.8/conf/hbase-site.xml
				<configuration>
					<property>
						<name>hbase.rootdir</name> 
						<value>hdfs://master-NameNode:9000/hbase</value>
					</property>
					<property>
						<name>hbase.cluster.distributed</name>
						<value>true</value>
					</property>
					<property>
						<name>hbase.zookeeper.quorum</name>
						<value>master-NameNode,master-SecondaryNameNode,master-JobTracker,slave1-DataNode-TaskTracker,slave2-DataNode-TaskTracker,slave3-DataNode-TaskTracker,slave4-DataNode-TaskTracker,slave5-DataNode-TaskTracker</value>
					</property>
					<property>
						<name>dfs.replication</name>
						<value>3</value>
					</property>
					<property>
						<name>hbase.zookeeper.property.dataDir</name>
						<value>/home/hadoop/hadoop-1.2.1/zookeeper-3.4.6/data</value>
					</property>
				</configuration>
		
			#其中，hbase.zookeeper.quorum这项需要配置出所有的zookeeper节点，包括我们的HMaster这个角色所在的节点（我们这里其实就是master-NameNode这个节点），但是如果选用hbase自带的zookeeper就不需要列出master-NameNode这个节点了，如：<value>master-SecondaryNameNode,master-JobTracker,slave1-DataNode-TaskTracker,slave2-DataNode-TaskTracker,slave3-DataNode-TaskTracker,slave4-DataNode-TaskTracker,slave5-DataNode-TaskTracker</value>
		
	2.5) 配置regionservers文件
		vi /home/hadoop/hadoop-1.2.1/hbase-0.98.8/conf/regionservers 
			slave1-DataNode-TaskTracker
			slave2-DataNode-TaskTracker
			slave3-DataNode-TaskTracker
			slave4-DataNode-TaskTracker
			slave5-DataNode-TaskTracker
		
	2.6) 把配置好的hbase文件夹scp到其他节点机器，其实就是每台机器上都有这个hbase。
			scp -r ./hbase-0.98.8 root@master-SecondaryNameNode:/home/hadoop/hadoop-1.2.1/
			scp -r ./hbase-0.98.8 root@master-JobTracker:/home/hadoop/hadoop-1.2.1/
			scp -r ./hbase-0.98.8 root@slave1-DataNode-TaskTracker:/home/hadoop/hadoop-1.2.1/
			scp -r ./hbase-0.98.8 root@slave2-DataNode-TaskTracker:/home/hadoop/hadoop-1.2.1/
			scp -r ./hbase-0.98.8 root@slave3-DataNode-TaskTracker:/home/hadoop/hadoop-1.2.1/
			scp -r ./hbase-0.98.8 root@slave4-DataNode-TaskTracker:/home/hadoop/hadoop-1.2.1/
			scp -r ./hbase-0.98.8 root@slave5-DataNode-TaskTracker:/home/hadoop/hadoop-1.2.1/
	
	2.7) 配置完成我们就可以启动啦，因为独立的zookeeper，所以启动的时候必须确保所有节点的zookeeper正常启动:
	
			ssh到所有节点，zkServer.sh start。
			然后在主节点（master-NameNode）上：才能打开hbase，命令为：start-hbase.sh
				ssh root@master-NameNode
			/home/hadoop/hadoop-1.2.1/hbase-0.98.8/start-hbase.sh
		
			jps去观察各节点上的进程，master-NameNode上应该有HMaster，别的slave机器上应该有HRegionServer进程。
		
	2.8) 关闭的时候顺序应该是反的：hbase->zookeeper->hadoop。
	
	
3、安装apache-ant-1.9.4

	3.1) 下载apache-ant-1.9.4-bin.tar

	3.2) tar xvf apache-ant-1.9.4-bin.tar
		mv apache-ant-1.9.4 /home/hadoop/hadoop-1.2.1/
	
4、安装mysql
	mysql单独装在一台非hadoop集群里的机器上。本文没有那么多机器，使用172.17.37.27代替。

	4.1） 下载mysql-5.5.40.tar.gz

	4.2) rm -f /etc/my.cnf

	4.3) tar zxf mysql-5.5.40.tar.gz
		cd mysql-5.5.40

	4.4) patch -p1 < ../mysql-openssl.patch
		如果patch: command not found则安装patch, yum install patch
		mysql-openssl.patch 文件内容如下：
	     	--- mysql-5.5.31/vio/viossl.c~  2013-03-25 14:14:58.000000000 +0100
				+++ mysql-5.5.31/vio/viossl.c   2013-04-18 16:58:38.552557538 +0200
				@@ -172,8 +172,10 @@
					SSL_SESSION_set_timeout(SSL_get_session(ssl), timeout);
					SSL_set_fd(ssl, vio->sd);
				#ifndef HAVE_YASSL
				+#ifdef SSL_OP_NO_COMPRESSION
					SSL_set_options(ssl, SSL_OP_NO_COMPRESSION);
				#endif
				+#endif
		
				if ((r= connect_accept_func(ssl)) < 1)
				{   
	
	4.5) cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_READLINE=1 -DWITH_SSL=system -DWITH_ZLIB=system -DWITH_EMBEDDED_SERVER=1 -DENABLED_LOCAL_INFILE=1
			如果cmake : command not found，则安装：yum install cmake
	
			如果在cmake过程中出现如下错误：
				Could NOT find Curses (missing:  CURSES_LIBRARY CURSES_INCLUDE_PATH) 
				CMake Error at cmake/readline.cmake:83 (MESSAGE):
				Curses library not found.  Please install appropriate package,
      	remove CMakeCache.txt and rerun cmake.On Debian/Ubuntu, package name is libncurses5-dev, on Redhat and derivates it is ncurses-devel.
	
			则：yum install ncurses-devel
	   			rm CMakeCache.txt
			然后重新执行cmake语句。
	
	4.6) make && make install
	
	4.7) groupadd mysql
		useradd -s /sbin/nologin -M -g mysql mysql

	4.8) cp support-files/my-medium.cnf /etc/my.cnf
		sed '/skip-external-locking/i\datadir = /usr/local/mysql/var' -i /etc/my.cnf
		sed -i 's:#innodb:innodb:g' /etc/my.cnf
		sed -i 's:/usr/local/mysql/data:/usr/local/mysql/var:g' /etc/my.cnf
		 
	4.9) /usr/local/mysql/scripts/mysql_install_db --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql --datadir=/usr/local/mysql/var --user=mysql

	4.10) chown -R mysql /usr/local/mysql/var
		chgrp -R mysql /usr/local/mysql/.
		  
	4.11) cp support-files/mysql.server /etc/init.d/mysql
		chmod 755 /etc/init.d/mysql
    	  
	4.12) 
		cat > /etc/ld.so.conf.d/mysql.conf<<EOF
				/usr/local/mysql/lib
				/usr/local/lib
			EOF
		
		ldconfig

	4.13) 
		ln -s /usr/local/mysql/lib/mysql /usr/lib/mysql
		ln -s /usr/local/mysql/include/mysql /usr/include/mysql
		
		if [ -d "/proc/vz" ];then
		ulimit -s unlimited
		fi

	4.14) 启动mysql
		/etc/init.d/mysql start
	
	4.15) 
		ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
		ln -s /usr/local/mysql/bin/mysqldump /usr/bin/mysqldump
		ln -s /usr/local/mysql/bin/myisamchk /usr/bin/myisamchk
		ln -s /usr/local/mysql/bin/mysqld_safe /usr/bin/mysqld_safe
	
	4.16) 设置mysql root用户密码
	    /usr/local/mysql/bin/mysqladmin -u root password zxsoftdb123
	    
	    cat > /tmp/mysql_sec_script<<EOF
				use mysql;
				update user set password=password('zxsoftdb123') where user='root';
				delete from user where not (user='root') ;
				delete from user where user='root' and password=''; 
				drop database test;
				DROP USER ''@'%';
				flush privileges;
				EOF
		
			/usr/local/mysql/bin/mysql -u root -pzxsoftdb123 -h localhost < /tmp/mysql_sec_script
		
			rm -f /tmp/mysql_sec_script
		
			/etc/init.d/mysql restart

	4.14) 关闭 mysql
		/etc/init.d/mysql stop
	
	4.15）授权
		授权用户root使用密码zxsoftdb123从指定ip为172.16.37.21的主机连接到mysql服务器：
			mysql -uroot -pzxsoftdb123 登陆 mysql shell：
				GRANT ALL PRIVILEGES ON *.* TO 'root'@'172.16.37.21' IDENTIFIED BY 'zxsoftdb123' WITH GRANT OPTION;
	flush privileges;
	
	4.16) 设置mysql开机启动
		chkconfig --level 345 mysql on
	 	/etc/init.d/mysql start
	
	
5、安装Hive
	
	在单独的hive集群上安装或者在单独一台机上安装，这里测试时没有那多机器，安装在了172.16.37.21上：
	
	注：由于apache-hive-0.14.0与hadoop1.2.1 结合时会出现一些问题，这里的hive后又改用apache-hive-0.13.1-bin，安装步骤不变，还是和0.14一样。
	
	5.1) 下载apache-hive-0.14.0-bin.tar.gz

	5.2) tar zxf apache-hive-0.14.0-bin.tar.gz
		 mv apache-hive-0.14.0-bin /home/hadoop/hadoop-1.2.1/

	5.3) 修改配置
		 cd /home/hadoop/hadoop-1.2.1/apache-hive-0.14.0-bin/conf
		 	拷贝hive-default.xml.template，名字改为: hive-default.xml，这是hive的默认配置。

		 cp hive-default.xml.template hive-default.xml
		 	建立新文件：hive-site.xml, 这个里面的配置会覆盖hive-default.xml中的配置。

		 vi hive-site.xml
		 	调参在这个文件中改相应配置。
		 
	5.4) 拷贝hive-env.sh.template，名字改为：hive-env.sh。
		cp hive-env.sh.template ./hive-env.sh
		vi hive-env.sh

		修改内容如下：
			export HADOOP_HEAPSIZE=1024
			HADOOP_HOME=/home/hadoop/hadoop-1.2.1
			export HIVE_CONF_DIR=/home/hadoop/hadoop-1.2.1/apache-hive-0.14.0-bin/conf
			export HIVE_AUX_JARS_PATH=/home/hadoop/hadoop-1.2.1/apache-hive-0.14.0-bin/lib
		 
	5.5) cd /home/hadoop/hadoop-1.2.1/apache-hive-0.14.0-bin/conf
		cp hive-exec-log4j.properties.template hive-exec-log4j.properties
		cp hive-log4j.properties.template hive-log4j.properties
	     
	5.6) 把hive的元数据放在传统的RDBMS中，这里选择mysql。
		安装mysql后，修改my.cnf文件：注释掉bind-address=127.0.0.1这行，是用#号注释。
		修改hive-site.xml的配置如下：
		vi conf/hive-site.xml
		 	<?xml version="1.0" encoding="UTF-8" standalone="no"?>
			<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
			<configuration>
				<property>
					<name>hive.metastore.warehouse.dir</name>
					<value>hdfs://zxsoft-hadoop-master-NameNode:9000/user/hive/warehouse</value>#hdfs下的目录
					<description>location of default database for the warehouse</description>
				</property>
				<property>
					<name>hive.querylog.location</name>
					<value>/home/hadoop/hadoop-1.2.1/apache-hive-0.14.0-bin/log</value>
					<description>Location of Hive run time structured log file</description>
				</property>
				<property>
					<name>hive.exec.scratchdir</name>
					<value>hdfs://zxsoft-hadoop-master-NameNode:9000/tmp/hive-${user.name}</value>
					<description>Scratch space for Hive jobs</description>
				</property>
				<property>
					<name>javax.jdo.option.ConnectionURL</name>
					<value>jdbc:mysql://172.16.37.27:3306/hive?createDatabaseIfNotExist=true</value>
				</property>
				<property>
					<name>javax.jdo.option.ConnectionDriverName</name>
					<value>com.mysql.jdbc.Driver</value>
				</property>
				<property>
					<name>javax.jdo.option.ConnectionUserName</name>
					<value>root</value>
				</property>
				<property>
					<name>javax.jdo.option.ConnectionPassword</name>
					<value>zxsoftdb123</value>
				</property>
			</configuration>

	5.7） 在hdfs中创建几个目录
		$HADOOP_HOME/bin/hadoop fs -mkdir /tmp
		$HADOOP_HOME/bin/hadoop fs -mkdir /user/hive/warehouse
		$HADOOP_HOME/bin/hadoop fs -chmod g+w /tmp
		$HADOOP_HOME/bin/hadoop fs -chmod g+w /user/hive/warehouse
	      
	5.8)  在本地建立log目录  
		mkdir /home/hadoop/hadoop-1.2.1/apache-hive-0.14.0-bin/log
	      
	5.8) 下载mysql的驱动到hive的lib文件夹中。
		mv mysql-connector-java-5.1.33-bin.jar /home/hadoop/hadoop-1.2.1/apache-hive-0.14.0-bin/lib/
	
	5.10) 启动hive
		配置下环境变量:
		vi /etc/profile
			HIVE_HOME=/home/hadoop/hadoop-1.2.1/apache-hive-0.14.0-bin
			PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_PREFIX/bin:$ZOOKEEPER_HOME/bin:$HIVE_HOME/bin
			export HIVE_HOME
	 	 
		启动Hive:  
			执行hive命令：
				/home/hadoop/hadoop-1.2.1/apache-hive-0.14.0-bin/bin/hive
	 	 
		如果执行hive报如下错误：
			Exception in thread "main" java.lang.RuntimeException: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
			at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:444)
			at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:672)
			at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:616)
			at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
			at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
			at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
			at java.lang.reflect.Method.invoke(Method.java:606)
			at org.apache.hadoop.util.RunJar.main(RunJar.java:160)
			Caused by: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
			at org.apache.hadoop.hive.metastore.MetaStoreUtils.newInstance(MetaStoreUtils.java:1449)
			at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.<init>(RetryingMetaStoreClient.java:63)
			at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.getProxy(RetryingMetaStoreClient.java:73)
			at org.apache.hadoop.hive.ql.metadata.Hive.createMetaStoreClient(Hive.java:2661)
			at org.apache.hadoop.hive.ql.metadata.Hive.getMSC(Hive.java:2680)
			at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:425)
			... 7 more
			Caused by: java.lang.reflect.InvocationTargetException
			at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
			at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:57)
			at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
			at java.lang.reflect.Constructor.newInstance(Constructor.java:526)
			at org.apache.hadoop.hive.metastore.MetaStoreUtils.newInstance(MetaStoreUtils.java:1447)
			... 12 more
			Caused by: javax.jdo.JDOUserException: One or more instances could not be made persistent
			NestedThrowables:
			org.datanucleus.exceptions.NucleusDataStoreException: Error(s) were found while auto-creating/validating the datastore for classes. The errors are printed in the log, and are attached to this exception.
			at org.datanucleus.api.jdo.JDOPersistenceManager.makePersistentAll(JDOPersistenceManager.java:787)
			at org.apache.hadoop.hive.metastore.ObjectStore.grantPrivileges(ObjectStore.java:4071)
			at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
			at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
			at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
			at java.lang.reflect.Method.invoke(Method.java:606)
			at org.apache.hadoop.hive.metastore.RawStoreProxy.invoke(RawStoreProxy.java:98)
			at com.sun.proxy.$Proxy4.grantPrivileges(Unknown Source)
			at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.createDefaultRoles_core(HiveMetaStore.java:646)
			at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.createDefaultRoles(HiveMetaStore.java:615)
			at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.init(HiveMetaStore.java:430)
			at org.apache.hadoop.hive.metastore.RetryingHMSHandler.<init>(RetryingHMSHandler.java:66)
			at org.apache.hadoop.hive.metastore.RetryingHMSHandler.getProxy(RetryingHMSHandler.java:72)
			at org.apache.hadoop.hive.metastore.HiveMetaStore.newRetryingHMSHandler(HiveMetaStore.java:5554)
			at org.apache.hadoop.hive.metastore.HiveMetaStoreClient.<init>(HiveMetaStoreClient.java:178)
			at org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient.<init>(SessionHiveMetaStoreClient.java:73)
			... 17 more
			Caused by: org.datanucleus.exceptions.NucleusDataStoreException: Error(s) were found while auto-creating/validating the datastore for classes. The errors are printed in the log, and are attached to this exception.
			at org.datanucleus.store.rdbms.RDBMSStoreManager$ClassAdder.verifyErrors(RDBMSStoreManager.java:3602)
			at org.datanucleus.store.rdbms.RDBMSStoreManager$ClassAdder.addClassTablesAndValidate(RDBMSStoreManager.java:3205)
			at org.datanucleus.store.rdbms.RDBMSStoreManager$ClassAdder.run(RDBMSStoreManager.java:2841)
			at org.datanucleus.store.rdbms.AbstractSchemaTransaction.execute(AbstractSchemaTransaction.java:122)
			at org.datanucleus.store.rdbms.RDBMSStoreManager.addClasses(RDBMSStoreManager.java:1605)
			at org.datanucleus.store.AbstractStoreManager.addClass(AbstractStoreManager.java:954)
			at org.datanucleus.store.rdbms.RDBMSStoreManager.getDatastoreClass(RDBMSStoreManager.java:679)
			at org.datanucleus.store.rdbms.RDBMSStoreManager.getPropertiesForGenerator(RDBMSStoreManager.java:2045)
			at org.datanucleus.store.AbstractStoreManager.getStrategyValue(AbstractStoreManager.java:1365)
			at org.datanucleus.ExecutionContextImpl.newObjectId(ExecutionContextImpl.java:3827)
			at org.datanucleus.state.JDOStateManager.setIdentity(JDOStateManager.java:2571)
			at org.datanucleus.state.JDOStateManager.initialiseForPersistentNew(JDOStateManager.java:513)
			at org.datanucleus.state.ObjectProviderFactoryImpl.newForPersistentNew(ObjectProviderFactoryImpl.java:232)
			at org.datanucleus.ExecutionContextImpl.newObjectProviderForPersistentNew(ExecutionContextImpl.java:1414)
			at org.datanucleus.ExecutionContextImpl.persistObjectInternal(ExecutionContextImpl.java:2218)
			at org.datanucleus.ExecutionContextImpl.persistObjectWork(ExecutionContextImpl.java:2065)
			at org.datanucleus.ExecutionContextImpl.persistObjects(ExecutionContextImpl.java:2005)
			at org.datanucleus.ExecutionContextThreadedImpl.persistObjects(ExecutionContextThreadedImpl.java:231)
			at org.datanucleus.api.jdo.JDOPersistenceManager.makePersistentAll(JDOPersistenceManager.java:776)
			... 32 more
			Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Specified key was too long; max key length is 1000 bytes
			at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
			at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:57)
			at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
			at java.lang.reflect.Constructor.newInstance(Constructor.java:526)
			at com.mysql.jdbc.Util.handleNewInstance(Util.java:377)
			at com.mysql.jdbc.Util.getInstance(Util.java:360)
			at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:978)
			at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3887)
			at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3823)
			at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2435)
			at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2582)
			at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2526)
			at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2484)
			at com.mysql.jdbc.StatementImpl.execute(StatementImpl.java:848)
			at com.mysql.jdbc.StatementImpl.execute(StatementImpl.java:742)
			at com.jolbox.bonecp.StatementHandle.execute(StatementHandle.java:254)
			at org.datanucleus.store.rdbms.table.AbstractTable.executeDdlStatement(AbstractTable.java:760)
			at org.datanucleus.store.rdbms.table.TableImpl.createIndices(TableImpl.java:648)
			at org.datanucleus.store.rdbms.table.TableImpl.createConstraints(TableImpl.java:422)
			at org.datanucleus.store.rdbms.RDBMSStoreManager$ClassAdder.performTablesValidation(RDBMSStoreManager.java:3459)
			at org.datanucleus.store.rdbms.RDBMSStoreManager$ClassAdder.addClassTablesAndValidate(RDBMSStoreManager.java:3190)
			... 49 more
	 	 
		则说明mysql数据库存储字段有误，需要在数据库中执行：alter database hive character set latin1;
		命令如下:
			登陆mysql所在的服务器主机，进入mysql管理shell:
	 	 	mysql -uroot -pzxsoftdb123
	 	 	alter database hive character set latin1; #把hive数据库的字符集改为latin1。
	 	 
		再次执行 ./hive 命令，又报如下错误：
			Exception in thread "main" java.lang.RuntimeException: java.lang.RuntimeException: The root scratch dir: /tmp/hive-root on HDFS should be writable. Current permissions are: rwx--x--x
	 	 
		解决方法：
			hadoop fs -chmod +w /tmp/hive-root
				给/tmp/hive-root加上可写的权限。
	 	 
		hive> show tables;
			如果正常就说明配置成功。 
```

### 4、安装Jboss  之接上面

```
4.1） 下载 jboss-as-7.1.1.Final.tar.gz

4.2） tar zxf jboss-as-7.1.1.Final.tar.gz
	mv jboss-as-7.1.1.Final /home/

4.3） vi /etc/profile
	JBOSS_HOME=/home/jboss-as-7.1.1.Final

4.4） 修改standalone/configuration/standalone.xml配置文件
	找到：
		<interface name="public">
			<inet-address value="${jboss.bind.address:127.0.0.1}"/>
		</interface>
        
	把127.0.0.1 改成	所在机器的IP地址（172.16.37.27或者0.0.0.0），使用局域网能够访问，否则在局域网内不能通过172.16.37.27:8080打开网页。
        
4.5） 关闭防火墙或开放8080端口
	/sbin/iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
    	
4.6） 创建管理员用户
	cd /home/jboss-as-7.1.1.Final/bin
	./add-user.sh
	
	出现如下提示：
		What type of user do you wish to add? 
		a) Management User (mgmt-users.properties) 
		b) Application User (application-users.properties)
	
		(a): 
			直接回车
			又出现如下提示：
				Enter the details of the new user to add.
				Realm (ManagementRealm) : #直接回车
				Username : admin       #输入管理员用户名，默认admin
				Password : zxjboss #输入管理员密码
				Re-enter Password : zxjboss
    	
    管理员控制台访问：
			http://127.0.0.1:9990/console

    如果也想在其他局域网机器上访问管理员控制台，防火墙需要开放端口9990，修改standalone.xml
    	vi standalone/configuration/standalone.xml
				找到：
					<interface name="management">
						<inet-address value="${jboss.bind.address.management:127.0.0.1}"/>
					</interface>
				把127.0.0.1改为jboss所在机器（本机）的IP地址172.16.37.27。
		
4.7） 启动服务
	/home/jboss-as-7.1.1.Final/bin/standalone.sh
	
4.8) 设置虚拟目录

		4.8.1）修改${JBOSS_HOME}/standalone/configuration/standalone.xml文件
			找到：
				<subsystem xmlns="urn:jboss:domain:deployment-scanner:1.1"> 节点处：
					<subsystem xmlns="urn:jboss:domain:deployment-scanner:1.1">
						<deployment-scanner path="deployments" relative-to="jboss.server.base.dir" scan-interval="5000"/>
						<deployment-scanner name="demo" path="/home/web/deployments" scan-enabled="true" scan-interval="5000" auto-deploy-zipped="false" auto-deploy-exploded="false" deployment-timeout="60" />
					</subsystem>
        
      #<deployment-scanner name="demo" 部份就是新增的虚拟目录新位置。
				/home/web/deployments 必须是存在的目录，其有可写权限。
		
		4.8.2）在新的虚拟目录位置/home/web/deployments放置一个demo.war目录，和一个文件demo.war.deployed。
			其中demo.war里面放置你的应用，demo.war.deployed是一个空文件。
		
		4.8.3） 修改${JBOSS_HOME}/standalone/configuration/standalone.xml文件
			找到：
			<subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="false"> 节点处：
				内容如下：
					<subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="false">
						<connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"/>
						<virtual-server name="default-host" enable-welcome-root="true">
							<alias name="localhost"/>
							<alias name="example.com"/>
						</virtual-server>
						##增加
						<virtual-server name="/demo" enable-welcome-root="false" default-web-module="demo.war">
							<alias name="localhost"/>
						</virtual-server>
					</subsystem>
		
			就可以通过http://localhost:8080/demo/访问了。
	
		
4.9） JBoss AS 7 虚拟主机配置
	新建zxweb.war WEB应用程序：/home/jboss-as-7.1.1.Final/standalone/deployments/zxweb.war

	新建zxweb.war.deployed：vi /home/jboss-as-7.1.1.Final/standalone/deployments/zxweb.war.deployed,内容为空即可。

	4.9.1) 在应用程序（如zxweb）的WEB-INF下新建jboss-web.xml文件
		vi /home/jboss-as-7.1.1.Final/standalone/deployments/zxweb.war/WEB-INF/jboss-web.xml
		文件内容:
			<!DOCTYPE jboss-web PUBLIC "-//JBoss//DTD Web Application 5.0//EN" "http://www.jboss.org/j2ee/dtd/jboss-web_5_0.dtd">
			<jboss-web>
				<context-root>/</context-root>
				<!-- For session copy -->
				<replication-config>
					<cache-name>standard-session-cache</cache-name>
				</replication-config>
				<!-- For session time -->
				<max-active-sessions>20</max-active-sessions>
				<passivation-config>
					<use-session-passivation>true</use-session-passivation>
					<passivation-min-idle-time>60</passivation-min-idle-time>
					<passivation-max-idle-time>600</passivation-max-idle-time>
				</passivation-config>
			</jboss-web>
		
	4.9.2) 修改${JBOSS_HOME}/standalone/configuration/standalone.xml文件
		找到：<subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="false"> 节点处：
		内容如下：
			<subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="zxweb.com" native="false">
				<connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"/>
				<virtual-server name="default-host" enable-welcome-root="true">
					<alias name="localhost"/>
				</virtual-server>
				<virtual-server name="zxweb.com" enable-welcome-root="false" default-web-module="zxweb.war">
					<alias name="www.zxweb.com"/>
				</virtual-server>
			</subsystem>
        
		修改default-virtual-server的值为你新配置虚拟主机的name,由于jboss-web.xml配置了<context-root>/</context-root>，所以这里zxweb.com中应配置enable-welcome-root="false"，以使用zxweb.war项目的主页作为服务器欢迎页。
        
	4.9.3) 最后修改host文件，以standalone.sh启动JBoss服务器即可通过www.zxweb.com访问了。
		
4.10） https
	4.10.1) 
		/usr/java/jdk1.7.0_71/bin/keytool -genkey -alias zxsoft-hadoop-WebUI -keyalg RSA -keystore /home/zxsoft-hd.keystore -validity 36500
			
	4.10.2) 
		mv /home/standalone_xml_history /home/jboss-as-7.1.1.Final/standalone/configuration/
		
	4.10.3) 修改配置文件 将standlone.xml中
		<connector name="http" protocol="HTTP/1.1" socket-binding="http" scheme="http"/>
		注释掉 改为:
			<connector name="https" protocol="HTTP/1.1" scheme="https" socket-binding="https" secure="true">
				<ssl name="https" password="zxsoft" certificate-key-file="/home/jboss-as-7.1.1.Final/standalone/configuration/zxsoft-hd.keystore"/>
			</connector>
		
	4.10.4) 重新运行服务器
		# ../standalone.sh
		访问：https://xxxx:28443/
		
4.11） 设置开机启动
	4.11.1) vi /etc/init.d/jboss	
		内容如下：
			#!/bin/sh
			#chkconfig: 345 99 10
			#description: JBoss auto start-stop script.
			# Source function library.
			. /etc/rc.d/init.d/functions
			# Get config.
			. /etc/sysconfig/network
			# Check that networking is up.
			[ "${NETWORKING}" = "no" ] && exit 0
			### CHANGE THE STARTUP PATH TO YOUR START SCRIPT ###
			startup='/home/jboss-as-7.1.1.Final/bin/standalone.sh > /dev/null 2> /dev/null &'
			shutdown='killall java'
			start(){
				echo -n $"Starting JBoss service: "
				$startup
				RETVAL=$?
				echo
			}
			stop(){
				action $"Stopping JBoss service: " $shutdown
				RETVAL=$?
				echo
			}
			restart(){
				stop
				sleep 10
				start
			}
			# See how we were called.
			case "$1" in
				start)
				start
				;;
				stop)
				stop
				;;
				restart)
				restart
				;;
			*)
			echo $"Usage: $0 {start|stop|restart}"
			exit 1
			esac
			
	4.11.2) 赋予执行权限
		chmod +x  /etc/init.d/jboss
		
	4.11.3) 设置jboss随系统启动
		chkconfig --add jboss
		chkconfig jboss on
		chkconfig --level 345 jboss on
		
	4.11.4) 启动与停止
		service jboss start
		service jboss stop
```