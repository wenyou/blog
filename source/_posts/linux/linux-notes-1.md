---
title: Linux笔记
author: Zeeny
comments: false
date: 2014-04-15
updated: 2014-04-15
tags: [linux,centos]
categories: [linux]
summary: Linux笔记
---


# Linux笔记

## 常用命令

- 在某个目录下查找包含某个字符串的文件：$grep -r "stk-logo" ./

- sar -n DEV 1 100  查看网速 网络带宽负载。


- 批量复制并重命名:

```
for F in *.txt ; do cp $F ${F%.txt}_1.txt;done 
	其中${F%.txt}_1.txt的意思是把F中得.txt去掉后加上_1.txt
```

- 统计某文件夹下文件的个数: ls -l |grep "^-"|wc -l
- 统计某文件夹下目录的个数: ls -l |grep "^ｄ"|wc -l
- 统计文件夹下文件的个数，包括子文件夹里的: ls -lR|grep "^-"|wc -l
- 查看目录下所有文件的大小：ls -lh   或 ll -h
- 查看单独一个文件的大小：ls -lh test.log  或 du -h test.log

```
1. 查看CPU使用情况的命令
	vmstat 5   #每5秒刷新一次，最右侧有CPU的占用率的数据
	
2. Centos下测试硬盘读写速度
	写速度:
	dd if=/dev/zero bs=1k count=1000000 of=1Gb.file
	
	读速度:
	dd if=1Gb.file bs=64k |dd of=/dev/null
	
	读写速度:
	dd if=1Gb.file of=2Gb.file bs=64k
	
	用time来计时:
	time dd if=/dev/zero bs=1k count=1000000 of=1Gb.file
	time dd if=1Gb.file bs=64k |dd of=/dev/null   
	
	
	磁盘连续写入测试（268MB）
	dd if=/dev/zero of=kwxgd bs=64k count=4k oflag=dsync

	磁盘连续读取测试（268MB)
	dd if=kwxgd of=/dev/zero bs=64k count=4k iflag=direct

	以上测试为通过DD命令先写入一个268MB的文件，再通过DD命令读取。分配单元大小（簇）：4K。
	

3. netstat    显示网络各种情况的命令
		-a 显示所有socket，包括正在监听的。
		-c 每隔1秒就重新显示一遍，直到用户中断它。
		-i 显示所有网络接口的信息，格式同“ipconfig -e”。
		-n 以网络IP地址代替名称，显示出网络连接情形。
		-r 显示核心路由表，格式同“route -e”。
		-t 显示TCP协议的连接情况。
		-u 显示UDP协议的连接情况。
		-v 显示正在进行的工作。
		
		netstat -an
		TCP连接状态并汇总:
		netstat -an | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
		
4. 查看网络状态
	ifconfig -a
		
5. 查看网络流量：
	watch ifconfig
	cat /proc/net/dev
	watch more /proc/net/dev
```


```
用法为：
1、chmod +x ./traff.sh 将文件改成可执行脚本。 
2、./traff.sh eth0即可开始监看接口eth0流量，按ctrl+c退出。 

#!/bin/bash
while [ "1" ]
do
	eth=$1
	RXpre=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $2}')
	TXpre=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $10}')
	sleep 1
	RXnext=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $2}')
	TXnext=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $10}')
	clear
	echo  -e  "\t RX `date +%k:%M:%S` TX"
	RX=$((${RXnext}-${RXpre}))
	TX=$((${TXnext}-${TXpre}))
 
	if [[ $RX -lt 1024 ]];then
		RX="${RX}B/s"
	elif [[ $RX -gt 1048576 ]];then
		RX=$(echo $RX | awk '{print $1/1048576 "MB/s"}')
	else
		RX=$(echo $RX | awk '{print $1/1024 "KB/s"}')
	fi
 
	if [[ $TX -lt 1024 ]];then
		TX="${TX}B/s"
	elif [[ $TX -gt 1048576 ]];then
		TX=$(echo $TX | awk '{print $1/1048576 "MB/s"}')
	else
		TX=$(echo $TX | awk '{print $1/1024 "KB/s"}')
	fi
 
	echo -e "$eth \t $RX   $TX "
done
```


# 网络与数据包工具

* tcpdump 命令

## Centos方法技巧

* 在VMware里克隆出来的CentOS启动网卡（ifup eth0）时，出现：Device eth0 does not seem to be present，delaying initialization.错误。

```
解决办法:
	首先，打开/etc/udev/rules.d/70-persistent-net.rules，找到并记录下eth1网卡的mac地址xx:xx:xx:xx:xx:xx。

	然后，vi /etc/sysconfig/network-scripts/ifcfg-eth0，将 DEVICE="eth0"  改成  DEVICE="eth1"  ,将 HWADDR="原mac地址" 改成上面的mac地址。

	最后，重启网络：service network restart
```

* 给centos 改主机名：vi /etc/sysconfig/network 然后，HOSTNAME=你想到的名字。


## 修改ssh登陆端口号

```
1、vi /etc/sysconfig/iptables   
	添加 -A INPUT -p tcp -m state --state NEW -m tcp --dport 22045 -j ACCEPT
		
2、/etc/init.d/iptables restart

3、vi /etc/ssh/sshd_config  添加 Port 22045

4、重启SSH服务 /etc/init.d/sshd restart
	
	
centos 7.1 修改ssh登陆端口号
	
1、vi /etc/sysconfig/iptables
	添加 -A INPUT -p tcp -m state --state NEW -m tcp --dport 22045 -j ACCEPT
	注销 -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
	systemctl restart firewalld.service
	
2、vi /etc/ssh/sshd_config  添加 Port 22045
	
3、systemctl restart sshd.service
	
4、systemctl status sshd
	
5、yum -y install policycoreutils-python
	
6、semanage port -l | grep ssh

7、semanage port -a -t ssh_port_t -p tcp 22045

8、semanage port -l | grep ssh

9、systemctl restart sshd.service
```


### /bin/mv: 参数列表过长

```
mv ./* ../matchres/   提示参数列表过长

修改命令，使用如下命令：
ls | xargs -t -I {} mv {} ../matchres/ 可以把当前目录下的所有文件移到 "../matchres"下
```

### SSH登陆慢

```
在server上/etc/ssh/sshd_config文件中修改或加入UseDNS=no，另外在authentication gssapi-with-mic也有可能出现问题，在server上/etc/ssh/sshd_config文件中修改GSSAPIAuthentication no.
	
centos6.5 重启
/etc/init.d/sshd restart
	
centos7.1 重启
systemctl restart sshd.service
```
