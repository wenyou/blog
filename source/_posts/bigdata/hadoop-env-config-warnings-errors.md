---
title: Hadoop环境配置之解决警告和错误
author: Zeeny
comments: false
date: 2014-11-24
updated: 2014-11-24
tags: [Hadoop,大数据]
categories: [大数据]
summary: Hadoop环境配置之解决警告和错误
---


# Hadoop环境配置之解决警告和错误#

## 一、一些警告信息

* 1、在运行hadoop的时候，出现警告: WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable

```
通过命令查看libhadoop.so.1.0.0是否是64位的，如果不是64位的，是32位的，则使用下面的重新编译Hadoop源码之方法来解决。
    
执行：
		file /home/wenyou/hadoop-2.5.1/lib/native/*

结果：
	/home/wenyou/hadoop-2.5.1/lib/native/libhadoop.so.1.0.0: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, not stripped
    
如果是64位的，还是出现上页的警告，说明本地库不存在，或者本地库与当前操作系统的版本不一致。
则可以通过以下方法来解决：
    
一、本地库与当前操作系统的版本不一致 
	出现 “WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable”警告信息 的解决办法：
    
	增加调试信息设置
	执行：export HADOOP_ROOT_LOGGER=DEBUG,console
     
	执行：hadoop fs -text /tmp/access.log.gz
	出现如下信息：
		DEBUG util.NativeCodeLoader: Failed to load native-hadoop with error: java.lang.UnsatisfiedLinkError: /home/wenyou/hadoop-2.5.1/lib/native/libhadoop.so.1.0.0: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by /home/wenyou/hadoop-2.5.1/lib/native/libhadoop.so.1.0.0)
    
	说明系统中的glibc的版本和libhadoop.so需要的版本不一致导致
    
	查看系统的libc版本：
	执行：
		ll /lib64/libc.so.6
    lrwxrwxrwx. 1 root root 12 6月  11 23:01 /lib64/libc.so.6 -> libc-2.12.so
	系统中的版本为2.12, 而libhadoop.so.1.0.0要求2.14。
    

	所以下载glibc-2.14
    wget http://ftp.gnu.org/gnu/glibc/glibc-2.14.tar.bz2
    wget http://ftp.gnu.org/gnu/glibc/glibc-linuxthreads-2.5.tar.bz2
    tar -jxvf glibc-2.14.tar.bz2
    cd glibc-2.14
    tar -jxvf ../glibc-linuxthreads-2.5.tar.bz2
    cd ../
    export CFLAGS="-g -O2"
    ./glibc-2.14/configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
    make
    make install
    
	安装编译过程中需要注意三点：
		(1)、要将glibc-linuxthreads解压到glibc目录下。
		(2)、不能在glibc当前目录下运行configure。
		(3)、加上优化开关，export CFLAGS="-g -O2"，否则会出现错误。
	
	安装完后，可以查看：ls -l /lib64/libc.so.6 已升级
	
	测试本地库是否升级
	执行：
		export HADOOP_ROOT_LOGGER=DEBUG,console

  执行：
		hadoop fs -text /tmp/access.log.gz
    
	可以看到将glibc升级后不再报错，已经成功加载本地库。
    

二、32位的本地库，需要通过重新编译Hadoop源码的方法来解决：
	
	原因：
		Apache提供的hadoop本地库是32位的，而在64位的服务器上就会有问题，因此需要自己编译64位的版本。

		有个WARN提示hadoop不能加载本地库，网上搜了下，这个问题基本上是由于在apache hadoop官网上下载的hadoopXXX.bin.tar.gz实在32位的机器上编译的(蛋疼吧)，我集群使用的64bit的，加载.so文件时出错，当然基本上不影响使用hadoop(如果你使用mahout做一些机器学习的任务时有可能会遇到麻烦，加载不成功，任务直接退出，所以还是有必要解决掉这个warn的)。
	
	解决方法：
		在自己的服务器上下载Hadoop源码hadoop-2.5.1-src.tar.gz，重新编译。
		只需要在master上重新编译就行了，把新的本地库替换原来32位的本地库就行了。
		编译步骤：
			1) yum -y install svn ncurses-devel gcc*                                             
			2) yum -y install lzo-devel zlib-devel autoconf automake libtool cmake openssl –devel
	
			3) 安装 maven
				wget http://www.eng.lsu.edu/mirrors/apache/maven/maven-3/3.2.3/binaries/apache-maven-3.2.3-bin.tar.gz
				tar -zxvf apache-maven-3.2.3-bin.tar.gz
				mv apache-maven-3.2.3 /home/wenyou/

				配置环境变量，编辑/etc/profile或者~/.bashrc文件
				vi /etc/profile
					增加：
						MAVEN_HOME=/home/wenyou/apache-maven-3.2.3
						PATH=$PATH:$MAVEN_HOME/bin
						export MAVEN_HOME
		
				执行：
					source /etc/profile
				通过mvn --version来检查是否安装正常
	
			4) 安装 protobuf(https://github.com/google/protobuf)
				wget https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz
				tar -zxvf protobuf-2.5.0.tar.gz
				cd protobuf-2.5.0
				./configure
				make
				make check
				make install

    		可以通过protoc --version来查看是否安装正常
    
    5）编译安装 hadoop-2.5.1
    	cd hadoop-2.5.1-src
    	mvn clean package –P dist,native –D skipTests –D tar
```


