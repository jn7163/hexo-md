---
title: mysql 双主复制
date: 2016-11-30 15:22:00
categories:
- 数据库
- mysql
tags:
- 数据库
- mysql-双主复制
keywords: mysql, mariadb, 双主复制
---
> 之前讲了mariadb主从复制，现在聊聊mariadb双主复制(主主复制)，也就是互为主备，实现双向同步

<!-- more -->

### 实验环境 CentOS 7
mariadb 版本号一致，全新安装，所以数据库一致。

master1: 192.168.255.101
master2: 192.168.255.102

[主从复制详见上一篇](https://www.zfl9.com/mysql-replication.html)

### 修改配置文件
<pre><code class="language-bash line-numbers">vim /etc/my.cnf.d/server.cnf
--- server.cnf ---
[mysqld]
log-bin=mysql-bin
log-slave-updates=1 # slave从master复制的数据会写入log-bin日志文件
server-id=1 # 必须唯一，一般取ip最后一段
--- server.cnf ---

另一个只需把server-id修改即可
</code></pre>

### 互相授权用户
<pre><code class="language-bash line-numbers">## master1 与 master2 一样的操作

mysql -p123456
GRANT REPLICATION SLAVE ON *.* TO 'backup'@'%' IDENTIFIED BY '123456'; # 新建backup用户
show master status; # 记录相关值
</code></pre>

### 互告log-bin信息
<pre><code class="language-bash line-numbers">## master1
mysql -p123456
change master to master_host='192.168.255.102',master_user='backup',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=1743; start slave;

## master2
mysql -p123456
change master to master_host='192.168.255.101',master_user='backup',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=1743; start slave;
</code></pre>

### 查看状态
<pre><code class="language-bash line-numbers">## master1
show slave status\G

Slave_IO_Running: Yes
Slave_SQL_Running: Yes


## master2
show slave status\G

Slave_IO_Running: Yes
Slave_SQL_Running: Yes

必须全部为YES状态！
</code></pre>
