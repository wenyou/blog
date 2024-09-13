---
title: CentOS通用系统需要安装的软件
author: Zeeny
comments: false
date: 2016-01-21
updated: 2016-01-21
tags: [linux,centos]
categories: [linux,centos]
summary: CentOS通用系统需要安装的软件
---


# CentOS通用系统需要安装的软件

```
yum install -y gcc
yum install -y gcc-c++ gcc-g77
yum install -y make cmake automake


yum install -y ntp    [test ntpdate -u time.nist.gov]
yum install -y openssl openssl-devel openssh-clients
yum install -y wget rsync
yum install -y zip unzip

yum install -y patch
yum install -y ncurses ncurses-devel
yum install -y ftp

yum install -y flex bison file libtool libtool-libs autoconf
yum install -y byacc libpcap libpcap-devel
yum install -y iftop
yum install -y glibc
yum install -y iptraf
yum install -y zlib zlib-devel

yum install -y kernel-devel
yum install -y libjpeg libjpeg-devel libpng libpng-devel libpng10 libpng10-devel gd gd-devel
yum install -y libxml2 libxml2-devel
yum install -y glib2 glib2-devel
yum install -y tar
yum install -y bzip2 bzip2-devel libevent libevent-devel
yum install -y curl curl-devel
yum install -y e2fsprogs e2fsprogs-devel
yum install -y krb5 krb5-devel
yum install -y libidn libidn-devel
yum install -y vim-minimal gettext gettext-devel
yum install -y gmp-devel pspell-devel libcap diffutils net-tools
yum install -y libc-client-devel psmisc libXpm-devel git-core c-ares-devel
yum -y install php-mcrypt libmcrypt libmcrypt-devel

	
	
安装 iptables
	yum install iptables-services
	
	
安装ifconfig
	yum provides ifconfig
	yum install net-tools
	
	
rar解压:
	tar -zxvf rarlinux-4.0.0.tar.gz
	cd rar
	make
	cp rar_static /usr/local/bin/rar
	rar x test.rar  解压test.rar
	
	
安装FTP服务:
	yum install -y vsftpd
	启动/重启/关闭vsftpd服务器 `/sbin/service vsftpd restart/start/stop`
	
	
免登陆：
	ssh-keygen -t rsa  【直接回车到最后，所有机器上执行】
	scp ~/.ssh/id_rsa.pub root@zxsoft-hadoop-slave-DataNode-TaskTracker-01:/root/.ssh/id_rsaNameNode.pub
	cat ~/.ssh/id_rsaNameNode.pub >> ~/.ssh/authorized_keys
	本机 SSH
	cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
	
	
关闭防火墙
	centos 7.1
		systemctl stop firewalld.service

		禁止开机启动
		systemctl disable firewalld.service
		
	centos 6.5
		chkconfig iptables off
		service iptables stop


ssh速度慢：
	vi /etc/ssh/sshd_config
		UseDNS no
		GSSAPIAuthentication no
		GSSAPIAuthentication yes
	
	systemctl restart sshd.service 或 /etc/init.d/sshd restart
```

## Hadoop节点的linux操作系统设置

### 1. ulimit  文件描述符的设置

``` 
查看：
ulimit -n
ulimit -u
ulimit -s
ulimit -a
	
修改：
vi /etc/security/limits.conf
	添加：
	root soft nofile 65536
	root hard nofile 65536
	* soft nofile 65536
	* hard nofile 65536
	
	root soft nproc 32768
	root hard nproc 32768
	* soft nproc 32768
	* hard nproc 32768


vi /etc/security/limits.d/90-nproc.conf
	#*         soft    nproc     1024
	*          soft    nproc     32768
	
	
vi /etc/pam.d/login最后添加禁止调试文件
	session required /lib/security/pam_limits.so
	
	或：
	Shell命令：
	echo ulimit -HSn 65536 >> /etc/rc.local
	echo ulimit -HSn 65536 >>/root/.bash_profile
	ulimit -HSn 65536
```

### 2. 内核参数的设置

