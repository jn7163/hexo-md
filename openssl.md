---
title: openssl和ssl证书的那些事
date: 2017-04-05 16:30:00
categories:
- linux
- 基础命令
- openssl
tags:
- linux
- 基础命令
- openssl
keywords: openssl, ssl证书, 多域名证书, 通配型证书
---
> 介绍openssl的相关用法，自签CA证书，颁发SSL证书，多域名证书，通配型证书等等

<!-- more -->

### 自建CA证书
<pre><code class="language-bash line-numbers">### 配置openssl.cnf, 不修改也行，主要就是一些默认参数
--- /etc/pki/tls/openssl.cnf ---
...
[ CA_default ]
...
default_days	= 3650			# how long to certify for
...
[ req_distinguished_name ]
countryName			= Country Name (2 letter code)
countryName_default		= CN
countryName_min			= 2
countryName_max			= 2
stateOrProvinceName		= State or Province Name (full name)
stateOrProvinceName_default	= GD
localityName			= Locality Name (eg, city)
localityName_default		= GZ
0.organizationName		= Organization Name (eg, company)
0.organizationName_default	= Otokaze
organizationalUnitName		= Organizational Unit Name (eg, section)
organizationalUnitName_default	= Otokaze
commonName			= Common Name (eg, your name or your server\'s hostname)
commonName_max			= 64
emailAddress			= Email Address
emailAddress_default    = root@zfl9.com
emailAddress_max		= 64
...
--------------------------------

### touch index.txt serial
cd /etc/pki/CA/
touch index.txt serial
echo 01 > serial

### 生成CA私钥
openssl genrsa -out private/cakey.pem 2048
chmod 600 private/cakey.pem

### 签署CA证书
openssl req -new -x509 -key private/cakey.pem -out cacert.pem
</code></pre>

### 单域名证书
<pre><code class="language-bash line-numbers">### 以nginx为例，apache同理
mkdir /etc/pki/nginx
cd /etc/pki/nginx/

### 生成私钥
openssl genrsa -out www.zfl.com.key 2048

### 生成csr证书签名请求
openssl req -new -key www.zfl.com.key -out www.zfl.com.csr  (Commone Name 填写域名)

### CA签署证书
openssl ca -in www.zfl.com.csr -out www.zfl.com.crt
或
openssl x509 -req -in www.zfl.com.csr -CA /etc/pki/CA/cacert.pem -CAkey /etc/pki/CA/private/cakey.pem -CAcreateserial -out www.zfl.com.crt
</code></pre>

### 多域名SAN/通配符CN 证书
<pre><code class="language-bash line-numbers"><script type="text/plain">### 生成私钥
openssl genrsa -out zfl.key 2048

### 生成csr证书签名请求
openssl req -new \
-key zfl.key \
-subj "/C=CN/ST=GD/L=GZ/O=Otokaze/OU=Otokaze/CN=Otokaze" \
-reqexts SAN \
-config <(cat /etc/pki/tls/openssl.cnf \
<(printf "[SAN]\nsubjectAltName=DNS:zfl.com,DNS:*.zfl.com")) \
-out zfl.csr
# 注意到subjectAltName=DNS:zfl.com,DNS:*.zfl.com
# SAN多域名证书可以是通配型的域名，也可以是单个具体域名
# *.zfl.com 不包含 zfl.com
# 你可以写任意多个域名上去

### 查看csr文件信息
openssl req -text -noout -in zfl.csr
# 可以看到包含了 Subject Alternative Names 字段

### CA签署证书
openssl ca -in zfl.csr \
-extensions SAN \
-config <(cat /etc/pki/tls/openssl.cnf \
<(printf "[SAN]\nsubjectAltName=DNS:zfl.com,DNS:*.zfl.com")) \
-out zfl.crt

### 查看crt证书信息
openssl x509 -text -noout -in zfl.crt
</script></code></pre>

### TXT_DB error number 2 错误
<pre><code class="language-bash line-numbers">rm -fr /etc/pki/CA/index.txt
touch /etc/pki/CA/index.txt
</code></pre>

### 导入CA证书
<pre><code class="language-bash line-numbers">### 我们自己颁发的ca证书是不被系统信任的，需要自己添加，否则浏览器或提示证书不安全，curl也会报错

### windows
Win+R 运行 certmgr.msc
定位到 受信任的根证书颁发机构 -> 证书 -> 右键单击 -> 所有任务 -> 导入
选择你的证书文件cacert.pem，导入即可

### linux
先备份系统默认的根证书 cp -af /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem{,.bak}

追加进去就行  cat /etc/pki/CA/cacert.pem >> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
</code></pre>

### 查看证书链
<pre><code class="language-bash line-numbers">openssl s_client -connect www.zfl9.com:443

可以看到下面的字段
Certificate chain
 0 s:/CN=www.zfl9.com
   i:/C=CN/O=TrustAsia Technologies, Inc./OU=Symantec Trust Network/OU=Domain Validated SSL/CN=TrustAsia DV SSL CA - G5
 1 s:/C=CN/O=TrustAsia Technologies, Inc./OU=Symantec Trust Network/OU=Domain Validated SSL/CN=TrustAsia DV SSL CA - G5
   i:/C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=(c) 2006 VeriSign, Inc. - For authorized use only/CN=VeriSign Class 3 Public Primary Certification Authority - G5
</code></pre>
