---
title: mysql 读写分离
date: 2016-12-01 08:29:00
categories:
- 数据库
- mysql
tags:
- 数据库
- mysql-读写分离
keywords: mysql, mariadb, mysql-proxy, 读写分离
---
> MySQL读写分离基本原理是让master数据库处理写操作，slave数据库处理读操作。master将写操作的变更同步到各个slave节点。MySQLProxy实际上是在客户端请求与MySQLServer之间建立了一个连接池。所有客户端请求都是发向MySQLProxy，然后经由MySQLProxy进行相应的分析，判断出是读操作还是写操作，分发至对应的MySQLServer上

<!-- more -->

### 实验环境 CentOS 7
mariadb 全新安装
已配置主从复制
master：192.168.255.101
slave：192.168.255.102
proxy：192.168.255.103

### 主从分别授权proxy
<pre><code class="language-bash line-numbers">grant all on *.* to proxy@'%' identified by '123456';
</code></pre>

### 配置mysql-proxy
<pre><code class="language-bash line-numbers">下载： http://dev.mysql.com/downloads/mysql-proxy/
64位链接：http://downloads.mysql.com/archives/get/file/mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz

cd /usr/local/src/
wget http://downloads.mysql.com/archives/get/file/mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz -O mysql-proxy.tar.gz

tar -xzvf mysql-proxy.tar.gz
mv mysql-proxy /usr/local/

chown -R root:root /usr/local/mysql-proxy

## 配置文件
vim /etc/mysql-proxy.conf
--- mysql-proxy.conf ---
[mysql-proxy]
user=mysql-proxy
plugins=admin,proxy

admin-username=admin
admin-password=123456
admin-lua-script=/usr/local/mysql-proxy/lib/mysql-proxy/lua/admin.lua

proxy-backend-addresses=192.168.255.101:3306 # 读写服务器
proxy-read-only-backend-addresses=192.168.255.102:3306 # 只读服务器，可添加多个
proxy-lua-script=/usr/local/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua

log-file=/var/log/mysql-proxy/mysql-proxy.log
pid-file=/var/run/mysql-proxy/mysql-proxy.pid
log-level=debug
daemon=true
keepalive=true
--- mysql-proxy.conf ---
chown 660 /etc/mysql-proxy.conf

## 创建mysql-proxy用户(组)
groupadd mysql-proxy
useradd -g mysql-proxy mysql-proxy

## 创建log、run文件夹
mkdir /var/log/mysql-proxy
mkdir /var/run/mysql-proxy
chown mysql-proxy:mysql-proxy /var/log/mysql-proxy
chown mysql-proxy:mysql-proxy /var/run/mysql-proxy
</code></pre>

### 修改读写分离脚本
<pre><code class="language-bash line-numbers">vim /usr/local/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua
--- rw-splitting.lua ---
if not proxy.global.config.rwsplit then
    proxy.global.config.rwsplit = {
        min_idle_connections = 1, ## 修改为1
        max_idle_connections = 8,

        is_debug = false
    }
end
--- rw-splitting.lua ---
</code></pre>

### 启动mysql-proxy
<pre><code class="language-bash line-numbers">/usr/local/mysql-proxy/bin/mysql-proxy --defaults-file=/etc/mysql-proxy.conf

ss -lnp|grep mysql-proxy ## 查看4040,4041端口是否开启
</code></pre>

### 连接admin
<pre><code class="language-bash line-numbers">mysql -h192.168.255.103 -P4041 -uadmin -p123456
MySQL [(none)]> select * from backends;
+-------------+----------------------+-------+------+------+-------------------+
| backend_ndx | address              | state | type | uuid | connected_clients |
+-------------+----------------------+-------+------+------+-------------------+
|           1 | 192.168.255.101:3306 | up    | rw   | NULL |                 0 |
|           2 | 192.168.255.102:3306 | up    | ro   | NULL |                 0 |
+-------------+----------------------+-------+------+------+-------------------+

## 多开几个终端连接4040 proxy端口，就可以看见state为up
</code></pre>

### 连接proxy
<pre><code class="language-bash line-numbers">mysql -h192.168.255.103 -P4040 -uproxy -p123456

## 为了看到读写分离效果，可以暂时停止slave复制
## 然后再写入数据，查看master上是否已写入
## 关闭master的mysql服务，再次查看数据库，看是否能读取slave上的数据
</code></pre>
