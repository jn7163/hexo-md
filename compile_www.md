---
title: 编译安装apache nginx php mysql mariadb
date: 2016-12-23 13:22:00
categories:
- linux
- 网络服务
- 编译安装
tags:
- linux
- 网络服务
- 编译安装
- apache
- nginx
- php
- mysql
- mariadb
keywords: lamp, lnmp, apache, nginx, php, mysql, mariadb, compile, www
---
> 以前一直都是yum安装的lamp lnmp环境，此篇记录的是我源码编译安装lamp + lnmp

<!-- more -->

### 准备源码包
<pre><code class="language-bash line-numbers">## apr-1.5.2.tar.gz
wget http://mirrors.tuna.tsinghua.edu.cn/apache//apr/apr-1.5.2.tar.gz

## apr-util-1.5.4.tar.gz
wget http://mirrors.tuna.tsinghua.edu.cn/apache//apr/apr-util-1.5.4.tar.gz

## httpd-2.4.25.tar.gz
wget http://mirrors.cnnic.cn/apache//httpd/httpd-2.4.25.tar.gz

## nginx-1.11.7.tar.gz
wget http://nginx.org/download/nginx-1.11.7.tar.gz

## mysql-boost-5.7.17.tar.gz
wget http://101.96.10.72/cdn.mysql.com/Downloads/MySQL-5.7/mysql-boost-5.7.17.tar.gz

## mariadb-10.1.20.tar.gz
wget https://mirrors.tuna.tsinghua.edu.cn/mariadb//mariadb-10.1.20/source/mariadb-10.1.20.tar.gz

## php-5.6.29.tar.gz
wget http://cn2.php.net/distributions/php-5.6.29.tar.gz

## php-7.1.0.tar.gz
wget http://cn2.php.net/distributions/php-7.1.0.tar.gz
</code></pre>

### 环境准备
<pre><code class="language-bash line-numbers">yum -y groupinstall 'Development Tools'

yum -y install bison bison-devel zlib-devel libmcrypt-devel mcrypt mhash-devel openssl-devel libxml2-devel libcurl-devel bzip2-devel readline-devel libedit-devel sqlite-devel

yum -y install cmake automake autoconf gd file bison patch mlocate flex \
diffutils zlib zlib-devel pcre pcre-devel \
libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel \
glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel \
ncurses ncurses-devel curl curl-devel libcurl libcurl-devel e2fsprogs e2fsprogs-devel \
krb5 krb5-devel openssl openssl-devel \
openldap openldap-devel nss_ldap openldap-clients openldap-servers \
openldap-devel libxslt-devel kernel-devel libtool-libs bison-devel ncurses-devel \
readline-devel gettext-devel libcap-devel libmcrypt libmcrypt-devel recode-devel

useradd apache
useradd mysql
useradd nginx

mkdir /opt/www
mkdir /opt/www/data
mkdir /opt/www/var
</code></pre>

### 安装apache

#### 安装apr
<pre><code class="language-bash line-numbers">./configure --prefix=/opt/www/apr
make && make install
</code></pre>

#### 安装apr-util
<pre><code class="language-bash line-numbers">./configure --prefix=/opt/www/apr-util --with-apr=/opt/www/apr
make && make install
</code></pre>

#### 安装httpd
<pre><code class="language-bash line-numbers">./configure --prefix=/opt/www/httpd \
--enable-so \
--enable-ssl \
--enable-cgi \
--enable-rewrite \
--with-zlib \
--with-pcre \
--with-apr=/opt/www/apr \
--with-apr-util=/opt/www/apr-util \
--enable-modules=most \
--enable-mpms-shared=all \
--with-mpm=event

make && make install
</code></pre>

### 安装nginx
<pre><code class="language-bash line-numbers">./configure --prefix=/opt/www/nginx \
--user=nginx \
--group=nginx \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-http_ssl_module

make && make install
</code></pre>

### 安装mysql
<pre><code class="language-bash line-numbers">cmake \
-DCMAKE_INSTALL_PREFIX=/opt/www/mysql \
-DMYSQL_UNIX_ADDR=/opt/www/var/mysql.sock \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DMYSQL_DATADIR=/opt/www/data \
-DMYSQL_TCP_PORT=3306 \
-DWITH_BOOST=boost

make && make install

# 建议选择utf8mb4,兼容utf8
</code></pre>

### 安装mariadb(2选1)
<pre><code class="language-bash line-numbers">cmake \
-DCMAKE_INSTALL_PREFIX=/opt/www/mysql \
-DMYSQL_DATADIR=/opt/www/data \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DWITH_SSL=system \
-DWITH_ZLIB=system \
-DWITH_LIBWRAP=0 \
-DMYSQL_UNIX_ADDR=/opt/www/var/mysql.sock \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci

make && make install

# 建议选择utf8mb4,兼容utf8
</code></pre>

### 安装php5
<pre><code class="language-bash line-numbers">./configure \
--prefix=/opt/www/php \
--with-config-file-path=/opt/www/php/etc \
--enable-inline-optimization \
--disable-debug \
--disable-rpath \
--enable-shared \
--enable-opcache \
--enable-fpm \
--with-fpm-user=nginx \
--with-fpm-group=nginx \
--with-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-gettext \
--enable-mbstring \
--with-iconv \
--with-mcrypt \
--with-mhash \
--with-openssl \
--enable-bcmath \
--enable-soap \
--with-libxml-dir \
--enable-pcntl \
--enable-shmop \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-sockets \
--with-curl \
--with-zlib \
--enable-zip \
--with-bz2 \
--with-readline \
--with-apxs2=/opt/www/httpd/bin/apxs

