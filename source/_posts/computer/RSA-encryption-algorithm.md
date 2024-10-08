---
title: RSA加密算法
date: 2014-10-17
updated: 2014-10-17
categories: 计算机
tags: [加密算法,RSA]
author: Zeeny
summary: RSA加密算法
comments: false
---


# RSA加密算法

__1.C语言实现RSA加密解密__

```
在RSA使用过程中，公钥加密一般用来协商密钥；私钥加密一般用来签名。
	n：模数
	e：公钥指数
	d：私钥指数
	n+e可以组成公钥
	n+d可以组成私钥

代码包括
	1、生成RSA的数据结构
	2、用指定的n,e,d生成RSA的数据结构
	3、用私钥加密
	4、用公钥解密
	5、SHA256报文摘要

	#include <stdio.h>   
	#include <stdlib.h>   
	#include <string.h>   
	#include <openssl/sha.h>   
	include <openssl/rsa.h>   
	
	void testRSAGen() {  
    	RSA *r;  
    	int bits = 512, ret;  
			unsigned long e = RSA_3;  
    	BIGNUM *bne;  
    	r=RSA_generate_key(bits, e, NULL, NULL);  
    	RSA_print_fp(stdout,r,11);  
    	printf("--------------------------------\n");  

    	RSA_free(r);  
    	bne = BN_new();  
    	ret = BN_set_word(bne,e);  
    	r = RSA_new();  
    	ret = RSA_generate_key_ex(r,bits,bne,NULL); 
    	if(ret!=1)  {  
    		printf("RSA_generate_key_ex err!/n");  
    		return -1;  
    	}  
    	//RSA_print_fp(stdout,r,11);
    	RSA_free(r);
	}


	void testRSA() {  
    	RSA *r;  
    	BIGNUM *bne, *bnn, *bnd;  
  
    	int bits = 1024, ret, len, flen, padding, i;  
    	unsigned char *key, *p;  
    	BIO *b;

    	//要加密的明文   
    	unsigned char *in = "abcef";  
    	unsigned char *encData, *decData, *tmpData; //加密后的数据/解密后的数据/临时指针   
  
    	//使用的密匙数据   
    	unsigned long e = 75011;  
    	const char *MODULUS = "9EC7D9A2DC5B095F8E5F90295121F41262FAEFBE9AF57B772A71F1F9D9635F8769CB78DA2BCFE9B27FC1F3AD4A3D178F8C61981225EF5DEACBDC5665F12E691AA13DDD321A59CFCF376F002036612FF3C5E057A3007FF675AFA3EDE34DC23A1A2637294870EBE823F76B5CE21E25F3FA5137F5DE12437DE0118245B927B28221";  

    	const char *PRIVATE = "8B26E30ECA6E8F3668F6FA78B0C55FB75A4A3FAD0667B152933A4991D7A815D1498F5E1EF44ACEF6CDF252E56F367DED5BA024DF6B267B7E36BD35552DFA0A4CC1E9D0A4BC8E7C76F98D4971441D6693745A0A76E175571BD160E4B1536A6EFF5A08EDA45236E96E7A4748CF4D031CA8B2F4CCE9F2E1286F432DE6495A535E43";  
  
    	//构建RSA数据结构
    	bne = BN_new();  
    	bnd = BN_new();  
    	bnn = BN_new();  
    	ret = BN_set_word(bne, e);  
    	BN_hex2bn(&bnd, PRIVATE);  
    	BN_hex2bn(&bnn, MODULUS);  
  
    	r = RSA_new();  
    	r->e = bne;  
    	r->d = bnd;  
    	r->n = bnn;  
    	RSA_print_fp(stdout, r, 5);
  
    	//准备输出的加密数据结构   
    	flen = RSA_size(r); // - 11;
    	encData = (unsigned char *)malloc(flen);
    	bzero(encData, flen); //memset(encData, 0, flen);
  
    	printf("Begin RSA_private_encrypt ...\n");  
    	
    	//RSA_PKCS1_PADDING , flen -= 11;
			//RSA_NO_PADDING , flen = flen;
			//RSA_X931_PADDING , flen = flen - 2;
    	
    	ret = RSA_private_encrypt(flen, in, encData, r,  RSA_NO_PADDING);
    	//flen：表示待处理数据的长度。对于加密操作来说，就是待加密明文的长度。
    	//ret = RSA_private_encrypt(strlen(in), in, encData, r, RSA_PKCS1_PADDING);
    	
    	//RSA_private_encrypt 返回值是加密后密文的长度
    	
    	if(ret < 0) {
				printf("Encrypt failed!\n");  
				return;  
    	} 
    	
			//char *base64_encData = base64_encode(encData,strlen(encData)); //这里不能用strlen取长度，因为RSA加密后的字符串中可能含有\0结束符，就截断了。用下面的ret代替strlen(encData)
			//char *base64_encData = base64_encode(encData,ret);
    	 
    	printf("Size: %d\n", ret);
    	printf("ClearText: %s\n", in);
    	printf("CipherText(Hex): \n");

    	tmpData = encData;
    	for(i=0; i<ret; i++) {
				printf("0x%02x, ", *tmpData);
				tmpData++;
    	}

    	printf("end private encrypt \n");  
    	printf("------------------------\n");  
  
    	//准备输出的解密数据结构   
    	flen = RSA_size(r); // - 11;
    	decData = (unsigned char *)malloc(flen);
    	bzero(decData, flen); //memset(encData, 0, flen);
  
    	printf("Begin RSA_public_decrypt ...\n");  
    	ret = RSA_public_decrypt(flen, encData, decData, r,  RSA_NO_PADDING);
    	if(ret < 0) {
				printf("RSA_public_decrypt failed!\n");  
				return;  
    	}

    	printf("Size:%d\n", ret);
    	printf("ClearText:%s\n", decData);
  
    	free(encData);
    	free(decData);
    	RSA_free(r);
	}


	void testSHA256(){  
    	unsigned char in[] = "asdfwerqrewrasfaser";
    	unsigned char out[32];  
  
    	size_t n;  
    	int i;  
  
    	n = strlen((const char*)in);  
  
    	SHA256(in, n, out);  
    	printf("\n\nSHA256 digest result:\n"); 
    	printf("%d\n",sizeof(out));
  
    	for(i=0;i<32;i++)  
        	printf("%d",out[i]);

    	printf("\n");  
	}
	

	int main(void) {
    	puts("!!!Hello World!!!");
    	testSHA256();
    	testRSA();
    	return EXIT_SUCCESS;
	} 
```