```
vi /etc/sysctl.conf
	添加：
	kernel.panic_on_oops = 1
	kernel.panic = 1
	kernel.panic_on_oops = 1
	kernel.softlockup_panic = 1
	kernel.hung_task_panic = 1
   	
	vm.swappiness = 0
	net.core.netdev_max_backlog = 4096
	net.core.somaxconn = 1024
	net.core.rmem_max = 16777216
	net.core.wmem_max = 16777216
	net.ipv4.tcp_rmem = 4096 87380 16777216
	net.ipv4.tcp_wmem = 4096 65536 16777216
	net.ipv4.tcp_max_syn_backlog = 8192
	
	net.ipv6.conf.all.disable_ipv6 = 1
	net.ipv6.conf.default.disable_ipv6 = 1
	net.ipv6.conf.lo.disable_ipv6 = 1


使生效：
	`/sbin/sysctl -p`		
```



### 设备时间服务器

```
如果没有安装crontab，则安装：
	yum install vixie-cron
	yum install crontabs

	service crond status

#启动服务
	/sbin/service crond start 
	service crond start
	
#加入开机自动启动:
	chkconfig --level 35 crond on
	
	
crontab范例
	每五分钟执行   */5 * * * *

	每小时执行     0 * * * *

	每天执行       0 0 * * *

	每周执行       0 0 * * 0

	每月执行       0 0 1 * *

	每年执行       0 0 1 1 *
	
	

设置时间服务器
	A. rpm -qf /usr/sbin/ntpd  [确定ntp包已经安装]

	B. 设置与自身时间同步
	  vi /etc/ntp.conf
	  		server 127.127.1.0
				fudge 127.127.1.0 stratum 8 
	
	C. service ntpd start  [动服务器，如果ntpd已经安装，则可以直接启动]

	D. 设置ntpd自启动
		find  /etc/rc.d/ -name "*ntpd"
		/sbin/chkconfig --level 345 ntpd on
		!find
	   
	E. 检查防火墙
		iptables -L
			对比较严格的防火墙，应该对ntp端口123进行配置：
			iptables -I INPUT -p udp --dport 123 -j ACCEPT
			iptables -I OUTPUT -p udp --sport 123 -j ACCEPT
			service iptables save
			這樣就可以了，注意-I, 不是-A 
	 	
	F. 客户端配置
		客户端采用ntpdate来更新，配置在crontab中。根据需要决定频率。在每一台需要同步时间的设备上设置crontab
		crontab -e
			00 00 * * * /usr/sbin/ntpdate zxsoft-hadoop-master-NameNode [crontab 设置的是每天0点同步时间。]
	 		   
		为了保证时间服务器可用，将命令先在命令行下执行一下。
		ntpdate zxsoft-hadoop-master-NameNode
	
		在ntp server上重新启动ntp服务后，ntp server自身或者与其server的同步的需要一个时间段，这个过程可能是5分钟，在这个时间之内在客户端运行ntpdate命令时会产生no server suitable for synchronization found的错误。

		那么如何知道何时ntp server完成了和自身同步的过程呢？
			在ntp server上使用命令：watch ntpq -p
			
			注意LOCAL的这个就是与自身同步的ntp server。
			注意reach这个值，在启动ntp server服务后，这个值就从0开始不断增加，当增加到17的时候，从0到17是5次的变更，每一次是poll的值的秒数，是64秒*5=320秒的时间。
			
			如果之后从ntp客户端同步ntp server还失败的话，用ntpdate -d来查询详细错误信息，再做判断。
```


### 安装JDK1.7

```
rpm -ivh jdk-7u71-linux-x64.rpm

vi /etc/profile   
JAVA_HOME=/usr/java/jdk1.7.0_71
JRE_HOME=/usr/java/jdk1.7.0_71/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

### 安装hadoop1.2.1

```
mkdir /home/hadoop
mkdir -p /home/hadoopData/tmp
mkdir -p /home/hadoopData/hdfs/data
mkdir -p /home/hadoopData/hdfs/name
chmod -R 755 /home/hadoopData
	
tar zxvf hadoop-1.2.1-bin.tar.gz
mv ./hadoop-1.2.1 /home/hadoop/
	
增加环境变量配置
vi /etc/profile 
	HADOOP_PREFIX=/home/hadoop/hadoop-1.2.1
	YARN_CONF_DIR=/home/hadoop/hadoop-1.2.1
	PATH=$PATH:$HADOOP_PREFIX/bin
	export HADOOP_PREFIX
	export YARN_CONF_DIR
	export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_PREFIX/lib/native
	export HADOOP_OPTS="-Djava.library.path=$HADOOP_PREFIX/lib"
