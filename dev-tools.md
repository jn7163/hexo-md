---
title: centos 编译环境
date: 2017-05-07 18:18:00
categories:
- linux
- 基础命令
- 编译环境
tags:
- linux
- 基础命令
- 编译环境
keywords: centos编译环境, 编译安装python3, python3 sqlite3
---

> 
编译安装的python3.6.1，但是`pip3 install sqlite3`的时候，发现怎么都安装不了，后来仔细看了`configure`脚本；
原来sqlite3要在编译的时候启用，不能通过pip安装，编译时带上参数`--enable-loadable-sqlite-extensions`就可以了；

<!-- more -->

## 编译环境
<pre><code class="language-bash line-numbers"><script type="text/plain">## "Development Tools"
yum -y groupinstall "Development Tools"

## EPEL 源
yum -y install epel-release

## 编译环境，工具
yum -y install make cmake autoconf automake grep sed gawk curl curl-devel mysql-devel sqlite-devel wget vim zsh git subversion nmap nc tcpdump iptables iptables-services ipset NetworkManager NetworkManager-devel bind-utils telnet iproute net-tools tar zip unzip gzip  bzip2 bzip2-devel tk-devel xz xz-devel p7zip pcre pcre-devel readline readline-devel gcc gcc-c++ gtk+-devel zlib-devel openssl openssl-devel pcre pcre-devel gd kernel keyutils patch perl perl-devel perl-libs python python-devel python-libs python-pip ruby ruby-devel ruby-libs lua lua-devel kernel-headers compat* cpp glibc libgomp libstdc++-devel keyutils-libs-devel libsepol-devel libselinux-devel krb5-devel libXpm* freetype freetype-devel freetype* fontconfig fontconfig-devel libjpeg* libpng* gettext gettext-devel ncurses ncurses-devel libtool* libxml2 libxml2-devel patch policycoreutils bison texinfo acpica-tools libuuid-devel glib2 glib2-devel libaio-devel yajl-devel glibc-devel glibc-devel.i686 pixman-devel bc perl-DBI mailx xorg-x11-xauth xorg-x11-server-utils xterm --skip-broken

## yum常用参数
yum check-update    # 检查是否有更新
yum update -y       # 更新软件包

yum provides xhost  # 查询哪个rpm包提供了xhost命令
yum search telnet   # 查询rpm包

yum deplist nginx   # 查看nginx包依赖

yum clean all       # 清除dbcache headers packages metadata
yum clean dbcache
yum clean headers
yum clean metadata
yum clean packages

# 重建yum缓存
yum clean all
yum makecache
yum makecache fast      # 建立fast_mirrors缓存

rpm -qf `which ipset`   # 查询ipset属于哪个rpm包

## yum常用插件
yum -y install yum-plugin-fastestmirror
</script></code></pre>

## python3 sqlite3
<pre><code class="language-bash line-numbers"><script type="text/plain">## 安装sqlite-devel，前面其实已经安装了
yum -y install sqlite sqlite-devel

## 重新编译python3.6.1
cd /usr/local/src/Python-3.6.1/
./configure --help  # 查看帮助
./configure --prefix=/usr/local/python3 --enable-loadable-sqlite-extensions --enable-optimizations
make && make install

## pip 阿里云镜像
mkdir -p ~/.pip
vim ~/.pip/pip.conf
[global]
index-url=http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
[list]
format=columns

## 常用模块
pip3 install ipython requests beautifulsoup4 pymysql mysqlclient pexpect tqdm

## 查看sqlite3是否安装成功
Python 3.6.1 (default, May  7 2017, 16:30:56)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-11)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sqlite3
</script></code></pre>
