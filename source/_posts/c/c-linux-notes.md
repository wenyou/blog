---
title: linux C编译运行测试
author: Zeeny
comments: false
date: 2019-08-04
updated: 2019-08-04
tags: [C,C++]
categories: [C,C++]
summary: linux C编译运行测试
---

# 一、linux C编译运行

* 编写回声服务器端实现代码:

```
编译：
gcc echo_server.c -o echo_server

运行：
./echo_server

ssh root@192.168.156.12 
gcc echo_server.c -o echo_server
./echo_server
```

# 二、客户端测试

* 使用 telnet 测试“回声服务器”

`telnet ip port`

```
telnet 192.168.156.12 777
```

