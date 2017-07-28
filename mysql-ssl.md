---
title: mysql SSL复制
date: 2016-11-30 17:28:00
categories:
- 数据库
- mysql
tags:
- 数据库
- mysql-ssl复制
keywords: mysql, mariadb, ssl复制, ssl
---
> 由于mysql主从之间都是明文传输进行数据同步的，不免会带来一些安全隐患。所以我们可以通过配置ssl进行加密传输

<!-- more -->

### 实验环境 CentOS 7
mariadb
全新安装，所以数据一致

master： 192.168.255.101
slave： 192.168.255.102

### openssl生成ssl证书

#### 生成CA证书
<pre><code class="language-bash line-numbers">cd /etc/pki/CA/
(umask 077;openssl genrsa -out private/cakey.pem 2048)
openssl req -new -x509 -key private/cakey.pem -out cacert.pem -days 3650
touch index.txt serial
echo 01 > serial
</code></pre>

#### 签署master证书
<pre><code class="language-bash line-numbers">mkdir /etc/pki/mysql
cd /etc/pki/mysql
(umask 077;openssl genrsa -out master.key 2048)
openssl req -new -key master.key -out master.csr
openssl ca -in master.csr -out master.crt -days 3650
</code></pre>

#### 签署slave证书
<pre><code class="language-bash line-numbers">mkdir /etc/pki/mysql
cd /etc/pki/mysql
(umask 077;openssl genrsa -out slave.key 2048)
openssl req -new -key slave.key -out slave.csr
openssl ca -in slave.csr -out slave.crt -days 3650

## 若报错：TXT_DB error number 2
## 修改 vim /etc/pki/CA/index.txt.attr --> unique_subject = no
</code></pre>

#### 复制CA证书到mysql-ssl目录
<pre><code class="language-bash line-numbers">cp -af /etc/pki/CA/cacert.pem /etc/pki/mysql/

chown -R mysql:mysql /etc/pki/mysql
</code></pre>

#### 复制slave证书到slave主机上
<pre><code class="language-bash line-numbers">## master
-rw-r--r-- 1 mysql mysql 1253 Nov 30 03:49 cacert.pem
-rw-r--r-- 1 mysql mysql 4407 Nov 30 03:41 master.crt
-rw------- 1 mysql mysql 1679 Nov 30 03:31 master.key

## slave
-rw-r--r-- 1 mysql mysql 1253 Nov 30 00:21 cacert.pem
-rw-r--r-- 1 mysql mysql 4407 Nov 30 00:18 slave.crt
-rw------- 1 mysql mysql 1675 Nov 30 00:18 slave.key
</code></pre>

### 启用ssl认证
<pre><code class="language-bash line-numbers">vim /etc/my.cnf.d/server.cnf
--- server.cnf ---
## master
ssl-ca = /etc/pki/mysql/cacert.pem
ssl-cert = /etc/pki/mysql/master.crt
ssl-key = /etc/pki/mysql/master.key

## slave
ssl-ca = /etc/pki/mysql/cacert.pem
ssl-cert = /etc/pki/mysql/slave.crt
ssl-key = /etc/pki/mysql/slave.key
--- server.cnf ---
</code></pre>

### 重启mariadb
<pre><code class="language-bash line-numbers">systemctl restart mysqld
</code></pre>

### 授权ssl用户
<pre><code class="language-bash line-numbers">grant replication slave,replication client on *.* to 'repl'@'%' identified by '123456' require ssl;
</code></pre>

### 配置ssl复制
<pre><code class="language-bash line-numbers">change master to master_host='192.168.255.101',master_user='repl',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=313,master_ssl=1,master_ssl_ca='/etc/pki/mysql/cacert.pem',master_ssl_cert='/etc/pki/mysql/slave.crt',master_ssl_key='/etc/pki/mysql/slave.key';

start slave;
show slave status\G

## 两个OK表示成功
</code></pre>
