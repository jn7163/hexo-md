---
title: 升级＆体验-PHP7
date: 2016-10-16 14:46:50
categories:
- linux
- 网络服务
- php
tags:
- linux
- 网络服务
- php
- php7
keywords: PHP7
---
> 距离php7发布已经有段时间了，大家可能现在用的都还是php5，php7这次更新，带来了很多特性，特别是`just in time`即时编译技术，让php7的执行速度快了不少倍！

<!-- more -->

## php7 yum源
### CentOS/RHEL 6.x
<pre><code class="language-bash line-numbers">rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el6/latest.rpm
</code></pre>

### CentOS/RHEL 7.x
<pre><code class="language-bash line-numbers">rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
</code></pre>

## 卸载旧版php5
<pre><code class="language-bash line-numbers">yum -y remove php php-fpm php-common
rpm -qa | grep php  # 卸载干净
</code></pre>

## 安装php7
### apache
<pre><code class="language-bash line-numbers">yum -y install php70w php70w-opcache
</code></pre>

### nginx
<pre><code class="language-bash line-numbers">yum -y install php70w-fpm php70w-opcache
</code></pre>

### php7相关模块
<pre><code class="language-bash line-numbers">yum -y install php70w-devel php70w-mysql php70w-gd libjpeg* php70w-imap php70w-ldap php70w-odbc php70w-pear php70w-xml php70w-xmlrpc php70w-mbstring php70w-mcrypt php70w-bcmath php70w-mhash libmcrypt
</code></pre>

## 启用php7
### apache
<pre><code class="language-bash line-numbers">service httpd reload
</code></pre>

### nginx
<pre><code class="language-bash line-numbers"># 配置...
service php-fpm start
service nginx reload
</code></pre>
