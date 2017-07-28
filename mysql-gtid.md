---
title: mysql GTID主从复制
date: 2016-11-30 16:00:00
categories:
- 数据库
- mysql
tags:
- 数据库
- mysql-GTID复制
keywords: mysql, mariadb, GTID, 主从复制
---
> 之前我们一直都是用 log-bin 进行主从复制的。由于需要获取log-bin的两个值，好麻烦，而且容易出错。所以新版mysql(mariadb)有了GTID复制功能

<!-- more -->

### 修改配置文件
<pre><code class="language-bash line-numbers">vim /etc/my.cnf.d/server.cnf
--- server.cnf ---
[mysqld]
server-id=1 # id须唯一
log-bin=mysql-bin
log-slave-updates=1
binlog_format=ROW
innodb_file_per_table=1
master-info-repository=TABLE
relay-log-info-repository=TABLE
sync-master-info=1
binlog-checksum=CRC32
master-verify-checksum=1
slave-sql-verify-checksum=1
binlog-rows-query-log_events=1
--- server.cnf ---

systemctl restart mysqld

## slave一样，只是修改一下id
</code></pre>

### 配置slave
<pre><code class="language-bash line-numbers">change master to master_host='192.168.255.101',master_user='backup',master_password='123456',master_use_gtid=current_pos;start slave;
</code></pre>

### 查看状态
<pre><code class="language-bash line-numbers">show slave status\G

Slave_IO_Running: Yes
Slave_SQL_Running: Yes

## 配置完成
</code></pre>