```

### zookeeper-3.4.6

```
tar zxvf zookeeper-3.4.6.tar.gz
mv zookeeper-3.4.6 /home/hadoop/
mkdir /home/hadoop/zookeeper-3.4.6/data  #创建一个文件夹用于存储zookeeper数据
mkdir /home/hadoop/zookeeper-3.4.6/log #创建一个文件夹用于存储zookeeperlog
	
vi /etc/profile
	添加：
	ZOOKEEPER_HOME=/home/hadoop/zookeeper-3.4.6

	修改：
	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_PREFIX/bin:$ZOOKEEPER_HOME/bin

	添加：
	export ZOOKEEPER_HOME
```


### 安装Hbase-0.98.8

```
tar zxvf hbase-0.98.8-hadoop1-bin.tar.gz
mv ./hbase-0.98.8-hadoop1 /home/hadoop/hbase-0.98.8

vi /etc/profile
添加
	HBASE_HOME=/home/hadoop/hbase-0.98.8
	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_PREFIX/bin:$ZOOKEEPER_HOME/bin:$HBASE_HOME/bin
	export HBASE_HOME
```

### 安装Solr

```
mkdir /home/hadoopData/solr
tar xzf solr-5.0.0.tgz solr-5.0.0/bin/install_solr_service.sh --strip-components=2

安装solr到指定位置并且自启动：
./install_solr_service.sh solr-5.0.0.tgz -i /home/hadoop -d /home/hadoopData/solr -u root

cd /home/hadoopData/solr/
vi solr.in.sh
	在配置文件最后添加
	SOLR_MODE=solrcloud
	
将所有的zookeeper服务器地址端口加入到ZK_HOST参数，包括自身。其中端口我们使用的不是zookeeper的默认端口2181，因为原本有供hbase使用的zookeeper存在，所以我们启动新的zookeeper实例使用2182端口供solr使用。

	ZK_HOST="zxsoft-hadoop-solr-01:2182,zxsoft-hadoop-solr-02:2182,zxsoft-hadoop-solr-03:2182,zxsoft-hadoop-solr-04:2182,zxsoft-hadoop-solr-05:2182"


设置自身的IP地址
	SOLR_HOST="zxsoft-hadoop-solr-01"
		    
	RMI_PORT=18983
	SOLR_JAVA_MEM="-Xms3072m -Xmx3072m"
	

配置zookeeper
	zookeeper存放数据的位置：/home/hadoopData/solr-zoo-data
	mkdir /home/hadoopData/solr-zoo-data
	mkdir /home/hadoopData/solr-zoo-data/log
	cd /home/hadoopData/solr-zoo-data
	vi myid
	   1  [myid文件内容为此服务器的对应标号]
```

### 安装oozie

```
cd /home/hadoop/
tar zxvf apache-maven-3.3.9-bin.tar.gz
vi /etc/profile
export MAVEN_HOME=/home/hadoop/apache-maven-3.3.9
export PATH=$PATH:$MAVEN_HOME/bin
	
cd /home/hadoop/
tar xvzf oozie-3.3.2.tar.gz
vi /etc/profile
	export OOZIE_HOME=/home/hadoop/oozie-3.3.2
	export PATH=$PATH:$OOZIE_HOME/bin
	
source /etc/profile
	

cd /home/hadoop/oozie-3.3.2
vi pom.xml
	requireJavaVersion  => 1.7
	hadoop.version  => 1.2.1
	hbase.version => 0.98.8
	
vi ./hadooplibs/hadoop-1/pom.xml    => 1.2.1
vi ./hadooplibs/hadoop-distcp-1/pom.xml   => 1.2.1
vi ./hadooplibs/hadoop-test-1/pom.xml  => 1.2.1
	
bin/mkdistro.sh -DskipTests -Dhadoop.version=1.2.1
	or:
