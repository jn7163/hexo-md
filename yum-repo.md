---
title: 常用 yum源 整理
date: 2016-11-28 18:07:00
categories:
- linux
- 基础命令
- yum
tags:
- linux
- 基础命令
- yum
- yum源
keywords: yum, yum源, yum仓库
---
> yum仓库的出现的确方便了Linux软件包的管理，日常生产生活中一些常用的yum软件源 在这个帖子里面整理一下，以便查找

<!-- more -->

### epel

> RedHat 扩展软件yum源

<pre><code class="language-bash line-numbers">官网 --> http://mirrors.ustc.edu.cn/fedora/epel/

CentOS 6
rpm -ivh http://mirrors.ustc.edu.cn/fedora/epel/epel-release-latest-6.noarch.rpm

CentOS 7
rpm -ivh http://mirrors.ustc.edu.cn/fedora/epel/epel-release-latest-7.noarch.rpm
</code></pre>

### remi

> php/mysql 等yum源

<pre><code class="language-bash line-numbers">官网 --> http://rpms.famillecollet.com/

CentOS 6
rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm

CentOS 7
rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
</code></pre>

### webtatic

> php7 最新版 yum源

<pre><code class="language-bash line-numbers">官网 --> https://mirror.webtatic.com/yum/

CentOS 6
rpm -ivh https://mirror.webtatic.com/yum/el6/latest.rpm

CentOS 7
rpm -ivh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
</code></pre>

### nginx

> nginx yum源(epel已经有了nginx软件包，可以不用配置nginx仓库)

<pre><code class="language-bash line-numbers">官网 --> https://www.nginx.com/resources/wiki/start/topics/tutorials/install/

CentOS
--- nginx.repo ---
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
--- nginx.repo ---

RHEL
--- nginx.repo ---
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/rhel/$releasever/$basearch/
gpgcheck=0
enabled=1
--- nginx.repo ---
</code></pre>

### mysql

> mysql 社区版 yum源

<pre><code class="language-bash line-numbers">官网 --> http://repo.mysql.com/

CentOS 6
rpm -ivh http://repo.mysql.com/mysql57-community-release-el6.rpm

CentOS 7
rpm -ivh http://repo.mysql.com/mysql57-community-release-el7.rpm
</code></pre>

### mariadb

> mariadb yum源

<pre><code class="language-bash line-numbers">官网 --> https://downloads.mariadb.org/mariadb/repositories/#mirror=neusoft

CentOS 6
--- mariadb.repo ---
# MariaDB 10.1 CentOS repository list - created 2016-11-28 10:25 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos6-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
--- mariadb.repo ---

CentOS 7
--- mariadb.repo ---
# MariaDB 10.1 CentOS repository list - created 2016-11-28 10:27 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
--- mariadb.repo ---
</code></pre>

### zabbix

> zabbix yum源

<pre><code class="language-bash line-numbers">官网 --> https://repo.zabbix.com/zabbix/

CentOS 6
rpm -ivh https://repo.zabbix.com/zabbix/3.2/rhel/6/x86_64/zabbix-release-3.2-1.el6.noarch.rpm

CentOS 7
rpm -ivh https://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm
</code></pre>

### tunctl

> tunctl yum源(仅用于 RHEL/CentOS7)

<pre><code class="language-bash line-numbers">官网 --> https://pkgs.org/centos-7/nux-misc-x86_64/tunctl-1.5-12.el7.nux.x86_64.rpm.html

CentOS 7
--- tunctl.repo ---
[nux-misc]
name=Nux Misc
baseurl=http://li.nux.ro/download/nux/misc/el7/x86_64/
enabled=0
gpgcheck=1
gpgkey=http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
--- tunctl.repo ---
</code></pre>