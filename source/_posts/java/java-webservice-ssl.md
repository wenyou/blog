---
title: Java webservice ssl
author: Zeeny
comments: false
date: 2014-12-11
updated: 2014-12-11
tags: [Java,Webservice,ssl]
categories: [Java]
summary: Java webservice ssl
---


# Java Webservice ssl

```
用keytool创建Keystore和Truststore文件:
	
keytool -importcert -alias CA -file ca.crt -keystore keystore.jks
keytool -list -v -keystore keystore.jks
keytool -export -alias CA -keystore keystore.jks -rfc -file selfsignedcert.cer
keytool -import -alias CA -file selfsignedcert.cer -keystore truststore.jks
keytool -list -v -keystore truststore.jks 
```