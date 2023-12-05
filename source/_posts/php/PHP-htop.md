---
title: 使用htop监控PHP的实时动作
author: Zeeny
comments: false
date: 2014-07-31
updated: 2014-07-31
tags: [php,linux,htop,top]
categories: [php,linux]
summary: 使用htop监控PHP的实时动作
---

# 使用htop监控PHP的实时动作

__一、安装htop__

1. htop网站：http://hisham.hm/htop/index.php?page=main

2. 下载：

   ```shell
   wget http://hisham.hm/htop/releases/1.0.3/htop-1.0.3.tar.gz
   ```

   

3. 安装：

   ```shell
   tar zvxf htop-1.0.3.tar.gz  ./configure  make && make install
   ```

   

4. 使用：
  &emsp;&emsp;安装完毕后，用于监控的环境就搭建好了。

  &emsp;&emsp;输入htop命令后，将看到强化版的top（系统监控）。将光标移动到php进程的位置按 ‘s’，就可以看到实时php进程的动作了，这些动作都是将php脚本翻译后的动作，均为直接的系统调用。
