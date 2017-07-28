---
title: mysql 多线程复制
date: 2016-11-30 17:14:00
categories:
- 数据库
- mysql
tags:
- 数据库
- mysql-多线程复制
keywords: mysql, mariadb, 多线程复制, multithreads
---
> 默认 mysql(mariadb) 复制是单线程的，这样可能无法发挥服务器的最大性能。所以在必要的时候我们需要开启mysql(mariadb)的多线程复制

<!-- more -->

### 实验环境 CentOS 7
mariadb 最新稳定版
已配置主从复制

### 修改slave配置文件
<pre><code class="language-bash line-numbers">slave-parallel-threads=4 # 启动4个线程
</code></pre>

### 重启mariadb服务
<pre><code class="language-bash line-numbers">systemctl restart mysqld
</code></pre>

### 查看进程数
<pre><code class="language-bash line-numbers">MariaDB [(none)]> show processlist;
+----+-------------+-----------+------+---------+------+-----------------------------------------------------------------------------+------------------+----------+
| Id | User        | Host      | db   | Command | Time | State                                                                       | Info             | Progress |
+----+-------------+-----------+------+---------+------+-----------------------------------------------------------------------------+------------------+----------+
|  3 | system user |           | NULL | Connect |   24 | Waiting for work from SQL thread                                            | NULL             |    0.000 |
|  4 | system user |           | NULL | Connect |   24 | Waiting for work from SQL thread                                            | NULL             |    0.000 |
|  5 | system user |           | NULL | Connect |   24 | Waiting for work from SQL thread                                            | NULL             |    0.000 |
|  6 | system user |           | NULL | Connect |   24 | Waiting for master to send event                                            | NULL             |    0.000 |
|  7 | system user |           | NULL | Connect |   24 | Waiting for work from SQL thread                                            | NULL             |    0.000 |
|  8 | system user |           | NULL | Connect |   24 | Slave has read all relay log; waiting for the slave I/O thread to update it | NULL             |    0.000 |
|  9 | root        | localhost | NULL | Query   |    0 | init                                                                        | show processlist |    0.000 |
+----+-------------+-----------+------+---------+------+-----------------------------------------------------------------------------+------------------+----------+

## 看到已经有4个复制线程启动了
</code></pre>
