---
title: MAC OS 10.10 转发80(HTTP)端口的方法
author: Zeeny
comments: false
date: 2015-02-11
updated: 2015-02-11
tags: [Mac,HTTP]
categories: [Mac]
summary: MAC OS 10.10 转发80(HTTP)端口的方法
---



# MAC OS 10.10 转发80(HTTP)端口的方法

<p>MAC OS 本质上还是 Unix 系统, Unix 系统大多默认情况下非root用户是无法使用小于1024的常用端口的。这时候如果你开发中需要在普通用户下用到80端口, 比如tomcat, 比如vitualbox下构建了一个http服务, 若你想直接通过浏览器的 localhost访问的话(不用加上莫名其妙的“:端口”的话)你就需要做一些系统端口转发的工作。</p>


* MAC OS 10.10 上 pfctl 就可以做这一件事情, 详情请参见

```
man pfctl
```

或者

```
man pf.conf
```

* 具体操作如下:

首先在 `/etc/pf.anchors/` 新建一个 `http` 文件内容如下:

```
rdr pass on lo0 inet proto tcp from any to any port 80 -> 127.0.0.1 port 8080

rdr pass on lo0 inet proto tcp from any to any port 443 -> 127.0.0.1 port 4443

rdr pass on en0 inet proto tcp from any to any port 80 -> 127.0.0.1 port 8080

rdr pass on en0 inet proto tcp from any to any port 443 -> 127.0.0.1 port 4443
```

* 然后使用 `pfctl` 命令检测配置文件

```
sudo pfctl -vnf /etc/pf.anchors/http
```

* 如果没有报错(正确的打印了配置信息, 没有明显的出错信息), 即修改pf的主配置文件`/etc/pf.conf`, 来引入这个转发规则:

在

```
rdr-anchor "com.apple/*"
```

下, 添加如下 anchor 声明:

```
rdr-anchor "http-forwarding"
```

<p>`pf.conf`对指令的顺序有严格要求, 否则会报出` Rules must be in order: options, normalization, queueing, translation, filtering` 的错误, 所以相同的指令需要放在一起。</p>

* 再在

```
load anchor "com.apple" from "/etc/pf.anchors/com.apple"
```
下, 添加 anchor 引入:

```
load anchor "http-forwarding" from "/etc/pf.anchors/http"
```

* 最后, 导入并允许运行 pf

```
sudo pfctl -ef /etc/pf.conf
```


* 如果需要开机启动, 则需要为 

```
/System/Library/LaunchDaemons/com.apple.pfctl.plist
```

* 针对 pfctl 的启动项, 新增一个 -e (允许) 参数, 这样, pf 规则开机机器可以生效了。