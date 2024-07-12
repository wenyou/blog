---
layout: post
title: Java webservice ssl
description: Java webservice ssl
category: webservice
tags: [Java,webservice,ssl]
---
#Java webservice ssl#

* http://zjumty.iteye.com/blog/1885356
* http://blog.csdn.net/cookieweb/article/details/7531477
* http://blog.csdn.net/liangzhao_jay/article/details/24114505

```
	用keytool创建Keystore和Truststore文件:
	
	keytool -importcert -alias CA -file ca.crt -keystore keystore.jks
	keytool -list -v -keystore keystore.jks
	keytool -export -alias CA -keystore keystore.jks -rfc -file selfsignedcert.cer
	keytool -import -alias CA -file selfsignedcert.cer -keystore truststore.jks
	keytool -list -v -keystore truststore.jks 
	
```

* [unable to find valid certification path to requested target 的简单解决办法](http://borissun.iteye.com/blog/813707)
* [java调用webservice接口方法](http://yang-min.iteye.com/blog/600172)
* [java调用https下的WebService的代码实例](http://blog.donews.com/ooFrank/archive/2008/01/02/1242535.aspx)