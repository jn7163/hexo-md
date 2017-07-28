---
title: mysql mysql-mmm高可用
date: 2016-12-01 09:48:00
categories:
- 数据库
- mysql
tags:
- 数据库
- mysql高可用
keywords: mysql, mariadb, mysql-mmm, 高可用
---
> MMM（Master-Master replication manager for MySQL）是一套支持双主故障切换和双主日常管理的脚本程序。MMM使用Perl语言开发，主要用来监控和管理MySQL Master-Master（双主）复制，虽然叫做双主复制，但是业务上同一时刻只允许对一个主进行写入，另一台备选主上提供部分读服务，以加速在主主切换时刻备选主的预热，可以说MMM这套脚本程序一方面实现了故障切换的功能，另一方面其内部附加的工具脚本也可以实现多个slave的read负载均衡

<!-- more -->

### 实验环境 CentOS 7

mariadb 已配置主从复制

master1: 192.168.255.101
slave1: 192.168.255.102
master2: 192.168.255.103
slave2: 192.168.255.104

monitor: 192.168.255.105

### 安装monitor(192.168.255.105)
<pre><code class="language-bash line-numbers">yum -y install epel-release

yum -y install mysql-mmm-monitor*
</code></pre>

### 安装agent(192.168.255.101-104)
<pre><code class="language-bash line-numbers">yum -y install epel-release

yum -y install mysql-mmm-agent*
</code></pre>

### 授权用户
<pre><code class="language-bash line-numbers">## master、slave上
grant process, super, replication client on *.* to 'mmm_agent'@'%' identified by '123456';
grant replication client on *.* to "mmm_monitor"@"%" identified by "123456";
</code></pre>

### 配置mmm_common.conf(monitor)
<pre><code class="language-bash line-numbers">vim /etc/mysql-mmm/mmm_common.conf
--- mmm_common.conf ---
active_master_role      writer

&lt;host default&gt;
    cluster_interface       eno16777736
    pid_path                /run/mysql-mmm-agent.pid
    bin_path                /usr/libexec/mysql-mmm/
    replication_user        repl ## repl用户
    replication_password    123456
    agent_user              mmm_agent ## agent用户
    agent_password          123456
&lt;/host&gt;

&lt;host db1&gt;
    ip      192.168.255.101
    mode    master
    peer    db2
&lt;/host&gt;

&lt;host db2&gt;
    ip      192.168.255.103
    mode    master
    peer    db1
&lt;/host&gt;

&lt;host db3&gt;
    ip      192.168.255.102
    mode    slave
&lt;/host&gt;

&lt;host db4&gt;
    ip      192.168.255.104
    mode    slave
&lt;/host&gt;

&lt;role writer&gt;
    hosts   db1, db2
    ips     192.168.255.200 ## writer服务器的虚拟IP
    mode    exclusive
&lt;/role&gt;

&lt;role reader&gt;
    hosts   db1, db2, db3, db4
    ips     192.168.255.201, 192.168.255.202, 192.168.255.203, 192.168.255.204 ## reader服务器的虚拟IP
    mode    balanced
&lt;/role&gt;
--- mmm_common.conf ---

## scp发送到agent
scp mmm_common.conf root@192.168.255.101:/etc/mysql-mmm/mmm_common.conf
scp mmm_common.conf root@192.168.255.102:/etc/mysql-mmm/mmm_common.conf
scp mmm_common.conf root@192.168.255.103:/etc/mysql-mmm/mmm_common.conf
scp mmm_common.conf root@192.168.255.104:/etc/mysql-mmm/mmm_common.conf
</code></pre>

### 配置mmm_mon.conf(monitor)
<pre><code class="language-bash line-numbers">vim /etc/mysql-mmm/mmm_mon.conf
--- mmm_mon.conf ---
include mmm_common.conf

&lt;monitor&gt;
    ip                  192.168.255.105 ## monitor-IP
    pid_path            /run/mysql-mmm-monitor.pid
    bin_path            /usr/libexec/mysql-mmm
    status_path         /var/lib/mysql-mmm/mmm_mond.status
    ping_ips            192.168.255.101,192.168.255.102,192.168.255.103,192.168.255.104 ## db-IP
    auto_set_online     60

    # The kill_host_bin does not exist by default, though the monitor will
    # throw a warning about it missing.  See the section 5.10 &quot;Kill Host
    # Functionality&quot; in the PDF documentation.
    #
    # kill_host_bin     /usr/libexec/mysql-mmm/monitor/kill_host
    #
&lt;/monitor&gt;

&lt;host default&gt;
    monitor_user        mmm_monitor ## monitor用户
    monitor_password    123456
&lt;/host&gt;

debug 0
--- mmm_mon.conf ---
</code></pre>

### 配置agent
<pre><code class="language-bash line-numbers">vim /etc/mysql-mmm/mmm_agent.conf
--- mmm_agent.conf ---
最后一行改为mmm_common.conf 上定义的db序号
--- mmm_agent.conf ---

## 我的是这样：
## master1
db1
## slave1
db3
## master2
db2
## slave2
db4
</code></pre>

### 启动mmm_agent
<pre><code class="language-bash line-numbers">## 192.168.255.101-104
systemctl start mysql-mmm-agent
</code></pre>

### 启动mmm_monitor
<pre><code class="language-bash line-numbers">## 192.168.255.105
systemctl start mysql-mmm-monitor
</code></pre>

### monitor监控(192.168.255.105)
<pre><code class="language-bash line-numbers">mmm_control show
db1(192.168.255.101) master/ONLINE. Roles: reader(192.168.255.201), writer(192.168.255.200)
db2(192.168.255.103) master/ONLINE. Roles: reader(192.168.255.202)
db3(192.168.255.102) slave/ONLINE. Roles: reader(192.168.255.203)
db4(192.168.255.104) slave/ONLINE. Roles: reader(192.168.255.204)

## 看到全部都UP状态
## 现在你可以手动down掉几台服务器，看看效果
</code></pre>
