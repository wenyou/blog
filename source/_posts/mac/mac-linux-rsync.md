---
title: 在Mac上把远程linux服务器上的目录映射到本机
author: Zeeny
comments: false
date: 2014-10-11
updated: 2014-10-11
tags: [Mac]
categories: [Mac]
summary: 在Mac上把远程linux服务器上的目录映射到本机的解决方案
---


# 在Mac上把远程linux服务器上的目录映射到本机

* 作为Web开发人员，经常要做的一个事情，就是将源代码上传到服务器上进行测试或运行。

* 有以几种方式可以把本机代码上传或同步到远程服务器目录里。

__1. FTP__

__2. SFTP__

__3. 版本控制工具-SVN，Git，等__

__4. NFS__


__5. rsync__

* rsync 特点：

```
增量 – 强大的原因，对比文件差异，然后只传输差异部分
压缩 – 强大的原因，怎么实现秒传？传得越少，速度越快
黑名单 – 某些目录和文件是永远不用传的
安全 – 有权限校验，可以走ssh通道，根据主机、或者用户名
连续 – 不是一次只传一个文件
```

* rsync 几种使用方式：

```
使用方式1：
rsync --rsh=ssh -avz SRC user@host:/path/to/DST
	
	使用这个命令，就可以把SRC目录，拷贝到服务器上的/path/to/DST/目录下了。而且是每次把差异部分给拷贝过去，非常快。
	
eg:
rsync --rsh=ssh  -avz ./myde root@172.16.37.2:/mnt/test
	需要输入172.16.37.2的密码。通过ssh帐户(需要密码)执行rsync，将文件同步镜像到远程服务器。
	

如何无需密码通过ssh执行rsync来同步文件？
1.使用ssh-keygen生成密匙
	现在我们来设置ssh,以便在执行ssh操作时不需要密码，使用ssh-keygen在本地生成公钥和私钥。
	在客户端（本地机）命令行中执行：ssh-keygen
		youwenzhangMacBook-Pro:webjava zxsoft$ ssh-keygen
	当提示输入密码时，只需输入两次回车键，不指配密码字符。
	

2.使用ssh-copy-id将公匙拷贝至远程主机
	执行ssh-copy-id,将通过ssh-keygen生成的公匙拷贝至远程主机。
		youwenzhangMacBook-Pro:webjava zxsoft$ ssh-copy-id -i ~/.ssh/id_rsa.pub 172.16.37.2
	执行以上操作时，将会提示输入远程主机帐户和密码，然后就会自动将公匙拷贝至远程目录。
	
	如果本地Mac机上没有 ssh-copy-id 命令，则可以使用以下命令代替ssh-copy-id:
		cat ~/.ssh/id_rsa.pub | ssh root@172.16.37.2 "umask 077; mkdir -p .ssh ; cat >> .ssh/authorized_keys"	
	

3.现在就可以无需密码通过ssh来执行rsync了。
	现在，你也可以不需要密码就可以ssh连接到远程主机：
		ssh root@172.16.37.2
	
	重新来执行rsync，现在应该就不会提示输入密码了,如下：
		rsync --rsh=ssh  -avz ./myde root@172.16.37.2:/mnt/test
	
	
	
	不过这还需要手动执行rsync命令，如果要自动激发执行，在linux下可以用crontab来定时执行任务，在mac下可以用launchctl来定时执行任务。
	
	于是在本地机(Mac机)的 ~/Library/LaunchAgents 目录中建立 myjava.server.rsync.37.2.plist 文件,内容如下：
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
		<plist version="1.0">
			<dict>
				<key>Label</key>
				<string>myjava.server.rsync.37.2</string>
				<key>ProgramArguments</key>
				<array>
					<string>/usr/bin/rsync</string>
					<string>-avz</string>
					<string>--rsh=ssh</string>
					<string>/Users/zxsoft/USVN/Java</string>
					<string>root@172.16.37.2:/mnt/java</string>
				</array>
				<key>WatchPaths</key> 
				<array>
					<string>/Users/zxsoft/USVN/Java/</string>
				</array>
				</dict>
			</plist>
     
然后执行：
	launchctl load ~/Library/LaunchAgents/myjava.server.rsync.37.2.plist
	launchctl start myjava.server.rsync.37.2
		就实现了每当目录下有文件变化，就自动同步到服务器了。
     
如果需要停止服务，则执行如下命令：
	launchctl stop myjava.server.rsync.37.2
	launchctl unload ~/Library/LaunchAgents/myjava.server.rsync.37.2.plist 
```