---
title: nagios 监控工具
date: 2016-12-10 20:40:00
categories:
- 监控工具
- nagios
tags:
- 监控工具
- nagios
keywords: nagios
---
> 之前介绍了zabbix的安装配置，现在说说另一个监控利器 - nagios 

<!-- more -->

### Server端

#### 前提准备
apache或nginx提供web服务

#### 下载源码包
<pre><code class="language-bash line-numbers">cd /usr/local/src/

## nagios
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.2.4.tar.gz#_ga=1.199672365.2062811684.1481328573 -O nagios-4.2.4.tar.gz

## nagios-plugins
wget https://nagios-plugins.org/download/nagios-plugins-2.1.4.tar.gz#_ga=1.199672365.2062811684.1481328573 -O nagios-plugins-2.1.4.tar.gz

## nrpe
git clone https://github.com/NagiosEnterprises/nrpe.git
</code></pre>

#### 编译安装
<pre><code class="language-bash line-numbers">useradd nagios -s /sbin/nologin
groupmems -g nagios -a apache

## nagios ##
# theme1
./configure --prefix=/opt/monitor/nagios --with-httpd-conf=/opt/www/httpd/conf/vhosts/
make all && make install && make install-init && make install-commandmode && make install-config && make install-webconf && make install-exfoliation

# theme2
./configure --prefix=/opt/monitor/nagios --with-httpd-conf=/opt/www/httpd/conf/vhosts/
make all && make install && make install-init && make install-commandmode && make install-config && make install-webconf && make install-classicui

## nagios-plugins ##
./configure --prefix=/opt/monitor/nagios --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl && make && make install

## nrpe ##
./configure --prefix=/opt/monitor/nagios --enable-ssl --enable-command-args
make all && make install && make install-plugin && make install-init && make install-daemon && make install-config

chown -R nagios:nagios /usr/local/nagios/
chmod 660 /usr/local/nagios/var/nagios.log
</code></pre>

#### 配置web界面(apache)
<pre><code class="language-bash line-numbers">vim /etc/httpd/conf.d/nagios.conf

--- nagios.conf ---
&lt;VirtualHost *:80&gt;
    ServerName www.zfl.com
    DocumentRoot /var/www/html/nagios
    Alias /nagios/ /var/www/html/nagios/
    &lt;Directory /var/www/html/nagios&gt;
        DirectoryIndex index.php
        AllowOverride None
        Options None
    &lt;/Directory&gt;
    ScriptAlias /nagios/cgi-bin/ /var/www/html/nagios/cgi-bin/
    &lt;Directory /var/www/html/nagios/cgi-bin&gt;
        AllowOverride None
        Options ExecCGI
    &lt;/Directory&gt;
&lt;/Virtualhost&gt;
--- nagios.conf ---

