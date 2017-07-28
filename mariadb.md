---
title: mysql更换为mariadb
date: 2016-10-18 19:35:23
categories:
- 数据库
- mysql
tags:
- 数据库
- mysql
keywords: mysql, mariadb
---
> 
MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可。开发这个分支的原因之一是：甲骨文公司收购了MySQL后，有将MySQL闭源的潜在风险，因此社区采用分支的方式来避开这个风险

<!-- more -->

## mariadb - yum源
### CentOS 6.x
<pre><code class="language-bash line-numbers">vim /etc/yum.repo.d/mariadb.repo
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2.2/centos6-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
</code></pre>

### CentOS 7.x
<pre><code class="language-bash line-numbers">vim /etc/yum.repo.d/mariadb.repo
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
</code></pre>

### RHEL 6.x
<pre><code class="language-bash line-numbers">vim /etc/yum.repo.d/mariadb.repo
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2.2/rhel6-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
</code></pre>

### RHEL 7.x
<pre><code class="language-bash line-numbers">vim /etc/yum.repo.d/mariadb.repo
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2.2/rhel7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
</code></pre>

## 安装mariadb
<pre><code class="language-bash line-numbers">yum -y install mariadb mariadb-server
</code></pre>

## 启动mariadb
<pre><code class="language-bash line-numbers">service mariadb start
mysql_secure_installation   # 安全配置
</code></pre>
