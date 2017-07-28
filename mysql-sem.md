---
title: mysql 半同步复制
date: 2016-11-30 16:25:00
categories:
- 数据库
- mysql
tags:
- 数据库
- mysql-半同步复制
keywords: mariadb, mysql, 半同步复制
---
> 介于异步复制和全同步复制之间，主库在执行完客户端提交的事务后不是立刻返回给客户端，而是等待至少一个从库接收到并写到relay log中才返回给客户端。相对于异步复制，半同步复制提高了数据的安全性，同时它也造成了一定程度的延迟，这个延迟最少是一个TCP/IP往返的时间。所以，半同步复制最好在低延时的网络中使用

<!-- more -->

### 实验环境 CentOS 7
mariadb

master：192.168.255.101
slave：192.168.255.102

已经配置好了主从复制

### 修改master配置文件
<pre><code class="language-bash line-numbers">rpl_semi_sync_master_enabled = 1
rpl_semi_sync_master_timeout = 1000
</code></pre>

### 安装master插件
<pre><code class="language-bash line-numbers">install plugin rpl_semi_sync_master soname 'semisync_master.so';
show global variables like '%rpl%';
set global rpl_semi_sync_master_enabled=1;
set global rpl_semi_sync_master_timeout=1000;
show global status like 'rpl_semi%';
</code></pre>

### 重启master-mariadb
<pre><code class="language-bash line-numbers">systemctl restart mysqld
</code></pre>

### 修改slave配置文件
<pre><code class="language-bash line-numbers">rpl_semi_sync_slave_enabled = 1
</code></pre>

### 安装slave插件
<pre><code class="language-bash line-numbers">install plugin rpl_semi_sync_slave soname 'semisync_slave.so';
set global rpl_semi_sync_slave_enabled = 1;
show global variables like '%rpl%';

stop slave;
start slave;
</code></pre>

### 重启slave-mariadb
<pre><code class="language-bash line-numbers">systemctl restart mysqld
</code></pre>

### 查看半同步状态
<pre><code class="language-bash line-numbers">show global status like 'rpl_semi%';
</code></pre>
