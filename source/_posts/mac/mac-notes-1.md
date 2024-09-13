---
title: Mac笔记
author: Zeeny
comments: false
date: 2014-04-03
updated: 2014-04-03
tags: [Mac]
categories: [Mac]
summary: Mac笔记
---


# Mac笔记

__一、Mac Tips__

* $say xxx 在终端输入say 后面加英语词句，即可语音读出。
* ctrl + space  打开spotlight 输入要找的程序和文件名，即可打开。
* sips 命令批量处理图片。1、把图片文件夹下的所有jpg图片宽度缩小为800px，高度按比例缩放：sips -Z 800 ./images/\*.jpg  2、顺时针旋转90度：sips -r 90 ./images/\*.jpg  3、垂直反转：sips -f vertical ./images/\*.jpg。
* 当使用Safari浏览网页时，如果想通过邮件把当前页面发送你自己或别人，可以用 Command + I 键，直接打开邮件并把当前网页附加到邮件中。
* du -sh \* 获取当前目录下各个文件和子目录各占多少空间。
* 英文自动完成，当使用系统软件文本编辑、Pages、Keynote时，输入英文开头几个字母时按esc键可以自动补全。
* 触发角：系统偏好设置->Mission Control->触发角，可以对屏幕的四个角进行设置了。
* 维护Mac，当使用系统时哪果发现出现异常，那么就该进行日常维护了，打开磁盘管理，选中系统盘，点击“修复磁盘权限”，对磁盘权限进行检查和修复。完成之后还可以手动执行维护脚本：sudo periodic daily , sudo periodic weekly , sudo periodic monthly  ；也可以一次全部执行：sudo periodic daily weekly monthly。这些操作可以定期执行。
* 关掉“使窗口按应用程序成组”，在Mission Control设置中把“使窗口按应用程序成组”关掉，Mission Control的行为就会跟10.7以前的expose一样，不会把同一个程序的多个窗口叠在一起，对程序员一个程序开多个窗口很有用。
* 截图：shift + command + 3: 全屏幕截图；shift + command + 4: 通过鼠标选取截图。截图是png文件，可以通过: defaults write com.apple.screencapture type -string JPEG，来修改截图文件类型。
* OS X中的ftp，1、终端输入ftp username@ftp.xxx.com 或者使用sftp通过ssh完成ftp功能，如：sftp username@172.16.37.4。2、利用OS X原生ftp工具，从Finder菜单栏中进入“前往-连接服务器...”，输入ftp服务器地址。
* 快速创建便笺，选中文字：shift + command + y。
* 终端open命令，open . 在Finder中打开当前目录。open /Users ,还可以直接打开文件、程序、网址等，如：open a.txt , open -a Safari , open -a TextMate a.txt , open http://www.baidu.com
* 激活窗口，如果在一个屏幕内打开了多个程序，除了当前激活的软件窗口，还想看其他窗口内容，按着CMD键拖动其他窗口，原来的窗口会一直在最上面。
* 语音识别，连接按两次fn键，呼出语音识别窗口。该功能需要联网。
* time命令，如果想知道在终端执行的某个程序耗时多久，对cpu等的使用情况，可以输入：time python fib.py
* 三指轻拍查找功能，把光标移到一个单词上面，无需选中，三指轻拍，系统就会弹出词典显示相关单词的释义。
* 输入pmset noidle，Mac不休眠，在终端输入pmset noidle，只要命令一直运行，Mac就不会进入睡眠状态。关掉终端或ctrl+c取消。
* pmset -g 查看当前电源的使用方案
* sudo pmset -b displaysleep 5 ,设置电池供电时，显示器5分钟内进入睡眠。
* sudo pmset schedule wake "04/05/14 02:00:00" ,设置电脑在2014年04月5日02点唤醒电脑。
* cmd + f3 显示桌面。 
* Space(空间)，四指上推，在桌面的最上方会出现当前的space，把鼠标移到space列表的右侧，会出现一个+号的空间，加击+号可以增加一个space.
* 隐藏程序，当不想在使用当前程序的时候看到其他程序的时候，可以使用快捷键option + cmd + h，把其他程序窗口隐藏起来，当想使用其他程序时用cmd + tab切换回来。
* locate php.ini 根据文件名快速查找文件。
* delete相当于退格键，fn + del 可以往前删。 fn + 上下左右方向键可以实现PageUp /PageDown /Home / End 的功能。
* QuickTime 可以用来 录像、录音、录制屏幕(打开quicktime后按：ctrl+cmd+n)。


__通用快捷键：__
* cmd + tab 向前循环切换应用程序。
* shift + cmd + tab 向后循环切换应用程序。
* cmd + del 删除文件
* shift + cmd + del 清倒废纸篓（有确认）
* shift + option + cmd + del 清倒废纸篓（无确认）
* cmd + ~ 同一应用程序多个窗口切换。
* cmd + Q 彻底退出当前应用程序。
* cmd + L 当前应用程序是游览器时，可以直接定位到地址栏。
* cmd + "+ / -"  放大缩小字体。


__特殊字符输入__
* 美元：shift + 4
* 美分：option + 4
* 英镑：option + 3
* 人民币：option + y
* 欧元：shift + option + 2
* 波折号：option + - 或 shift + option + -
* 省略号：option + ;
* 约等于：option + x 
* 度：shift + option + 8
* 除号：option + / 
* 无穷大：option + 5
* 小于等于：option + , 
* 大于等于：option + .
* 不等于：option + = 
* 圆周率Pi：option + p
* 正负±：shift + option + =
* 平方根√：option + v
* 总和∑：option + w
* 商标Trademark ™：option + 2
* 注册® ：option + r
* 版权 © ：option + g
