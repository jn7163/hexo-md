---
title: keytool的相关用法
date: 2017-04-05 16:46:00
categories:
- linux
- 基础命令
- keytool
tags:
- linux
- 基础命令
- keytool
keywords: keytool, keystore, java, jdk, ssl
---
> 除了主流的openssl制作ssl证书外，java也有一个自带的工具 - keytool，也是和证书相关的

<!-- more -->

### 生成keystore
<pre><code class="language-bash line-numbers">keytool -genkey -keyalg RSA -validity 3650 -alias tomcat -keystore tomcat.jks -storepass 123456 -keypass 123456 -dname "cn=www.zfl.com,ou=otokaze,o=otokaze,l=gd,st=gz,c=cn"

## cn=填写你的域名或者IP
</code></pre>

### 常用用法
<pre><code class="language-bash line-numbers">### 导出证书文件cer，可以导入到windows中
keytool -export -alias tomcat -keystore tomcat.jks -storepass 123456 -file ca.cer

### 导入到jdk的根证书库中
keytool -import -keystore /usr/java/jdk1.8.0_121/jre/lib/security/cacerts -storepass changeit -file ca.cer -noprompt

### 从jdk根证书库中删除某个证书
keytool -delete -keystore /usr/java/jdk1.8.0_121/jre/lib/security/cacerts -storepass changeit -alias tomcat

### 转换cer到crt格式(openssl), 三种方式都可以
keytool -printcert -rfc -file ca.cer > ca.crt
keytool -list -rfc -keystore tomcat.jks -storepass 123456
openssl x509 -inform der -in ca.cer -out ca.crt

### 导入到linux的根证书中
cat ca.crt >> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem

### 生成csr文件 (用于向第三方CA机构认证)
keytool -certReq -alias tomcat -keystore tomcat.jks -storepass 123456 -file ca.csr

### 查看keystore信息
keytool -list -keystore tomcat.jks -storepass 123456
keytool -list -v -keystore tomcat.jks -storepass 123456
keytool -list -rfc -keystore tomcat.jks -storepass 123456
keytool -printcert -rfc -file ca.cer
keytool -printcert -v -file ca.cer

### 合并两个keystore
1. 生成aaa.jks
keytool -genkey -keyalg RSA -validity 3650 -alias aaa -keystore aaa.jks -storepass 123456 -keypass 123456 -dname "cn=www.zfl.com,ou=otokaze,o=otokaze,l=gd,st=gz,c=cn"

2. 生成bbb.jks
keytool -genkey -keyalg RSA -validity 3650 -alias bbb -keystore bbb.jks -storepass 123456 -keypass 123456 -dname "cn=www.zfl.com,ou=otokaze,o=otokaze,l=gd,st=gz,c=cn"

3. 合并 aaa.jks 到 bbb.jks
keytool -importkeystore -srckeystore aaa.jks -srcstorepass 123456 -destkeystore bbb.jks -deststorepass 123456

4. 查看是否合并成功bbb.jks
keytool -list -v -keystore bbb.jks -storepass 123456

## 输出如下，看到有两个keystore，说明合并成功
Keystore type: JKS
Keystore provider: SUN

Your keystore contains 2 entries

Alias name: bbb
Creation date: Apr 6, 2017
Entry type: PrivateKeyEntry
Certificate chain length: 1
Certificate[1]:
Owner: CN=www.zfl.com, OU=otokaze, O=otokaze, L=gd, ST=gz, C=cn
Issuer: CN=www.zfl.com, OU=otokaze, O=otokaze, L=gd, ST=gz, C=cn
Serial number: d5fd8c8
Valid from: Thu Apr 06 08:37:06 CST 2017 until: Sun Apr 04 08:37:06 CST 2027
Certificate fingerprints:
	 MD5:  57:89:48:3F:A5:0D:E0:93:59:D8:11:24:F8:2C:CA:2A
	 SHA1: A5:58:64:F6:0F:E1:B2:44:1B:9B:71:D2:25:9F:D1:0E:1E:1B:9A:43
	 SHA256: CF:F3:CC:2E:DA:86:75:22:FF:B5:4B:82:07:77:E9:30:D9:E6:19:E1:62:43:C6:E5:E5:2C:8A:A2:4D:87:2A:06
	 Signature algorithm name: SHA256withRSA
	 Version: 3

Extensions: 

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 1C B0 20 EB D8 25 B4 B7   ED 00 E2 72 FB C2 60 6B  .. ..%.....r..`k
0010: 77 B0 32 32                                        w.22
]
]



*******************************************
*******************************************


Alias name: aaa
Creation date: Apr 6, 2017
Entry type: PrivateKeyEntry
Certificate chain length: 1
Certificate[1]:
Owner: CN=www.zfl.com, OU=otokaze, O=otokaze, L=gd, ST=gz, C=cn
Issuer: CN=www.zfl.com, OU=otokaze, O=otokaze, L=gd, ST=gz, C=cn
Serial number: 51014406
Valid from: Thu Apr 06 08:36:54 CST 2017 until: Sun Apr 04 08:36:54 CST 2027
Certificate fingerprints:
	 MD5:  F0:F2:BF:CD:B3:69:7C:8B:EF:34:77:9B:F3:8B:F4:C6
	 SHA1: 25:FA:AB:66:71:C0:E2:8A:E4:85:17:E0:46:D9:A0:93:37:04:E6:B8
	 SHA256: AF:01:37:38:22:A8:87:94:04:8B:79:C1:8A:A7:44:BF:82:82:C5:51:4C:8F:C8:70:15:89:4C:3B:CC:D0:63:42
	 Signature algorithm name: SHA256withRSA
	 Version: 3

Extensions: 

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: CA 57 9E 53 CA 9A DD F2   16 19 1C 3F 81 14 94 B7  .W.S.......?....
0010: C6 01 FA CA                                        ....
]
]



*******************************************
*******************************************
</code></pre>