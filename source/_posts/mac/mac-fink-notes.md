---
title: Mac下Fink的使用
author: Zeeny
comments: false
date: 2014-03-26
updated: 2014-03-26
tags: [Mac,fink]
categories: [Mac]
summary: Mac下fink的安装与使用
---


# Mac下Fink的安装与使用

__1 安装__

* 参考如下网址: http://scottlab.ucsc.edu/~wgscott/xtal/wiki/index.php/Installing_Fink_on_10.6_and_10.7#Just_the_Base_Install:_10.7


__2 删除__

* Fink 的所有文件几乎都安装在 /sw （或你选择安装的地方）。因此，如果你想删除 Fink，输入下面的命令：sudo rm -rf /sw


__3 升级fink__

* fink selfupdate
* fink selfupdate-rsync
* fink index -f
* fink selfupdate


__4 安装软件__

* fink install xxx
* 重新安装整个包
* fink reinstall


__5 卸载软件__

* fink remove xxx  如果想把依赖包也一起卸载，加-r。如果想配置文件一并卸载，用
   fink purge ，类似与ubuntu里面的remove –purge。


__6 更新所有已安装包__

* fink update-all

__7 查看可安装包__

* fink list xxx 或者 fink apropos xxx
* 也支持正泽表达式 fink list “xxx\*”

__8 查看相关包的描述__

* fink info


__9 显示某个包的依赖关系__

* fink show-deps xxx

