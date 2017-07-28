---
title: mysql 主从复制
date: 2016-11-30 15:08:00
categories:
- 数据库
- mysql
tags:
- 数据库
- mysql-主从复制
keywords: mariadb, mysql, replication, 主从复制
---
> MySQL是开源的关系型数据库系统。复制(Replication)是从一台MySQL数据库服务器（主服务器master）复制数据到另一个服务器（从服务器slave）的一个进程。此篇讲讲如何进行mysql(mariadb)主从复制

<!-- more -->

### 实验环境 CentOS 7
Master: mariadb-10.1.19     IP: 192.168.255.101
Slave: mariadb-10.1.19      IP: 192.168.255.102
主从数据库数据必须一致！这是前提

### 安装mariadb
<pre><code class="language-bash line-numbers">vim /etc/yum.repos.d/mariadb.repo
--- mariadb.repo ---
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
--- mariadb.repo ---

yum -y install MariaDB-server MariaDB
</code></pre>

### 配置master
<pre><code class="language-bash line-numbers">vim /etc/my.cnf.d/server.cnf
--- server.cnf ---
[mysqld]
log-bin=mysql-bin
log-slave-updates=1 # slave从master复制的数据会写入log-bin日志文件
server-id=1 # 必须唯一，一般取ip最后一段
--- server.cnf ---
</code></pre>

### 初始化mariadb
<pre><code class="language-bash line-numbers">systemctl start mysqld
mysql_secure_installation # 设置root密码
</code></pre>

### 配置slave
同master，只需把server-id更改即可

### 配置主从复制
#### 授权backup账户(master)
<pre><code class="language-bash line-numbers">mysql -p123456
GRANT REPLICATION SLAVE ON *.* TO 'backup'@'%' IDENTIFIED BY '123456'; # 新建backup用户
FLUSH TABLES WITH READ LOCK; # 锁表
另开一个终端，导出数据
mysqldump -uroot -p123456 -A > all.sql
scp all.sql root@192.168.255.102:

返回第一个终端
show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |     1743 |              |                  |
+------------------+----------+--------------+------------------+
出现上面的类似信息
然后解锁表
UNLOCK TABLES;
</code></pre>

#### 启用backup(slave)
<pre><code class="language-bash line-numbers">mysql -p123456 < all.sql # 导入master的数据库

mysql -p123456
change master to master_host='192.168.255.101',master_user='backup',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=1743; start slave;
## 注意修改host，user，passwd，log-file及log-pos(修改为master上的信息)

show slave status\G # 查看状态
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
都为yes则成功，否则失败！
</code></pre>

#### 测试同步
<pre><code class="language-bash line-numbers">## master上创建数据库test
create database test;

## slave上查看是否同步
show databases;

发现同步成功！好了，mariadb主从复制配置完了
</code></pre>