```
解决方法一：
	
原因：
	Apache提供的hadoop本地库是32位的，而在64位的服务器上就会有问题，因此需要自己编译64位的版本。http://blog.sina.com.cn/s/blog_6d932f2a0101pgk3.html  
	
1.安装JDK和GCC
	
2.安装maven
	wget http://www.eng.lsu.edu/mirrors/apache/maven/maven-3/3.2.3/binaries/apache-maven-3.2.3-bin.tar.gz
	tar -zxvf apache-maven-3.2.3-bin.tar.gz
	mv apache-maven-3.2.3 /home/wenyou/
	
	配置环境变量，编辑/etc/profile或者~/.bashrc文件
		vi /etc/profile
		增加：
			MAVEN_HOME=/home/wenyou/apache-maven-3.2.3
			PATH=$PATH:$MAVEN_HOME/bin
			export MAVEN_HOME
	
			执行：source /etc/profile
			通过 mvn --version 来检查是否安装正常
	
3.安装protobuf (https://github.com/google/protobuf)
	wget https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz
	tar -zxvf protobuf-2.5.0.tar.gz
	cd protobuf-2.5.0
	./configure
	make
	make check
	make install
	
	可以通过protoc --version来查看是否安装正常


4.查看cmake是否已经安装（cmake --version），如果没有安装则：
	wget http://www.cmake.org/files/v2.8/cmake-2.8.12.2.tar.gz
	tar -zxvf cmake-2.8.12.2.tar.gz 
	cd cmake-2.8.12.2
	./bootstrap
	make
	make install
	
		可以通过cmake --version来查看是否安装正常
	
5.安装autotool
	yum install autoconf automake libtool
	
6.查看openssl-devel是否已经安装，如果没有则安装openssl-devel:
	yum install openssl-devel
	
7.编译hadoop源码：
	下载源码包hadoop-2.5.1-src.tar.gz并解压
	进入hadoop-2.5.1-src目录
	执行：mvn clean package -Pdist,native -DskipTests -Dtar

	编译好的hadoop-2.5.1.tar.gz在hadoop-2.5.1-src木目录下的hadoop-dist/target/的目录中，接下来就可以安装了。
```


```
解决方法二：

	在Ubuntu上安装完hadoop2.4以后，使用以下命令：

	hadoop fs -ls
		14/09/09 11:33:51 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
		Found 1 items
		drwxr-xr-x   - duomai supergroup          0 2014-09-05 12:10 flume
		
		有个WARN提示hadoop不能加载本地库，网上搜了下，这个问题基本上是由于在apache hadoop官网上下载的hadoopXXX.bin.tar.gz实在32位的机器上编译的(蛋疼吧)，我集群使用的64bit的，加载.so文件时出错，当然基本上不影响使用hadoop(如果你使用mahout做一些机器学习的任务时有可能会遇到麻烦，加载不成功，任务直接退出，所以还是有必要解决掉这个warn的)。

		但是每次运行一个命令多有这么个WARN很不爽，想干掉也很简单：
			1. 下载hadoop2.4源码
			2. 在集群的某台机器上编译
			3. 替换之前的$HADOOP_HOME/lib/native为新编译的native
				其中有几点注意的：

				（1）在2(进行编译)之前，先修改maven安装目录下conf/settings.xml(因为maven使用的国外的reposity，国内有时无法访问，修改为国内镜像即可)，<mirrors></mirros>里添加，其他的不需改动,具体修改如下：
					<mirror>
						<id>nexus-osc</id>
						<mirrorOf>*</mirrorOf>
						<name>Nexusosc</name>
						<url>http://maven.oschina.net/content/groups/public/</url>
					</mirror>
				同样，在<profiles></profiles>内新添加：
						<profile>
							<id>jdk-1.7</id>
							<activation>
								<jdk>1.7</jdk>
							</activation>
							<repositories>
								<repository>
									<id>nexus</id>
									<name>local private nexus</name>
									<url>http://maven.oschina.net/content/groups/public/</url>
									<releases>
										<enabled>true</enabled>
									</releases>
									<snapshots>
										<enabled>false</enabled>
									</snapshots>
								</repository>
							</repositories>
							<pluginRepositories>
								<pluginRepository>
									<id>nexus</id>
									<name>local private nexus</name>
									<url>http://maven.oschina.net/content/groups/public/</url>
									<releases>
										<enabled>true</enabled>
									</releases>
									<snapshots>
										<enabled>false</enabled>
									</snapshots>
								</pluginRepository>
							</pluginRepositories>
						</profile>

				修改完成后使用以下命令编译hadoop：
					mvn package -Dmaven.javadoc.skip=true -Pdist,native -DskipTests -Dtar
				然后就是等待，大概20min后，build success，目标在
					hadoop-2.4.1-src/hadoop-dist/target/hadoop-2.4.1.tar.gz

			（2）在编译成功后，将新的lib/native替换到集群中原来的lib/native，记得要修改$HADOOP_HOME/etc/hadoop/hadoop-env.sh，在最后加上：
				export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib:$HADOOP_HOME/lib/native"
	======

重新运行如下命令：
	hadoop fs -ls
	Found 1 items
	drwxr-xr-x   - duomai supergroup          0 2014-09-05 12:10 flume
	WARN消失，good！
```