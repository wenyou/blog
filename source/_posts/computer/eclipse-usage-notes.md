---
title: Eclipse使用与常用插件
date: 2014-12-09
updated: 2014-12-09
categories: [计算机,开发工具]
tags: [Eclipse]
author: Zeeny
summary: Eclipse使用：快捷键与常用插件
comments: false
---



# Eclipse使用与常用插件

## 一、快捷键

* Ctrl+o (mac:command + o), 快速outline
* Ctrl + H，全局文件内容搜索
* Ctrl + Shift + R (mac:command + shift + r)，全局文件名搜索
* Ctrl + Shift + T (mac:command + shift + t)，JAVA类搜索
* Ctrl + F (mac:command + f)，文件中字符串搜索或替换
* F4 ，查看java类的继承结构
* Ctrl + Shift + L (mac:command + shift + l)，查看快捷键或者编辑快捷键
* Ctrl + L (mac:command + l)，行数搜索
* [Eclipse快捷键 10个最有用的快捷键](http://www.open-open.com/bbs/view/1320934157953/)


## 二、常用插件

* eclipse svn: Subclipse 1.8.x  -  http://subclipse.tigris.org/update_1.8.x
* jbossTools ：jbossTools   -  http://download.jboss.org/jbosstools/updates/development/luna/
* findbugs - http://findbugs.cs.umd.edu/eclipse


## 三、Eclipse启动提速

### 3.1 调整Eclipse的Preferences

* General > Startup and Shutdown : 移除所有在启动时加载的插件。
* General > Editors > Text Editors > Spelling : 关闭拼写检查。
* General > Validation > 勾选“Suspend all validator”。
* Window > Customize Perspective > 移除所有用不到或不想用的内容（尽量使用快捷键），菜单栏也是如此（你用过几次菜单栏的打印按钮？）。
* Install/Update > Automatic Updates > 取消勾选“Automatically find new updates and notify me”。
* General > Appearance > 取消勾选“Enable Animations”。
* 使用默认的主题。其他主题可能会降低运行速度。
