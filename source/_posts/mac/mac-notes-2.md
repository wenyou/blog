---
title: Mac笔记之杂记
author: Zeeny
comments: false
date: 2014-08-29
updated: 2014-08-29
tags: [Mac]
categories: [Mac]
summary: Mac笔记之杂记
---



# Mac笔记之杂记

1. command + k ,调出连接服务器窗口,link eg:ftp://root@172.16.18.112:28022/,afp://root@172.16.18.112

2. Mac 访问 Windows 的共享文件夹：command + k，在连接服务器对话框中输入「smb://Windows主机的IP地址」，其中 smb 是访问 Windows 共享文件夹所使用的协议名称。

3. 通过命令释放 Mac OS X 内存空间:  purge

4. netstat -nr 查看本地路由表，linux 下用route -n 命令。

5. 选中一个或多个文件，按空格键  -> 可以预览文件、图片。

6. 右键：按住 control + 单击。

7. 抓屏 command+shift+数字键[3,4] 3是全屏,4是自定义大小，存放在桌面。

8. 强制关闭程序：option + command + esc

9. Mac命令行下切换到root用户：sudo -i

10. Mac打包时加密码：
 		
 		压缩打包一个文件：
 		zip -e test.zip test.txt
 		 
 		压缩打包一个文件夹:
 		zip -er ~/Desktop/demo.zip ~/Desktop/demo/  