---
title: cacti 监控工具
date: 2017-04-06 11:08:00
categories:
- 监控工具
- cacti
tags:
- 监控工具
- cacti
keywords: cacti, 监控工具
---
> 监控三大利器，zabbix，nagios，cacti。今天介绍如何在CentOS上安装cacti

<!-- more -->

### 安装软件包
<pre><code class="language-bash line-numbers">## lamp或者lnmp环境都可以，这里就不多说了，我安装的是lnmp,php7

yum -y install php70w-snmp net-snmp net-snmp-devel net-snmp-utils net-snmp-libs rrdtool
</code></pre>

### 相关配置修改
<pre><code class="language-bash line-numbers">### 启动snmpd
systemctl enable snmpd
systemctl start snmpd
# 测试snmp
snmpwalk -v 1 -c public localhost .1.3.6.1.2.1.1.1.0

### mariadb
# 创建cacti数据库
mysql -uroot -p123456 -e 'create database cacti;'

# 导入cacti数据库
mysql -uroot -p123456 cacti < /root/cacti-1.1.2/cacti.sql

# 导入time_zone_name表
mysql_tzinfo_to_sql /usr/share/zoneinfo/ | mysql -p123456 mysql

# 授权cactiuser
grant all on cacti.* to cactiuser@'%' identified by '123456';
grant select on mysql.time_zone_name to cactiuser@'%';
flush privileges;

# 修改my.cnf
--- my.cnf ---
[client]
default-character-set = utf8mb4
[mysql]
default-character-set = utf8mb4
[mysqld]
init-connect = 'SET NAMES utf8mb4'
collation_server = utf8mb4_unicode_ci
character_set_server = utf8mb4
skip-character-set-client-handshake
max_heap_table_size = 150M
max_allowed_packet = 16777216
tmp_table_size = 64M
join_buffer_size = 64M
innodb_buffer_pool_size = 731M
innodb_doublewrite = OFF
innodb_additional_mem_pool_size = 80M
innodb_flush_log_at_timeout = 3
innodb_read_io_threads = 32
innodb_write_io_threads = 16
--- my.cnf ---

### 修改config.php
vim include/config.php
$database_type     = 'mysql';
$database_default  = 'cacti';
$database_hostname = 'localhost';
$database_username = 'cactiuser';
$database_password = '123456';
$database_port     = '3306';
$database_ssl      = false;

### 新建计划任务
echo "*/5 * * * * nginx php /usr/share/nginx/html/cacti/poller.php > /dev/null 2>&1" >> /etc/crontab

systemctl enable crond
systemctl restart crond

### 配置php
vim /etc/php.ini
--------------
date.timezone = Asia/Shanghai
--------------

vim /etc/php-fpm.d/www.conf
--------------
user = nginx
group = nginx
--------------

chown nginx /var/log/php-fpm
cd /var/lib/php/
mkdir session wsdlcache
chown :nginx *
chmod 775 *

### 配置nginx
cp -af /root/cacti-1.1.2 /usr/share/nginx/html/cacti
chown -R nginx:nginx /usr/share/nginx/html/

--- /etc/nginx/conf.d/cacti.conf ---
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.php;
    location ~* \.php$ {
        fastcgi_pass    127.0.0.1:9000;
        fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name; 
        include         fastcgi_params; 
    }
}
------------------------------------


### 运行服务
systemctl enable nginx php-fpm mariadb snmpd crond
systemctl start nginx php-fpm mariadb snmpd crond
</code></pre>

### 安装cacti
<pre><code class="language-bash line-numbers">浏览器打开 http://192.168.255.101/cacti/

默认账号密码 admin:admin

按照提示一步一步安装，最后提示修改密码，安装完成
</code></pre>