```
各种 padding 对输入数据长度的要求：
	私钥加密：
		RSA_PKCS1_PADDING RSA_size-11
		RSA_NO_PADDING RSA_size-0
		RSA_X931_PADDING RSA_size-2
	
	公钥加密:
		RSA_PKCS1_PADDING RSA_size-11
		RSA_SSLV23_PADDING RSA_size-11
		RSA_X931_PADDING RSA_size-2
		RSA_NO_PADDING RSA_size-0
		RSA_PKCS1_OAEP_PADDING RSA_size-2 * SHA_DIGEST_LENGTH-2
	
	
	#include <openssl/rsa.h>
	#include <openssl/sha.h>
	int main() {
		RSA *r;
		int bits = 1024, ret, len, flen, padding, i;
		unsigned long e = RSA_3;
		BIGNUM *bne;
		unsigned char *key, *p;
		BIO *b;
		unsigned char from[500], to[500], out[500];

		bne = BN_new();
		ret = BN_set_word(bne,e);
		r = RSA_new();
		ret = RSA_generate_key_ex(r, bits, bne, NULL);
		if(ret!=1) {
			printf("RSA_generate_key_ex err!\n");
			return -1;
		}

		/* 私钥i2d */
		b = BIO_new(BIO_s_mem());
		ret = i2d_RSAPrivateKey_bio(b, r);
		key = malloc(1024);
		len = BIO_read(b,key,1024);
		BIO_free(b);
		b = BIO_new_file("rsa.key", "w");
		ret = i2d_RSAPrivateKey_bio(b, r);
		BIO_free(b);
		
		/* 私钥d2i */
		/* 公钥i2d */
		/* 公钥d2i */
		/* 私钥加密 */
		flen = RSA_size(r);
		printf("please select private enc padding : \n");
		printf("1.RSA_PKCS1_PADDING\n");
		printf("3.RSA_NO_PADDING\n");
		printf("5.RSA_X931_PADDING\n");
		scanf("%d",&padding);
		if(padding == RSA_PKCS1_PADDING)
			flen -= 11;
		else if(padding == RSA_X931_PADDING)
			flen -= 2;
		else if(padding == RSA_NO_PADDING)
			flen = flen;
		else {
			printf("rsa not surport !\n");
			return -1;
		}

		for(i=0; i<flen; i++)
			memset(&from[i], i, 1);

		len = RSA_private_encrypt(flen, from, to, r, padding);
		if(len <= 0) {
			printf("RSA_private_encrypt err!\n");
			return -1;
		}

		len = RSA_public_decrypt(len, to, out, r, padding);
		if(len <= 0) {
			printf("RSA_public_decrypt err!\n");
			return -1;
		}

		if(memcmp(from, out, flen)) {
			printf("err!\n");
			return -1;
		}

		/* */
		printf("please select public enc padding : \n");
		printf("1.RSA_PKCS1_PADDING\n");

		printf("2.RSA_SSLV23_PADDING\n");
		printf("3.RSA_NO_PADDING\n");
		printf("4.RSA_PKCS1_OAEP_PADDING\n");
		scanf("%d", &padding);
		flen = RSA_size(r);
		if(padding == RSA_PKCS1_PADDING)
			flen -= 11;
		else if(padding == RSA_SSLV23_PADDING)
			flen -= 11;
		else if(padding == RSA_X931_PADDING)
			flen -= 2;
		else if(padding == RSA_NO_PADDING)
			flen = flen;
		else if(padding == RSA_PKCS1_OAEP_PADDING)
			flen = flen-2 * SHA_DIGEST_LENGTH-2;
		else {
			printf("rsa not surport !\n");
			return -1;
		}

		for(i=0; i<flen; i++)
			memset(&from[i], i+1, 1);

		len = RSA_public_encrypt(flen, from, to, r, padding);
		if(len <= 0) {
			printf("RSA_public_encrypt err!\n");
			return -1;
		}

		len = RSA_private_decrypt(len, to, out, r, padding);
		if(len <= 0) {
			printf("RSA_private_decrypt err!\n");
			return -1;
		}

		if(memcmp(from, out, flen)) {
			printf("err!\n");
			return -1;
		}
		printf("test ok!\n");
		RSA_free(r);

		return 0;
}
```


```
RSA加密， 密钥长度  密文长度  明文长度  他们之间的关系。

加密的明文长度不能超过RSA密钥的长度-11，比如1024位的，明文长度不能超过117。

密文的长度总是密钥的长度的一半，比如1024位的，密文长度是64，如果是1032位，密文长度是65位。
	
1024位 = 1024bit = 128byte，128-11 = 117。(RSA_PKCS1_PADDING)
```