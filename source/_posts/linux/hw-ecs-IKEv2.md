---
title: 华为ecs云服务器IKEv2 IPSec VPN配置
author: Zeeny
comments: false
date: 2022-03-28
updated: 2022-03-28
tags: [linux,centos,ecs,云主机,vpn]
categories: [linux,centos]
summary: 华为ecs云服务器IKEv2 IPSec VPN配置
---


# 华为ecs云服务器IKEv2 IPSec VPN配置

## 1、安装strongswan

```
yum -y install epel-release    
yum -y install openssl
yum -y install strongswan
systemctl enable strongswan    (设置开机启动)
```

## 2、创建证书

* 创建 CA 根证书:

```
strongswan pki --gen --outform pem > ca.key.pem
strongswan pki --self --in ca.key.pem --dn "C=CN, O=stkit, CN=stkit StrongSwan CA" --ca --lifetime 3650 --outform pem > ca.cert.pem
```

* 创建服务器端证书:

```
strongswan pki --gen --outform pem > server.key.pem
strongswan pki --pub --in server.key.pem --outform pem > server.pub.pem
strongswan pki --issue --lifetime 3650 --cacert ca.cert.pem --cakey ca.key.pem --in server.pub.pem --dn "C=CN, O=stkit, CN=119.8.167.90" --san="119.8.167.90" --flag serverAuth --flag ikeIntermediate --outform pem > server.cert.pem
```

* 创建客户端证书:

```
strongswan pki --gen --outform pem > client.key.pem
strongswan pki --pub --in client.key.pem --outform pem > client.pub.pem
strongswan pki --issue --lifetime 3650 --cacert ca.cert.pem --cakey ca.key.pem --in client.pub.pem --dn "C=CN, O=stkit, CN=119.8.167.90" --outform pem > client.cert.pem
```

* 打包证书为 pkcs12:

```
openssl pkcs12 -export -inkey client.key.pem -in client.cert.pem -name "stkit StrongSwan Client Cert" -certfile ca.cert.pem -caname "stkit StrongSwan CA" -out client.cert.p12
```

* 安装证书:

```
cp -r ca.key.pem /etc/strongswan/ipsec.d/private/
cp -r ca.cert.pem /etc/strongswan/ipsec.d/cacerts/
cp -r server.cert.pem /etc/strongswan/ipsec.d/certs/
cp -r server.pub.pem /etc/strongswan/ipsec.d/certs/
cp -r server.key.pem /etc/strongswan/ipsec.d/private/
cp -r client.cert.pem /etc/strongswan/ipsec.d/certs/
cp -r client.key.pem /etc/strongswan/ipsec.d/private/
```


<p>把 CA 证书（ca.cert.pem）、客户端证书（client.cert.pem）和 .p12 证书（client.cert.p12）用 FTP 复制出来给客户端用。</p>

<p>注：自创证书不受客户操作系统信任，需要发给客户端导入使用。如果要创建不需要客户端导入证书的vpn，需要使用CA机构的证书，可以使用下面的方式申请免费ssl证书。</p>

* 申请 ssl 证书
<p>StrongSwan IPsec IKEv2 连接需要用到服务器证书，用于验证服务器身份。由于自签发证书不受操作系统信任，我们需要申请 Let’s Encrypt 免费证书。申请证书需要有域名，提前将域名解析到你的vps地址。 https://letsencrypt.org/</p>


## 3、配置 VPN

* 修改主配置文件 ipsec.conf
* vi /etc/strongswan/ipsec.conf

