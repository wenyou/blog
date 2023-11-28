---
title: windows常用命令
author: Zeeny
comments: false
date: 2018-01-24 11:46:45
updated: 2023-11-10 11:46:45
tags: [windows,cmd]
categories: windows
summary: windows常用命令,windows common commands#
---

## 1、查看系统信息

**命令：**

```
systeminfo
```

**例如： **

​	&emsp;&emsp;如果不确定自己计算机的类型，请打开命令提示符或 PowerShell，并输入：

```
systeminfo | find "System Type"
systeminfo | find "系统类型"
```

 	&emsp;&emsp;**Caveat：** 在非英文版 Windows 上，你可能必须修改搜索文本，对“System Type”字符串进行翻译。 你可能还需要对引号进行转义来用于 find 命令。 例如，在中文版中使用 systeminfo | find "系统类型"。



## 2、chocolatey

​	&emsp;&emsp;在 powershell 下执行 choco 命令，可以安装软件包等相关操作。

​	&emsp;&emsp;chocolatey 是windows下的一个包管理软件，相当于 Linux  的 apt-get。

​	&emsp;&emsp;以管理员身份打开powershell ，执行以下代码，安装Chocolatey：

```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

​	

## 3、启动mysql

```
net stop mysql

net start mysql
```



## 4、关机命令

​	&emsp;&emsp;在cmd命令行里输入:

```
# 60秒后关机
shutdown -s -t 60

# 取消关机：
shutdown -a
```



## 5、Wifi相关命令

​	&emsp;&emsp;Windows查看WIFI的BSSID:

​	&emsp;&emsp;WIFI热点的BSSID其实就是WIFI的MAC地址，Windows查看WIFI MAC地址，可以通过命令行实现。

​	&emsp;&emsp;打开命令提示符cmd.exe，输入以下命令，回车，可以看到Windows系统能够发现的附近所有WIFI的BSSID和SSID等详细信息。

```
netsh wlan show networks mode=bssid
```

​	&emsp;&emsp;命令显示的内容：其中SSID为WIFI的热点名称，BSSID冒号后面为WIFI的MAC地址。



​	&emsp;&emsp;除了上述命令查看所有附近WIFI的BSSID和SSID以外，也可以运行 *netsh wlan show interfaces* 命令查看已经连接的WIFI的BSSID和SSID。

```
netsh wlan show interface

netsh wlan show drivers

netsh wlan show all
```

