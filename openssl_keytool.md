---
title: openssl keytool互相转换
date: 2017-04-05 17:15:00
categories:
- linux
- 基础命令
- keytool
tags:
- linux
- 基础命令
- openssl2keytool
- keytool2openssl
keywords: openssl, keytool, pem与der互相转换
---
> openssl(pem) 和 keytool(der) 互相转换

<!-- more -->

### openssl -> keytool
<pre><code class="language-bash line-numbers">1. pem转换为pkcs12
openssl pkcs12 -export -in www.zfl.com.crt -inkey www.zfl.com.key -out www.zfl.com.p12 -name www.zfl.com -passout pass:123456

2. pkcs12转换为keystore
keytool -importkeystore -srcstoretype pkcs12 -srckeystore www.zfl.com.p12 -srcstorepass 123456 -srcalias www.zfl.com -srckeypass 123456 -deststoretype jks -destkeystore www.zfl.com.jks -deststorepass 123456 -destalias www.zfl.com -destkeypass 123456
</code></pre>

### keytool -> openssl
<pre><code class="language-bash line-numbers">1. keystore转换为pkcs12
keytool -importkeystore -srcstoretype jks -srckeystore www.zfl.com.jks -srcstorepass 123456 -srcalias www.zfl.com -srckeypass 123456 -deststoretype pkcs12 -destkeystore www.zfl.com.p12 -deststorepass 123456 -destalias www.zfl.com -destkeypass 123456 -noprompt

2. pkcs12转换为pem
openssl pkcs12 -in www.zfl.com.p12 -out www.zfl.com.pem -passin pass:123456 -nodes

3. pem提取crt
openssl x509 -in www.zfl.com.pem -out www.zfl.com.crt

4. pem提取key
openssl rsa -in www.zfl.com.pem -out www.zfl.com.key
</code></pre>
