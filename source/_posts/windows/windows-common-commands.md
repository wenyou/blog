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

**例如：**

​	&emsp;&emsp;如果不确定自己计算机的类型，请打开命令提示符（cmd) 或者 PowerShell，并输入：

```
systeminfo | find "System Type"
systeminfo | find "系统类型"
```

​	&emsp;&emsp;**Caveat：** 在非英文版 Windows 上，你可能必须修改搜索文本，对“System Type”字符串进行翻译。 你可能还需要对引号进行转义来用于 find 命令。 例如，在中文版中使用 systeminfo | find "系统类型"。

**例如：**

​	&emsp;&emsp;获取操作系统安装日期：使用systeminfo命令获取（初始安装日期），获取结果如下：
```
主机名:           xxx-PC
OS 名称:          Microsoft Windows 10 专业版
OS 版本:          10.0.10240 暂缺 Build 10240
OS 制造商:        Microsoft Corporation
OS 配置:          独立工作站
OS 构件类型:      Multiprocessor Free
注册的所有人:     pc-4
注册的组织:
产品 ID:          00330-80000-00000-AA866
初始安装日期:     2017-01-04, 09:47:21
系统启动时间:     2024-07-12, 09:21:06
系统制造商:       Gigabyte Technology Co., Ltd.
系统型号:         H81M-DS2
系统类型:         x64-based PC
处理器:           安装了 1 个处理器。
                  [01]: Intel64 Family 6 Model 60 Stepping 3 GenuineIntel ~3201 Mhz
BIOS 版本:        American Megatrends Inc. F2, 2015-08-06
Windows 目录:     C:\Windows
系统目录:         C:\Windows\system32
启动设备:         \Device\HarddiskVolume2
。。。。
```


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


## 6、硬盘序列号：使用diskpart获取
​	&emsp;&emsp;打开命令提示符cmd.exe，输入以下命令：diskpart，回车，打开diskpart命令行窗口，在窗口中输入以下命令：list disk，回车，显示磁盘列表。再次输入 select disk 1，回车，选择一个磁盘，输入 detail disk，回车，显示磁盘详细信息，即可获取硬盘序列号。
```
DISKPART> list disk

  磁盘 ###  状态           大小     可用     Dyn  Gpt
  --------  -------------  -------  -------  ---  ---
  磁盘 0    联机              465 GB      0 B        *
  磁盘 1    联机              465 GB  1024 KB

DISKPART> select disk 1

磁盘 1 现在是所选磁盘。

DISKPART> detail disk

ST500DM002-1ER14C ATA Device
磁盘 ID: C81657A2
类型   : ATA
状态 : 联机
路径   : 0
目标 : 0
LUN ID : 0
位置路径 : ACPI(_SB_)#ACPI(PCI0)#ACPI(SAT1)#ACPI(CHN0)#ATA(C00T00L00)
当前只读状态: 否
只读: 否
启动磁盘: 否
页面文件磁盘: 否
休眠文件磁盘: 否
故障转储磁盘: 否
群集磁盘  : 否

  卷 ###      LTR  标签         FS     类型        大小     状态       信息
  ----------  ---  -----------  -----  ----------  -------  ---------  --------
  卷     4     E   zwybk        NTFS   磁盘分区         465 GB  正常
```

## 7、查看U盘启用时间
​	&emsp;&emsp;插入u盘，点击右键，选择属性，点击“硬件”选项卡，在所有磁盘驱动器列表中找到u盘，点击“属性”，在“详细信息”选项卡中，在属性里找到“安装日期”或者“驱动器启用时间”，查看u盘的启用时间。