---
title: CentOS LAMP环境搭建
date: 2016-10-14 15:33:12
categories:
- linux
- 网络服务
- lamp
tags:
- linux
- 网络服务
- lamp
- apache
- httpd
- mysql
- mariadb
- php
keywords: centos lamp, centos lamp环境
---
> 
Linux - Apache - MySQL - PHP
一组常用来搭建动态网站或者服务器的开源软件，本身都是各自独立的程序，但是因为常被放在一起使用，拥有了越来越高的兼容度，共同组成了一个强大的Web应用程序平台

<!-- more -->

## 编译安装
如果你想快速搭建lamp环境，建议选择yum安装，如果想要自己定制或者是生产环境，建议选择编译安装
[编译安装LAMP、LNMP环境](https://www.zfl9.com/compile_www.html)

## 配置yum源
### epel
> RedHat 扩展软件yum源

<pre><code class="language-bash line-numbers">官网 --> http://mirrors.ustc.edu.cn/fedora/epel/

CentOS 6
rpm -ivh http://mirrors.ustc.edu.cn/fedora/epel/epel-release-latest-6.noarch.rpm

CentOS 7
rpm -ivh http://mirrors.ustc.edu.cn/fedora/epel/epel-release-latest-7.noarch.rpm
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
mysql被甲骨文收购之前的一个分支版本，避免mysql闭源风险
这里选择mysql，你也可以选择安装mariadb，配置基本兼容
[MariaDB安装配置参考](https://www.zfl9.com/mariadb.html)

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

## 安装Apache
<pre><code class="language-bash line-numbers">yum -y install httpd httpd-tools
</code></pre>

## 安装MySQL
<pre><code class="language-bash line-numbers">yum -y install mysql mysql-server mysql-devel

service mysqld start
mysql_secure_installation   # mysql安全初始化设置
# 设置root的密码，一路回车 出现Thanks for using MySQL! 表示设置成功！
</code></pre>

## 安装PHP5
[PHP7 - 参考](https://www.zfl9.com/php7.html)
<pre><code class="language-bash line-numbers">yum -y install php php-devel php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt
</code></pre>

## 启动服务
<pre><code class="language-bash line-numbers">chkconfig httpd on
chkconfig mysqld on
service httpd start
service mysqld start
ss -lnp | egrep 'httpd|mysqld'
</code></pre>

## 测试
<pre><code class="language-bash line-numbers">vim /var/www/html/info.php
--- info.php ---
&lt;?php
phpinfo();
?&gt;
--- info.php ---

chown apache:apache /var/www/html/www/info.php
</code></pre>

浏览`http://IP/info.php`
![lamp环境搭建](images/lamp.png)
