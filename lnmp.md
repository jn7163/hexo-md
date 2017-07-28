---
title: CentOS LNMP环境搭建
date: 2016-10-14 16:29:20
categories:
- linux
- 网络服务
- lnmp
tags:
- linux
- 网络服务
- lnmp
- nginx
- mysql
- mariadb
- php
- php-fpm
keywords: lnmp环境, lnmp安装
---
> 
上一篇说了如何搭建LAMP环境，那除了Apache之外我们知道还有一个著名的web服务软件-Nginx，今天说说如何搭建LNMP环境 - Linux - Nginx - MySQL - PHP

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

## 安装Nginx
<pre><code class="language-bash line-numbers">yum -y install nginx
</code></pre>

## 安装MySQL
<pre><code class="language-bash line-numbers">yum -y install mysql mysql-server mysql-devel
</code></pre>

## 安装PHP5(php-fpm)
[PHP7 - 参考](https://www.zfl9.com/php7.html)
<pre><code class="language-bash line-numbers">yum -y install php-fpm php-devel php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt

# nginx不能处理php，所以需要安装php-fpm,将php交给php-fpm处理
</code></pre>

## 配置
### nginx
<pre><code class="language-nginx line-numbers">vim /etc/nginx/conf.d/www.conf
## server字段添加location
location ~* \.php$ {
        fastcgi_pass   127.0.0.1:9000;  # 9000为php-fpm监听端口
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
}

## 隐藏nginx服务器信息
http {
server_tokens off;
}

## 默认编码utf-8
http {
    charset utf-8;
}
</code></pre>

### php-fpm
<pre><code class="language-bash line-numbers">vim /etc/php-fpm.d/www.conf
user = nginx    # 改为nginx
group = nginx   # 改为nginx

chown nginx /var/log/php-fpm/

cd /var/lib/php/
mkdir session wsdlcache
chmod 775 *
chown :nginx *
</code></pre>

### mysql
<pre><code class="language-bash line-numbers">service mysqld start
mysql_secure_installation   # 安全配置
# 输入root密码，然后设置root的密码，一路回车 出现Thanks for using MySQL! 表示设置成功！
</code></pre>

## 启动服务
<pre><code class="language-bash line-numbers">chkconfig nginx on
chkconfig php-fpm on
chkconfig mysqld on
service nginx start
service php-fpm start
service mysqld start

ss -lnp | egrep 'nginx|php-fpm|mysqld'
</code></pre>

## 测试
<pre><code class="language-bash line-numbers">vim /usr/share/nginx/html/www/info.php
--- info.php ---
&lt;?php
phpinfo();
?&gt;
--- info.php ---

chown nginx:nginx /usr/share/nginx/html/www/info.php
</code></pre>

> 浏览`http://IP/info.php`

![lnmp环境搭建](images/lamp.png)
