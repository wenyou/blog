---
title: Windows使用记录
author: Zeeny
comments: false
date: 2022-12-02 12:48:15
updated: 2022-12-02 12:48:15
tags: [windows]
categories: windows
summary: Windows使用记录，windows usages
---

# 一、Windows 11 使用记录

## 1、快捷键

- 快速显示桌面  Win + D
- Win + Shift + s  ，调出截图工具Snipping Tool进行截图。





# 二、windows系统优化提速

## 1、清空临时文件

  	win + R 键，调出“运行” ，输入 %temp% ，删除所有临时文件。



# 三、PowerShell使用

## 1、常用命令

### 打开文件夹/目录命令:

```
# open the current directory in windows explorer
ii .
# open the windows directory in windows explorer
ii c:\windows 
# open book.xls in Excel
ii c:\book.xls 

#对于Powershell和cmd兼容的方式(最常见的方式):
start .
start c:\

#打开当前文件夹:
explorer.exe $(pwd)
```



### 查看运行端口

```
netstat -ano|findstr "8339"
```

​	&emsp;&emsp;查看运行端口，查看端口号为8339的程序。