cd /var/www/html/
mkdir nagios
cd nagios
cp -af /usr/local/nagios/share/* .
mkdir cgi-bin
cp -af /usr/local/nagios/sbin/* cgi-bin/
chown -R apache:apache -R /var/www/html/nagios/

systemctl reload httpd
</code></pre>

#### 配置web界面(nginx)
<pre><code class="language-bash line-numbers">--- nagios.conf ---
server {
    listen 80;
    server_name nagios.zfl.com;
    root /opt/www/nginx/html/nagios;
    index index.php index.html index.htm;
    location /nagios/ {
        alias /opt/www/nginx/html/nagios/;
    }
    location /nagios/cgi-bin/ {
        alias /opt/www/nginx/html/nagios/cgi-bin/;
    }
    location ~* ^/nagios(/.*\.php)$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$1;
        include fastcgi_params;
    }
    location ~* ^/nagios(/cgi-bin/.*\.cgi)$ {
        fastcgi_pass unix:/var/run/fcgiwrap.socket;
        fastcgi_param SCRIPT_FILENAME $document_root$1;
        include fastcgi_params;
    }
    location = / {
        rewrite ^.*$ /nagios/ last;
    }
}
--- nagios.conf ---

cp -af /opt/monitor/nagios/share /opt/www/nginx/html/nagios
mkdir /opt/www/nginx/html/nagios/cgi-bin/
cp -af /opt/monitor/nagios/sbin/* /opt/www/nginx/html/nagios/cgi-bin/

chown -R nginx:nginx /opt/www/nginx/html/*
</code></pre>

#### 配置nagios
<pre><code class="language-bash line-numbers">cd /usr/local/nagios/etc/
vim cgi.cfg

--- cgi.cfg ---
use_authentication=1 --> use_authentication=0
--- cgi-bin ---
</code></pre>

#### 启动nagios
<pre><code class="language-bash line-numbers">service nagios start
</code></pre>

#### 打开web页面
`http://Your_IP/nagios`

### Client端

#### 下载源码包
<pre><code class="language-bash line-numbers">nagios-plugins  nrpe ## 不多说，看前面
</code></pre>

#### 编译安装
<pre><code class="language-bash line-numbers">useradd nagios

## 看前面
</code></pre>

#### 配置nrpe
<pre><code class="language-bash line-numbers">cd /usr/local/nagios/
mkdir etc
cp -af /usr/local/src/nrpe/sample-config/nrpe.cfg etc/
chown -R nagios:nagios /usr/local/nagios/

vim nrpe.cfg

--- nrpe.cfg ---
allowed_hosts=127.0.0.1,192.168.255.103 # 添加server的ip
dont_blame_nrpe=1
--- nrpe.cfg ---

## 启动nrpe
/usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
</code></pre>

#### 添加client(在server上操作)
<pre><code class="language-bash line-numbers">cd /usr/local/nagios/etc/
vim nagios.cfg

--- nagios.cfg ---
cfg_dir=/usr/local/nagios/etc/services ## client配置文件夹
--- nagios.cfg ---

mkdir /usr/local/nagios/etc/services
cd /usr/local/nagios/etc/services

--- 192.168.255.102.cfg ---
define host{
    use linux-server
    host_name 192.168.255.102
    alias 192.168.255.102
    address 192.168.255.102
}
define service{
    use generic-service
    host_name 192.168.255.102
    service_description PING
    check_command check_ping!100.0,20%!500.0,60%
}
define service{
    use generic-service
    host_name 192.168.255.102
    service_description Root Partition
    check_command check_local_disk!20%!10%!/
}
define service{
    use generic-service
    host_name 192.168.255.102
    service_description Current Users
    check_command check_local_users!20!50
}
define service{
    use generic-service
    host_name 192.168.255.102
    service_description Total Processes
    check_command check_local_procs!250!400!RSZDT
}
define service{
    use generic-service
    host_name 192.168.255.102
    service_description Current Load
    check_command check_local_load!5.0,4.0,3.0!10.0,6.0,4.0
}
define service{
    use generic-service
    host_name 192.168.255.102
    service_description SSH
    check_command check_ssh
    notifications_enabled 0
}
--- 192.168.255.102.cfg ---

/usr/local/nagios/etc/objects/commands.cfg

--- commands.cfg ---
...
define command{
command_name check_nrpe
command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
--- commands.cfg ---

service nagios restart


## server上测试
./check_nrpe -H 192.168.255.102 显示版本号则没问题
</code></pre>

#### sendEmail替换mailx邮件报警
<pre><code class="language-bash line-numbers">修改为你的Email地址，密码，smtp服务器地址
--- commands.cfg ---
# 'notify-host-by-email' command definition
define command{
    command_name notify-host-by-email
    command_line /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | /usr/local/bin/sendEmail -f postmaster@zfl9.com -t $CONTACTEMAIL$ -s smtp.zfl9.com -xu postmaster@zfl9.com -xp 123 -u "$NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **"
}

# 'notify-service-by-email' command definition
define command{
    command_name notify-service-by-email
    command_line /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | /usr/local/bin/sendEmail -f postmaster@zfl9.com -t $CONTACTEMAIL$ -s smtp.zfl9.com -xu postmaster@zfl9.com -xp 123 -u "$NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **"
}
--- commands.cfg ---

修改为你的邮箱地址
--- contacts.cfg ---
define contact{
    contact_name    nagiosadmin
    use             generic-contact
    alias           Nagios Admin
    email           postmaster@zfl9.com
}
define contactgroup{
    contactgroup_name   admins
    alias               Nagios Administrators
    members             nagiosadmin
}
--- contacts.cfg ---

--- nagios.cfg ---
enable_notifications=1
enable_event_handlers=1
--- nagios.cfg ---

下载sendEmail 解压出来把sendEmail文件放在/usr/local/bin/

然后随便关闭一个服务。过几分钟就有邮件了...
</code></pre>