make && make install
</code></pre>

### 安装php7(2选1)
<pre><code class="language-bash line-numbers">./configure \
--prefix=/opt/www/php \
--with-libdir=lib64 \
--with-config-file-path=/opt/www/php/etc \
--with-mcrypt \
--with-mhash \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-gd \
--with-iconv \
--with-zlib \
--with-curl \
--with-jpeg-dir \
--with-freetype-dir \
--with-openssl \
--with-xmlrpc \
--with-bz2 \
--with-gettext \
--with-readline \
--with-recode \
--with-ldap \
--with-fpm-user=nginx \
--with-fpm-group=nginx \
--enable-fpm \
--enable-cgi \
--enable-xml \
--enable-bcmath \
--enable-inline-optimization \
--enable-mbregex \
--enable-mbstring \
--enable-ftp \
--enable-gd-native-ttf \
--enable-pcntl \
--enable-sockets \
--enable-sysvmsg \
--enable-sysvshm \
--enable-shmop \
--enable-zip \
--enable-soap \
--enable-session \
--enable-opcache \
--enable-cli \
--with-apxs2=/opt/www/httpd/bin/apxs

make && make install
</code></pre>

### 修改配置
<pre><code class="language-bash line-numbers">cd /opt/www/

chown -R mysql:mysql /opt/www/data
chown -R mysql:mysql /opt/www/var
chown -R apache:apache /opt/www/httpd/htdocs
chown -R apache:apache /opt/www/httpd/logs
chown -R nginx:nginx /opt/www/nginx/html
chown -R nginx:nginx /opt/www/nginx/logs
chown -R nginx:nginx /opt/www/nginx/*_temp
chown -R nginx:nginx /opt/www/php/var/log


## apache ##
vim /opt/www/httpd/conf/httpd.conf
--- httpd.conf ---
LoadModule php7_module        modules/libphp7.so
AddType application/x-httpd-php .php

User apache
Group apache
--- httpd.conf ---

cp -af /opt/www/httpd/bin/apachectl /etc/init.d/httpd

## mysql ##
cp -af /opt/www/mysql/support-files/my-default.cnf /etc/my.cnf    # 可选，放在/opt/www/mysql/ 目录下也可以，my.cnf
cp -af /opt/www/mysql/support-files/mysql.server /etc/init.d/mysql
/opt/www/mysql/bin/mysqld --initialize-insecure --user=mysql --datadir=/opt/www/data --basedir=/opt/www/mysql --socket=/opt/www/var/mysql.sock
# mysql 5.7.6版本以前是 bin/mysql_install_db
/opt/www/mysql/bin/mysql_secure_installation

## mysql 5.7 mysql.user表没有password字段 改为了authentication_string

## mariadb ##
cp -af /opt/www/mysql/support-files/my-small.cnf /etc/my.cnf    # 可选，放在/opt/www/mysql/ 目录下也可以，my.cnf
cp -af /opt/www/mysql/support-files/mysql.server /etc/init.d/mysql
/opt/www/mysql/scripts/mysql_install_db --user=mysql --datadir=/opt/www/data --basedir=/opt/www/mysql --socket=/opt/www/var/mysql.sock
/opt/www/mysql/bin/mysql_secure_installation

## nginx ##
vim /etc/init.d/nginx
--- nginx ---
#!/bin/bash
#chkconfig: 35 20 80
#description: nginx

. /etc/init.d/functions

nginx=/opt/www/nginx/sbin/nginx

case $1 in
start)
    $nginx
    ;;
stop)
    $nginx -s stop
    ;;
reload)
    $nginx -s reload
    ;;
restart)
    $nginx -s stop
    $nginx
    ;;
*)
    echo 'usage: start|stop|restart|reload'
    ;;
esac
--- nginx ---

chmod +x /etc/init.d/nginx

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#!/bin/bash
#chkconfig: 35 20 80
#description: nginx

第二行的chkconfig: 35 20 80 的意思
35 表示在哪个运行级别开机自启，这里是3和5两个运行级别
20 表示开机自启的优先级，通常在0-100之间，值越小优先级越高，就越先启动
80 表示关机时关闭服务的优先级，通常在0-100之间，值越小优先级越高，就越先关闭
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

## php5 ##
cp -af /opt/www/php/etc/php-fpm.conf.default /opt/www/php/etc/php-fpm.conf
cp -af /usr/local/src/php-5.6.29/php.ini-production /opt/www/php/etc/php.ini
cp -af /usr/local/src/php-5.6.29/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm

## php7 ##
cp -af /opt/www/php/etc/php-fpm.conf.default /opt/www/php/etc/php-fpm.conf
cp -af /opt/www/php/etc/php-fpm.d/www.conf.default /opt/www/php/etc/php-fpm.d/www.conf
cp -af /usr/local/src/php-7.1.0/php.ini-production /opt/www/php/etc/php.ini
cp -af /usr/local/src/php-7.1.0/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm
</code></pre>
