---
title: Centos安装FTP服务
author: Zeeny
comments: false
date: 2015-03-24
updated: 2015-03-24
tags: [FTP,linux,centos]
categories: [linux,centos]
summary: Centos安装FTP服务
---


# Centos安装FTP服务

## 1.安装FTP服务

1. 首先判断你服务器上是否安装了vsftpd: `rpm -q vsftpd`；安装vsftpd：`yum install vsftpd`

2. 启动/重启/关闭vsftpd服务器 `/sbin/service vsftpd restart/start/stop`

3. 与vsftpd服务器有关的文件和文件夹 `/etc/vsftpd/vsftpd.conf`

4. 添加FTP本地用户 `/usr/sbin/adduser -d /var/ftp/pub/BusLogData -g ftp -s /sbin/nologin hfbususer` , hfbususer这个用户只能连接ftp无法登录系统，默认home目录是在/var/ftp/pub/BusLogData文件夹下面。`passwd hfbususer`给hfbususer用户设置密码。

5. 然后给home目录修改权限，否则无法上传文件:chmod o+w /var/ftp/pub/BusLogData

6.  修改selinux：`vi /etc/selinux/config`，SELINUX=enforcing，默认是enforcing  把他修改为disabled。

7.  `setenforce 0`,因为修改selinux后需要重启服务，因为服务器不可以重启所以执行上面这个命令，临时修改selinux的策略，无需重启！ 生产环境不能改selinux，可以这样做`setsebool -P ftp_home_dir on`

```	
如果你连接到在CentOS上运行的使用chroot改变根目录的FTP服务器时遇到了下面这个错误，禁用SELinux是一个办法。

500 OOPS: cannot change directory:/home/dev
Login failed.

虽然关闭SELinux是一个快速的解决办法，但在生产环境下这么做可能不安全。所以，改而打开SELinux中的下列布尔表达式可以解决这个问题。

$ sudo setsebool -P ftp_home_dir on	
```

8. 配置vsftpd.conf

```
vi /etc/vsftpd/vsftpd.conf
	
禁止匿名用户登录:
anonymous_enable=YES 改为 anonymous_enable=NO

不可以让ftp用户跳出自己的home目录:
#chroot_local_user=YES 改为 chroot_local_user=YES 把#号去掉
```


9. 重启vsftpd服务，并且下次自动启动：

```
/sbin/service vsftpd restart
chkconfig vsftpd on
```


10. 关闭防火墙： 

```
service iptables stop
	或者
vi /etc/sysconfig/iptables

	在-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT这一行后追加：
		-A INPUT -m state --state NEW -m tcp -p tcp --dport 20 -j ACCEPT
		-A INPUT -m state --state NEW -m tcp -p tcp --dport 21 -j ACCEPT
		-A INPUT -m state --state NEW -m tcp -p tcp --dport 64000:65535 -j ACCEPT
	保存，重启：service iptables restart


centos7.1:
	firewall-cmd --state
	systemctl status firewalld.service
	systemctl stop firewalld.service
		
添加开放21端口：
	firewall-cmd --query-port=21/tcp 查看端口是否开放
	firewall-cmd --zone=public --add-port=21/tcp --permanent
		[--zone #作用域  --permanent #永久生效，没有此参数重启后失效]
		
添加开放被动端口范围：
	firewall-cmd --zone=public --add-port=64000-65535/tcp --permanent

重启防火墙：
	firewall-cmd --reload 	
```


11. FTP不能上传文件解决方案：

```
getsebool -a|grep ftp

setsebool -P ftp_home_dir  on
setsebool -P ftpd_full_access on

/sbin/service vsftpd restart  
centos7.1: systemctl restart  vsftpd.service
```


12. FTPS配置

```
openssl req -x509 -days 3650 -newkey rsa:2048 -nodes -keyout /etc/vsftpd/vsftpd.pem -out /etc/vsftpd/vsftpd.pem
	
vi /etc/vsftpd/vsftpd.conf
	
追加：
# 启用TLS/SSL
	ssl_enable=YES
# 强迫客户机在登录时使用TLS
	allow_anon_ssl=NO
	force_local_data_ssl=YES
	force_local_logins_ssl=YES
	ssl_tlsv1=YES
	ssl_sslv2=NO
	ssl_sslv3=NO
	require_ssl_reuse=NO
	ssl_ciphers=HIGH
	
# 指定SSL证书/私钥
	rsa_cert_file=/etc/vsftpd/vsftpd.pem
	rsa_private_key_file=/etc/vsftpd/vsftpd.pem
	
# 为被动模式下的连接定义端口范围
	pasv_max_port=65535
	pasv_min_port=64000
```
	
13. 启用日志功能

```
vi /etc/vsftpd/vsftpd.conf
	修改：
	xferlog_enable=YES
	xferlog_std_format=NO
	xferlog_file=/var/log/vsftpd.log
	log_ftp_protocol=YES
	debug_ssl=YES
```


14. ftps 上传命令：

```
​curl -vvv -T test.txt -k --ftp-ssl ftp://username:usrpwd@ip:21/

​curl -T ./test.log -k --ftp-ssl [ftp://username:usrpwd@ip:21/](ftp://username:usrpwd@ip:21/)
```


## 2. 安装ftp客户端

* `yum install ftp`
* ftp命令进入ftp命令行窗口
* ftp>提示符下，运行 open 192.168.37.1 命令，连接FTP服务器。非21端口时用：open 192.168.37.1 2821 
* 上传文件：put test.log
* ftp> cd images ， 进入images目录。
* ftp> lcd /root  ，改变本地工作目录。
* ftp> mget *.jpg ，下载所有.jpg文件到本地。
* 常用的ftp子命令总结如下：
* open: 与服务器相连接。
* send(put):上传文件。
* get： 下载文件。
* mget：下载多个文件。
* cd： 切换目录。
* lcd: 切换本地目录。
* dir：查看当前目录下的文件。
* del：删除文件。
```
ftps 上传命令：
curl -vvv -T test.txt -k --ftp-ssl ftp://zxwawmgr:zxwamgr20150606@hfsc.zxxxxx.com:21/
		
curl -T ./test.log -k --ftp-ssl ftp://zxwawmgr:jzxwamgr20150606@hfsc.zxxxxxx.com:21/
```
* bye：中断与服务器的连接。