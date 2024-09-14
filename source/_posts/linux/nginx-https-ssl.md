---
title: nginx配置https配置SSL证书
author: Zeeny
comments: false
date: 2023-10-21
updated: 2023-10-21
tags: [linux,centos,https,ssl,nginx]
categories: [linux,centos,nginx]
summary: centos nginx 配置https 配置SSL证书
---

# centos nginx 配置https 配置SSL证书

## 1. 证书申请

* 证书申请前往阿里云SSL证书申请，选择免费证书申请即可。

* 使用aliyun 免费ssl证书

![alt text](/images/nginx-ssl-ca-image.png)


## 2、检查nginx SSL模块是否支持 

* 查看nginx是否安装http_ssl_module模块：

```
#命令：
nginx -V
```

* 显示：

```
nginx version: nginx/1.24.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.1.1t  7 Feb 2023
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_v2_module --with-http_gzip_static_module --with-http_sub_module --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_realip_module --with-openssl=/root/lnmp2.0/src/openssl-1.1.1t --with-openssl-opt='enable-weak-ssl-ciphers' --with-ld-opt='-ljemalloc'
```

* 有 --with-http_ssl_module 配置项，说明支持。
* 否则需要在安装nginx 时，添加ssl支持配置项，如下：

```
# 配置ssl模块
cd nginx-1.18.0
./configure --prefix=/usr/local/nginx --with-http_ssl_module
```

## 3、下载证书，上传服务器，配置nginx。

* nginx.conf

```
server
{
    listen 443 ssl;
    server_name stkit.cn;
    index index.html index.htm index.php;
    root  /home/zxsign;
    
    #start - SSL证书 配置         
    ssl_certificate      ../cert/aliyun_stkit.cn.pem;
    ssl_certificate_key  ../cert/aliyun_stkit.cn.key;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    
    # 表示使用的TLS协议的类型
    ssl_protocols TLSv1.2;
    #ssl_protocols  TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    # 表示使用的加密套件的类型
    #ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    #end - SSL证书 配置
    
    include enable-php.conf;

    location /nginx_status
    {
        stub_status on;
        access_log   off;
    }

    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
        expires      30d;
    }

    location ~ .*\.(js|css)?$
    {
        expires      12h;
    }

    location ~ /.well-known {
        allow all;
    }

    location ~ /\.
    {
        deny all;
    }

    access_log  /home/wwwlogs/zxsign-access.log;
}
```

* 查看openssl支持的加密套件
* 使用`openssl ciphers -v`查出服务具有的加密套件

```
#查看openssl支持的加密套件
openssl ciphers -v
```

* -v：详细列出所有加密套件。包括ssl版本（SSLv2、SSLv3以及 TLS）、密钥交换算法、身份验证算法、对称算法、摘要算法以及该算法是否可以出口。

*  nginx 默认配置是 HIGH:!aNULL:!MD5


## 4、将 http 重定向 https

```
server {
  listen 80;
  server_name stkit.cn;
  #将请求转成https
  rewrite ^(.*)$ https://$host$1 permanent;
}
```

## 5、重启nginx

```
# 重新载入配置文件
nginx -s reload /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

# 查看nginx 进程命令
ps aux | grep nginx
```

## 6、查看端口使用

```
netstat -lntp
```

## 7、开放端口

```
firewall-cmd --zone=public --add-port=443/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-ports
```