```
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration

config setup
    # strictcrlpolicy=yes
    uniqueids = never

# Add connections here.

# Sample VPN connections

#conn sample-self-signed
#      leftsubnet=10.1.0.0/16
#      leftcert=selfCert.der
#      leftsendcert=never
#      right=192.168.0.2
#      rightsubnet=10.2.0.0/16
#      rightcert=peerCert.der
#      auto=start

#conn sample-with-ca-cert
#      leftsubnet=10.1.0.0/16
#      leftcert=myCert.pem
#      right=192.168.0.2
#      rightsubnet=10.2.0.0/16
#      rightid="C=CH, O=Linux strongSwan CN=peer name"
#      auto=start
conn %default
    compress = yes
    esp = aes256-sha256,aes256-sha1,3des-sha1!
    ike = aes256-sha256-modp2048,aes256-sha1-modp2048,aes128-sha1-modp2048,3des-sha1-modp2048,aes256-sha256-modp1024,aes256-sha1-modp1024,aes128-sha1-modp1024,3des-sha1-modp1024!
    keyexchange = ike
    keyingtries = 1
    leftdns = 8.8.8.8,8.8.4.4
    rightdns = 8.8.8.8,8.8.4.4

conn IKEv2-BASE
    # 服务器端根证书 DN 名称
    leftca = "C=CN, O=stkit, CN=stkit StrongSwan CA"
    # 是否发送服务器证书到客户端
    leftsendcert = always
    # 客户端不发送证书
    rightsendcert = never

conn IKEv2-EAP
    leftca = "C=CN, O=stkit, CN=stkit StrongSwan CA"
    leftcert = server.cert.pem
    leftsendcert = always
    rightsendcert = never
    leftid = 119.8.167.90
    left = %any
    right = %any
    leftauth = pubkey
    rightauth = eap-mschapv2
    leftfirewall = yes
    leftsubnet = 0.0.0.0/0
    rightsourceip = 10.1.0.0/16
    fragmentation = yes
    rekey = no
    eap_identity = %any
    auto = add
```

* 修改 DNS 配置
* vim /etc/strongswan/strongswan.d/charon.conf

```
设置 Windows 公用 DNS，去掉dns1和”dns2前面的井号（#）。
dns1 = 8.8.8.8
dns2 = 8.8.4.4
```

* 配置用户名与密码
* vim /etc/strongswan/ipsec.secrets

```
# ipsec.secrets - strongSwan IPsec secrets file
# 使用证书验证时的服务器端私钥
# 格式 : RSA <private key file> [ <passphrase> | %prompt ]
: RSA server.key.pem

# 使用预设加密密钥, 越长越好
# 格式 [ <id selectors> ] : PSK <secret>
%any %any : PSK "hw20220101"

# EAP 方式, 格式同 psk 相同
Zeeny %any : EAP "hw123456"
Olly %any : EAP "hw123456"

# XAUTH 方式, 只适用于 IKEv1
Zeeny %any : XAUTH "hw123456"
Olly %any : XAUTH "hw123456"
```

* 开启内核转发
* vim /etc/sysctl.conf

```
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
```

* 最后重新加载系统参数，使上面的配置生效，执行命令：`sysctl -p`


## 4、配置防火墙

```
firewall-cmd --permanent --add-service="ipsec"

# ESP (the encrypted data packets)
firewall-cmd --permanent --add-rich-rule='rule protocol value="esp" accept'
# AH (authenticated headers)
firewall-cmd --permanent --add-rich-rule='rule protocol value="ah" accept'

# IKE  (security associations)
firewall-cmd --permanent --add-port=500/udp
# IKE NAT Traversal (IPsec between natted devices)
firewall-cmd --permanent --add-port=4500/udp

firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.1.0.0/16" masquerade'


firewall-cmd --permanen --add-rich-rule='rule family="ipv4" source address="10.1.0.0/16" forward-port port="4500" protocol="udp" to-port="4500"'
firewall-cmd --permanen --add-rich-rule='rule family="ipv4" source address="10.1.0.0/16" forward-port port="500" protocol="udp" to-port="500"'

firewall-cmd --reload
firewall-cmd --list-all
```


## 5、strongSwan 服务操作

```
使用 strongswan 自身的命令

# 停止服务
strongswan stop

# 查看是否连接了客户端
strongswan status

# 查看命令帮助
strongswan --help


使用 systemctl 命令
# 设置开机启动 strongswan 服务
systemctl enable strongswan

# 启动服务
systemctl start strongswan

# 停止服务
systemctl stop strongswan

# 重启服务
systemctl restart strongswan

# 查看服务状态
systemctl status strongswan
```