mvn clean package assembly:single -Dhadoop.version=1.2.1 -DjavaVersion=1.7 -DtargetJavaVersion=1.7 -DskipTests
	
	
出现错误：
	[ERROR] Failed to execute goal on project oozie-sharelib-distcp: Could not resolve dependencies for project org.apache.oozie:oozie-sharelib-distcp:jar:3.3.2: Failed to collect dependencies at org.apache.oozie:oozie-hadoop-distcp:jar:1.2.1.oozie-3.3.2: Failed to read artifact descriptor for org.apache.oozie:oozie-hadoop-distcp:jar:1.2.1.oozie-3.3.2: Could not transfer artifact org.apache.oozie:oozie-hadoop-distcp:pom:1.2.1.oozie-3.3.2 from/to Codehaus repository (http://repository.codehaus.org/): repository.codehaus.org: 未知的名称或服务: Unknown host repository.codehaus.org: 未知的名称或服务 -> [Help 1]
	
	ERROR, Oozie distro creation failed
	
	
将ExtJS工具包拷贝到目录$OOZIE_HOME中:
	cp /home/hadoop/ext-2.2.zip $OOZIE_HOME
	
在上面的目录下创建libext目录，并将hadoop相关的jar库文件拷贝到libext下面，我使用的是Hadoop 1.2.1版本：
cd $OOZIE_HOME
mkdir libext
cp /home/hadoop/hadoop-1.2.1/hadoop-*.jar ./libext/
cp /home/hadoop/hadoop-1.2.1/lib/*.jar ./libext/
```

### 安装编译后的 解压运行版

```
unzip oozie-3.3.2-解压-运行版.zip
cd oozie-3.3.2/bin
./oozie-start.sh
```


### 安装mysql

```
mysql-5.5.40.tar.gz
mysql-openssl.patch
	
rm -f /etc/my.cnf
tar zxf mysql-5.5.40.tar.gz
cd mysql-5.5.40
patch -p1 < ../mysql-openssl.patch
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_READLINE=1 -DWITH_SSL=system -DWITH_ZLIB=system -DWITH_EMBEDDED_SERVER=1 -DENABLED_LOCAL_INFILE=1
	
	如果cmake出错，重新cmake的话，先rm -f CMakeCache.txt。
	
make && make install
groupadd mysql
useradd -s /sbin/nologin -M -g mysql mysql
cp support-files/my-medium.cnf /etc/my.cnf
sed '/skip-external-locking/i\datadir = /usr/local/mysql/var' -i /etc/my.cnf
sed -i 's:#innodb:innodb:g' /etc/my.cnf
sed -i 's:/usr/local/mysql/data:/usr/local/mysql/var:g' /etc/my.cnf
/usr/local/mysql/scripts/mysql_install_db --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql --datadir=/usr/local/mysql/var --user=mysql
chown -R mysql /usr/local/mysql/var
chgrp -R mysql /usr/local/mysql/.
cp support-files/mysql.server /etc/init.d/mysql
chmod 755 /etc/init.d/mysql
	
cat > /etc/ld.so.conf.d/mysql.conf<<EOF
	/usr/local/mysql/lib
	/usr/local/lib
	EOF
		
ldconfig
	
ln -s /usr/local/mysql/lib/mysql /usr/lib/mysql
ln -s /usr/local/mysql/include/mysql /usr/include/mysql
	
if [ -d "/proc/vz" ];then
	ulimit -s unlimited
fi
	
ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
ln -s /usr/local/mysql/bin/mysqldump /usr/bin/mysqldump
ln -s /usr/local/mysql/bin/myisamchk /usr/bin/myisamchk
ln -s /usr/local/mysql/bin/mysqld_safe /usr/bin/mysqld_safe
	
/etc/init.d/mysql start
	

设置mysql root用户密码
/usr/local/mysql/bin/mysqladmin -u root password zxsoftdb123
	
cat > /tmp/mysql_sec_script<<EOF
	use mysql;
	update user set password=password('zxsoftdb123') where user='root';
	delete from user where not (user='root');
	delete from user where user='root' and password=''; 
	drop database test;
	DROP USER ''@'%';
	flush privileges;
	EOF
		
/usr/local/mysql/bin/mysql -u root -pzxsoftdb123 -h localhost < /tmp/mysql_sec_script
rm -f /tmp/mysql_sec_script
	
/etc/init.d/mysql restart
		

关闭mysql
/etc/init.d/mysql stop
	

授权用户root使用密码zxsoftdb123从任意主机连接到mysql服务器：
mysql -uroot -pzxsoftdb123 登陆mysql shell
	
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'zxsoftdb123' WITH GRANT OPTION;
flush privileges;
	

设置mysql开机启动
# chkconfig --level 345 mysql on
# /etc/init.d/mysql start
		
		
如果安装mysql时编译了InnoDb，只要启用InnoDb支持就行了。
/etc/init.d/mysql stop
vi /etc/my.cnf
	注释掉#loose-skip-innodb
保存，重启mysql：
	/etc/init.d/mysql start

就行了，再次show plugins;查看:
		InnoDB  | ACTIVE
```

### 安装Node

```
tar -zxvf node-v0.12.6-linux-x64.tar.gz
mv node-v0.12.6-linux-x64 /usr/local/
cd /usr/local/node-v0.12.6-linux-x64/bin
./node -v
ln -s /usr/local/node-v0.12.6-linux-x64/bin/node /usr/local/bin/node
ln -s /usr/local/node-v0.12.6-linux-x64/bin/npm /usr/local/bin/npm


node环境装好后，安装forever模块， 命令： npm install forever -gd
```

### 安装Redis

```
tar xzf redis-3.0.2.tar.gz
cd redis-3.0.2
#安装
make     
make install
	
#安装结束后 启动服务 
src/redis-server  

#测试是否成功
src/redis-cli
redis> set foo bar
	OK                    
redis> get foo
	"bar"   
redis> del foo     
			
#设为开机启动：
vi /etc/rc.local
	nohup redis-server >/tmp/redis-server.log &
```

### 安装lnmp

```
tar zxvf lnmp1.2.tar.gz 
cd lnmp1.2
./install.sh  
	全部安装或手工选择性安装

#先查看时区
date -R   
	修改时区
	rm -rf /etc/localtime
	ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    

安装apache前先安装的软件：
tar zxvf apr-1.5.2.tar.gz
cd apr-1.5.2
./configure --prefix=/usr/local/apr
make && make install
    
tar zxvf apr-util-1.5.4.tar.gz 
cd apr-util-1.5.4
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
make && make install
    
tar jxvf pcre-8.36.tar.bz2
cd pcre-8.36
./configure
make && make install
    
    
安装apache：
tar zxvf httpd-2.4.17.tar.gz
cd httpd-2.4.17
./configure --prefix=/usr/local/apache2 --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util --with-pcre=/usr/local/pcre --enable-so --enable-rewrite
make && make install
	
cp /usr/local/apache2/bin/apachectl /etc/rc.d/init.d/httpd

测试是否安装成功：
service httpd start
	出现“It works”就说明Apache已经正常安装
	
	如果64位linux（service httpd start）出现这种错误：
		/usr/local/apache2/bin/httpd: error while loading shared libraries: libpcre.so.1: cannot open shared object file: No such file or directory
	则：
		cd /lib64/
		ln -s libpcre.so.0.0.1 libpcre.so.1
		然后再启动（service httpd start）即可。
	
	如果出现如下错误：
		httpd: Could not reliably determine the server's fully qualified domain name, using localhost.localdomain. Set the 'ServerName' directive globally to suppress this message
	则：
		vi /usr/local/apache2/conf/httpd.conf
		添加：ServerName  localhost:80 即可。
	
	
安装php:
tar zxvf libmcrypt-2.5.8.tar.gz 
cd libmcrypt-2.5.8
./configure --prefix=/usr/local
make && make install
	
tar zxvf php-5.6.15.tar.gz
cd php-5.6.15
./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2/bin/apxs --with-libxml-dir=/usr/include/libxml2 --with-config-file-path=/usr/local/apache2/conf --with-mysql=/usr/local/mysql --with-mysqli=/usr/local/mysql/bin/mysql_config --with-gd --enable-gd-native-ttf --with-zlib --with-mcrypt --with-pdo-mysql=/usr/local/mysql --enable-shmop --enable-soap --enable-sockets --enable-wddx --enable-zip --with-xmlrpc --enable-fpm --enable-mbstring --with-zlib-dir --with-bz2 --with-curl --enable-exif --enable-ftp --with-jpeg-dir=/usr/lib --with-png-dir=/usr/lib --with-freetype-dir=/usr/lib/
make && make install
	
	
修改apache配置文件httpd.conf以支持PHP:
vi /usr/local/apache2/conf/httpd.conf
	添加php支持
		AddType application/x-httpd-php .php .phtml
		AddType application/x-httpd-php-source .phps
	添加默认索引页面index.php，再找到“DirectoryIndex”，在index.html后面加上“ index.php”
	
	不显示目录结构，找到“Options Indexes FollowSymLinks”，修改为 Options FollowSymLinks
	开启Apache支持伪静态，找到“AllowOverride None”，修改为AllowOverride All
	
	
保存httpd.conf配置，然后再执行以下两行命令:
chown -R nobody. /usr/local/apache2/htdocs/
chmod -R 777 /usr/local/apache2/htdocs/
service httpd restart
写个php页面测试成功。
```


### 配置Tomcat

```
修改server.xml：
	添加：
		<Service name="Catalina2">
			<Connector URIEncoding="UTF-8" connectionTimeout="20000" port="80" protocol="HTTP/1.1" redirectPort="8444"/>
			<Connector port="8089" protocol="AJP/1.3" redirectPort="8444"/>
			<Engine defaultHost="localhost" name="Catalina2">
				<Host appBase="" autoDeploy="true" name="localhost" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
					<Context docBase="/usr/local/apache-tomcat-8.0.26/webapps/VIBAP-WEB" path="" reloadable="true" workDir="/usr/local/apache-tomcat-8.0.26/webapps/VIBAP-WEB"/>
					</Host>
			</Engine>
		</Service>
```

### 配置FTPS服务

```
vi /etc/vsftpd/vsftpd.conf
禁止匿名用户登录:
	anonymous_enable=YES 改为 anonymous_enable=NO
	
不可以让ftp用户跳出自己的home目录:
	#chroot_local_user=YES 改为 chroot_local_user=YES 把#号去掉
	
	
mkdir /var/ftp/pub/fycity
/usr/sbin/adduser -d /var/ftp/pub/fycity -g ftp -s /sbin/nologin fycity
passwd fycity
chmod o+w /var/ftp/pub/fycity
```

### 配置转发

```
转发前配置：
	
linux上开启IP forward: echo "1" > /proc/sys/net/ipv4/ip_forward 
	注意：当机器网络重新启动，这个数字会恢复default＝0。要修改的话，可以修改/etc/sysctl.conf文件，永久生效。	
	
配置转发（centos7.1的DNAT）
	firewall-cmd --zone=public --add-forward-port=port=50070:proto=tcp:toaddr=172.16.222.55 --permanent
	firewall-cmd --zone=public --add-forward-port=port=50030:proto=tcp:toaddr=172.16.222.55 --permanent
	firewall-cmd --zone=public --add-forward-port=port=60010:proto=tcp:toaddr=172.16.222.58 --permanent
	firewall-cmd --zone=public --add-forward-port=port=8983:proto=tcp:toaddr=172.16.222.58 --permanent
	firewall-cmd --zone=public --add-forward-port=port=50075:proto=tcp:toaddr=172.16.222.62 --permanent
	firewall-cmd --reload #重启防火墙
	
开放端口：
	firewall-cmd --zone=public --add-port=50070/tcp --permanent
	firewall-cmd --zone=public --add-port=50030/tcp --permanent
	firewall-cmd --zone=public --add-port=60010/tcp --permanent
	firewall-cmd --zone=public --add-port=8983/tcp --permanent
	firewall-cmd --zone=public --add-port=50075/tcp --permanent
	firewall-cmd --zone=public --add-port=3306/tcp --permanent
	firewall-cmd --zone=public --add-port=11000/tcp --permanent

#重启防火墙
	firewall-cmd --reload 
```

```
centos6.5 配置转发 （centos6.5的DNAT）
	
iptables -t nat -A PREROUTING  -i eth0 -d 15.124.225.101  -p tcp --dport 50030 -j DNAT --to-destination 172.16.37.30
service iptables save
	
/sbin/iptables -I INPUT -p tcp --dport 50030 -j ACCEPT
```

### 初始化、并启动Hadoop

```
1、格式化 NameNode
	格式化一个新的分布式文件系统
	第一次启动 Hadoop 集群前，我们需要先格式化 NameNode。
	首先将各个节点主机启动起来(不是启动hadoop)，然后在 master-NameNode上面格式化 NameNode：
		cd /home/hadoop/hadoop-1.2.1/bin/
		./hadoop namenode -format
	
	
2、启动集群
	启动Hadoop集群需要启动HDFS集群和Map/Reduce集群。
	首先，应该启动 HDFS：
	在分配的NameNode上，运行下面的命令启动HDFS:
		cd /home/hadoop/hadoop-1.2.1/bin/
		./start-dfs.shkkkk jm
	
	在确认 HDFS 启动正常后，启动 MapReduce，在分配的JobTracker上，运行下面的命令启动Map/Reduce：
    ./start-mapred.sh
```