* 注意，如果使用strongswan restart命令重启 strongSwan 后，再用systemctl status strongswan命令得不到正确的运行状态。

```
systemctl unmask firewalld #取消服务的锁定
systemctl start firewalld.service #开启服务
systemctl enable firewalld.service #设置开机启动
systemctl status firewalld
```


## 6、客户端使用vpn

### IOS 系统

* 先导入 CA 证书，将之前创建的`ca.cert.pem`用 FTP 导出，写邮件以附件的方式发到邮箱, 在 IOS 浏览器登录邮箱，下载附件，安装 CA 证书。

#### 1、使用 IKEv2 + EAP 认证

* 找到手机上“设置->VPN->添加配置”，选`IKEv2`。

    - 描述：随便填
    - 服务器：填 URL 或 IP
    - 远程 ID：ipsec.conf 中的 leftid
    - 用户鉴定：用户名
    - 用户名：EAP 项用户名
    - 密码：EAP 项密码

#### 2、使用 IKEv2 + 客户端证书 认证

* 把之前的 .p12 证书（里面包含 CA 证书）发到邮箱在手机上打开。导入到手机（此时需要之前设置的证书密码）。
* 找到手机上“设置->VPN->添加配置’，选 IKEv2 。

    - 描述：随便填
    - 服务器：填 URL 或 IP
    - 远程ID：ipsec.conf 中的 leftid
    - 用户鉴定：证书
    - 证书：选择安装完的客户端证书

#### 3、使用 IKEv2 + 预设密钥 认证

* 找到手机上“设置->VPN->添加配置”，选 IKEv2 。

    - 描述：随便填
    - 服务器：填 URL 或 IP
    - 远程ID：ipsec.conf 中的 leftid
    - 用户鉴定：无
    - 使用证书：关
    - 密钥：PSK 项密钥

### Windows 10

* 导入证书：

    - 将 CA 根证书 ca.cert.pem 重命名为 ca.cert.crt；
    - 双击 ca.cert.crt 开始安装证书；
    - 点击安装证书；
    - “存储位置”选择“本地计算机”，下一步；
    - 选择“将所有的证书都放入下列存储区”，点浏览，选择“受信任的根证书颁发机构”，确定，下一步，完成；
    建立连接：
    - “控制面板”-“网络和共享中心”-“设置新的连接或网络”-“连接到工作区”-“使用我的 Internet 连接”；
    - Internet 地址写服务器 IP 或 URL；
    - 描述随便写；
    - 用户名密码写之前配置的 EAP 的那个；
    - 确定；
    - 转到 控制面板网络和 Internet 网络连接；
    - 在新建的 VPN 连接上右键属性然后切换到“安全”选项卡；
    - VPN 类型选 IKEv2 ；
    - 数据加密选“需要加密”；
    - 身份认证这里需要说一下，如果想要使用 EAP 认证的话就选择“Microsoft : 安全密码(EAP-MSCHAP v2)”；想要使用私人证书认证的话就选择“使用计算机证书”；
    - 再切换到“网络”选项卡，双击“Internet 协议版本 4”以打开属性窗口，这里说一下，如果你使用的是老版本的 Win10，可能会打不开属性窗口，这是已知的 Bug，升级最新版本即可解决；
    - 点击“高级”按钮，勾选“在远程网络上使用默认网关”，确定退出；

* Windows 7 导入证书略有不同

    - 开始菜单搜索“cmd”，打开后输入 MMC（Microsoft 管理控制台）；
    - “文件”-“添加/删除管理单元”，添加“证书”单元；
    - 证书单元的弹出窗口中一定要选“计算机账户”，之后选“本地计算机”，确定；
    - 在左边的“控制台根节点”下选择“证书”-“受信任的根证书颁发机构”-“证书”，右键“所有任务”-“导入”打开证书导入窗口；
    - 选择 CA 证书 ca.cert.crt 导入即可；

* 注意，千万不要双击 .p12 证书导入！因为那样会导入到当前用户而不是本机计算机中。
