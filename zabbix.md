---
title: zabbix 监控工具
date: 2016-11-29 20:05:00
categories:
- 监控工具
- zabbix
tags:
- 监控工具
- zabbix
keywords: zabbix, 监控, 邮件报警
---
> zabbix是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。zabbix能监视各种网络参数，保证服务器系统的安全运营；并提供柔软的通知机制以让系统管理员快速定位/解决存在的各种问题。zabbix由2部分构成，zabbix server与可选组件zabbix agent。本文将详细介绍在 CentOS 7 操作系统上搭建一套Zabbix监控环境

<!-- more -->

## 安装LNMP环境(或者LAMP)
<pre><code class="language-bash line-numbers">## 1. 配置yum源
yum -y install epel-release

vim /etc/yum.repos.d/mariadb.repo
--- mariadb.repo ---
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
--- mariadb.repo ---

rpm -ivh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

yum clean && yum makecache

## 2. 快速安装lnmp
yum -y install nginx php70w-fpm php70w-opcache php70w-mysql php70w-gd libjpeg* php70w-imap php70w-ldap php70w-odbc php70w-pear php70w-xml php70w-xmlrpc php70w-mbstring php70w-mcrypt php70w-bcmath php70w-mhash libmcrypt MariaDB MariaDB-server MariaDB-devel
</code></pre>

## zabbix编译安装

### 安装编译环境
<pre><code class="language-bash line-numbers">yum -y groupinstall 'Development Tools'
</code></pre>

### 下载zabbix最新源码包
<pre><code class="language-bash line-numbers">cd /usr/local/src
wget https://sourceforge.net/projects/zabbix/files/latest/download?source=files -O zabbix-3.2.1.tar.gz
tar -xzvf zabbix-3.2.1.tar.gz
</code></pre>

### 导入mariadb数据库
<pre><code class="language-bash line-numbers">systemctl start mysqld
mysql_secure_installation # 回车，设置root密码，为了方便我设置为123456，然后一路回车即可

cd /usr/local/src/zabbix-3.2.1/database/mysql/

# 创建zabbix数据库并授权用户zabbix
mysql -p123456
create database zabbix;
GRANT ALL ON zabbix.* TO 'zabbix'@'%' IDENTIFIED BY '123456';
exit;

# 导入数据库文件
mysql -p123456 zabbix < schema.sql
mysql -p123456 zabbix < images.sql
mysql -p123456 zabbix < data.sql 
</code></pre>

### 安装相关依赖包
<pre><code class="language-bash line-numbers">yum -y install curl-devel perl-DBI libxml2-devel
</code></pre>

### 新建zabbix用户(组)
<pre><code class="language-bash line-numbers">useradd zabbix
</code></pre>

### 编译安装zabbix
<pre><code class="language-bash line-numbers">cd /usr/local/src/zabbix-3.2.1
./configure --prefix=/usr/local/zabbix-server --enable-server --enable-agent --enable-get  --with-mysql --with-libcurl --with-libxml2
make install
</code></pre>

## 配置zabbix-server
<pre><code class="language-bash line-numbers">cd /usr/local/zabbix-server/etc/
vim zabbix_server.conf
--- zabbix_server.conf ---
### 修改下面几项 ###
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=1024
PidFile=/var/run/zabbix/zabbix_server.pid
DBHost=localhost # mariadb数据库的主机
DBName=zabbix # 数据库名
DBUser=zabbix # 数据库用户
DBPassword=123456 # 用户密码
--- zabbix_server.conf ---
</code></pre>

## 创建log及run目录
<pre><code class="language-bash line-numbers">mkdir /var/log/zabbix && chown zabbix:zabbix /var/log/zabbix
mkdir /var/run/zabbix && chown zabbix:zabbix /var/run/zabbix
</code></pre>

## 创建php-session目录及修改相关权限
<pre><code class="language-bash line-numbers">mkdir /var/lib/php/session && chown :nginx /var/lib/php/session && chmod 770 /var/lib/php/session
chown nginx /var/log/php-fpm/
</code></pre>

## 配置php-fpm
<pre><code class="language-bash line-numbers">vim /etc/php-fpm.d/www.conf
--- www.conf ---
# 修改
user = nginx
group = nginx
--- www.conf ---
</code></pre>

## 修改php参数
<pre><code class="language-bash line-numbers">sed -i 's/^;date.timezone =/date.timezone = Asia\/Shanghai/' /etc/php.ini
sed -i 's/^post_max_size =.*/post_max_size = 16M/' /etc/php.ini
sed -i 's/^max_execution_time =.*/max_execution_time = 300/' /etc/php.ini
sed -i 's/^max_input_time =.*/max_input_time = 300/' /etc/php.ini

## 由于我是编译安装的php7.1.1

所以还需要修改php.ini
soap.wsdl_cache_dir="/opt/www/php7/var/lib/wsdlcache"
session.save_path = "/opt/www/php7/var/lib/session"

cd $PHP/var/
mkdir -p lib/{session,wsdlcache}
chown :nginx lib/*
chmod 770 lib/*
chown nginx:nginx log/
</code></pre>

## 配置nginx虚拟主机
<pre><code class="language-bash line-numbers">vim /etc/nginx/nginx.conf
--- nginx.conf ---
## 注释掉server字段
--- nginx.conf ---

vim /etc/nginx/conf.d/zabbix.conf
--- zabbix.conf ---
server {
    listen 80;
    root /usr/share/nginx/html/zabbix;
    index index.php;
    location ~* \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
--- zabbix.conf ---

## 复制zabbix-web源码到nginx的web目录

cp -af /usr/local/src/zabbix-3.2.1/frontends/php/ /usr/share/nginx/html/zabbix
chown -R nginx:nginx /usr/share/nginx/html/*
</code></pre>

## 启动服务
<pre><code class="language-bash line-numbers">systemctl restart mysqld
systemctl start php-fpm
systemctl start nginx
/usr/local/zabbix-server/sbin/zabbix_server

ss -lnp|grep 10051 # 查看zabbix是否启动成功
</code></pre>

## 浏览器安装web端
输入 http://192.168.255.101
![zabbix-1](images/zabbix-1.png)
![zabbix-2](images/zabbix-2.png)
![zabbix-3](images/zabbix-3.png)
![zabbix-4](images/zabbix-4.png)
![zabbix-5](images/zabbix-5.png)
![zabbix-6](images/zabbix-6.png)

默认登录用户名和密码  admin:zabbix

![zabbix-7](images/zabbix-7.png)
![zabbix-8](images/zabbix-8.png)

***安装成功！***

## 添加监控主机

### 配置agent端
<pre><code class="language-bash line-numbers">cd /usr/local/zabbix-server/etc/
vim zabbix_agentd.conf
--- zabbix_agentd.conf ---
## 修改以下几项
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=1024
Server=192.168.255.101 # zabbix服务器的ip
ServerActive=192.168.255.101 # zabbix服务器的ip
Hostname=192.168.255.101 # **特别注意，该主机名必须与 web_server 添加的主机名一致！
--- zabbix_agentd.conf ---
</code></pre>

### 启动agent端
<pre><code class="language-bash line-numbers">/usr/local/zabbix-server/sbin/zabbix_agentd

ss -lnp|grep 10050 # 查看是否运行成功
</code></pre>

### web_server添加
![zabbix-9](images/zabbix-9.png)
![zabbix-10](images/zabbix-10.png)
![zabbix-11](images/zabbix-11.png)
![zabbix-12](images/zabbix-12.png)
![zabbix-13](images/zabbix-13.png)
![zabbix-14](images/zabbix-14.png)
![zabbix-15](images/zabbix-15.png)

***添加成功！***

## 修复Lack of free swap space on xxx问题
![zabbix-16](images/zabbix-16.png)

如上图所示，提示没有swap内存空间

一般我们的服务器都会有一个swap交换分区
不过因为我用的vm虚拟机和云vps都没有配置交换分区

这种情况下，如果开启Zabbix监控，Zabbix将会报告系统缺少交换分区空间（“Lack of free swap space”）
这完全可以理解，因为按照正常的逻辑，一台物理服务器不可能不设置交换分区。
显然，这样的设计没有考虑到云主机用户，但需要适当调整监控文件配置即可解决问题。

![zabbix-17](images/zabbix-17.png)
![zabbix-18](images/zabbix-18.png)
![zabbix-19](images/zabbix-19.png)
![zabbix-20](images/zabbix-20.png)
![zabbix-21](images/zabbix-21.png)
![zabbix-22](images/zabbix-22.png)

## 添加电子邮件报警

### 创建自定义脚本(sendEmail)
<pre><code class="language-bash line-numbers">wget http://caspian.dotconf.net/menu/Software/SendEmail/sendEmail-v1.56.tar.gz
tar -xzvf sendEmail-v1.56.tar.gz
cp -af sendEmail-v1.56/sendEmail /usr/local/bin

cd /usr/local/zabbix-server/share/zabbix/alertscripts
vim sendEmail.sh
--- sendEmail.sh ---
#!/bin/bash
to=$1
subject=$2
body=$3
/usr/local/bin/sendEmail -f 123456@qq.com -t "$to" -s smtp.qq.com -u "$subject" -o message-content-type=html -o message-charset=utf8 -xu 123456@qq.com -xp 123456 -m "$body"

## 修改为你的电子邮件地址，smtp服务器，密码
--- sendEmail.sh ---

chmod +x sendEmail.sh
chown zabbix:zabbix sendEmail.sh
</code></pre>

### 测试Email脚本
<pre><code class="language-bash line-numbers">./sendEmail.sh &#x27;123456@qq.com&#x27; &#x27;test&#x27; &#x27;&lt;h1&gt;----test----&lt;/h1&gt;&#x27;
</code></pre>

![zabbix-mail](images/zabbix-mail.png)

**发送成功**

### 添加zabbix邮件报警
![zabbix-mail](images/zabbix-mail-1.png)

sendEmail.sh 参数
`{ALERT.SENDTO}`
`{ALERT.SUBJECT}`
`{ALERT.MESSAGE}`

![zabbix-mail](images/zabbix-mail-2.png)
![zabbix-mail](images/zabbix-mail-3.png)
![zabbix-mail](images/zabbix-mail-4.png)
![zabbix-mail](images/zabbix-mail-5.png)
![zabbix-mail](images/zabbix-mail-6.png)
![zabbix-mail](images/zabbix-mail-7.png)
![zabbix-mail](images/zabbix-mail-8.png)
![zabbix-mail](images/zabbix-mail-9.png)

<pre><code class="language-bash line-numbers">告警主机:&amp;nbsp;{HOSTNAME1}&lt;br/&gt;
告警时间:&amp;nbsp;{EVENT.DATE} {EVENT.TIME}&lt;br/&gt;
告警等级:&amp;nbsp;{TRIGGER.SEVERITY}&lt;br/&gt;
告警信息:&amp;nbsp;{TRIGGER.NAME}&lt;br/&gt;
告警项目:&amp;nbsp;{TRIGGER.KEY1}&lt;br/&gt;
问题详情:&amp;nbsp;{ITEM.NAME}:&amp;nbsp;{ITEM.VALUE}&lt;br/&gt;
当前状态:&amp;nbsp;{TRIGGER.STATUS}:&amp;nbsp;{ITEM.VALUE1}&lt;br/&gt;
事件ID:&amp;nbsp;{EVENT.ID}&lt;br/&gt;
</code></pre>

默认的步骤是1-1,也即是从1开始到1结束。
一旦故障发生，就是执行sendEmail.sh脚本发生报警邮件给zabbix administrator组；

假如故障持续了1个小时，它也只发送一次。
如果改成1-0，0是表示不限制,无限发送；

间隔就是默认持续时间60秒。那么一个小时，就会发送60封邮件。

如果需要短信报警的话,可以再创建一条新的动作，选择短信脚本。


zabbix 图形中文乱码解决办法：
复制windows的微软雅黑.ttc字体文件到web服务器上，替换$WebDir/zabbix/fonts/DejaVuSans.ttf

![zabbix-mail](images/zabbix-mail-10.png)
![zabbix-mail](images/zabbix-mail-11.png)
![zabbix-mail](images/zabbix-mail-12.png)
![zabbix-mail](images/zabbix-mail-13.png)

**添加邮件报警完成**

## 自定义监控
<pre><code class="language-bash line-numbers">1. 写一个脚本用于获取待监控服务的一些状态信息
2. 在zabbix客户端的配置文件zabbix_agentd.conf中添加上自定义的“UserParameter”，目的是方便zabbix调用我们上面写的那个脚本去获取待监控服务的信息
3. 在zabbix服务端使用zabbix_get测试是否能够通过第二步定义的参数去获取zabbix客户端收集的数据
4. 在zabbix服务端的web界面中新建模板，同时第一步的脚本能够获取什么信息就添加上什么监控项，“键值”设置成前面配置的“UserParameter”的值
5. 数据显示图表，这一步就很简单了，直接新建图形并选择上一步的监控项来生成动态图表即可

具体步骤：
--- etc/zabbix_agentd.conf ---
UnsafeUserParameters=1
UserParameter=lnmp_chk[*],/opt/monitor/zabbix/script/lnmp_chk.sh $1

pkill zabbix_agentd
./zabbix_agentd

mkdir script/
--- lnmp_chk.sh ---
#!/bin/bash

case $1 in
ok)
    ping_nginx=$(ps -e|grep -c nginx)
    ping_php=$(ps -e|grep -c php-fpm)
    ping_mysql=$(ps -e|grep -c mysqld)
    if test $ping_nginx -ne 0 -a $ping_php -ne 0 -a $ping_mysql -ne 0; then
        echo 0
    else
        echo 1
    fi
    ;;
mem)
    mem=$(top -n1 -b -p $(pgrep 'nginx|php-fpm|mysqld' | xargs | tr ' ' ',') | awk 'BEGIN{mem=0} /^\s*[0-9]/ {mem+=$6} END{print mem}')
    echo $mem
    ;;
cpu)
    cpu=$(top -n1 -b -p $(pgrep 'nginx|php-fpm|mysqld' | xargs | tr ' ' ',') | awk 'BEGIN{cpu=0} /^\s*[0-9]/ {cpu+=$5} END{print cpu}')
    echo $cpu
    ;;
esac

chmod +x lnmp_chk.sh
chown -R zabbix:zabbix *

测试脚本 ./lnmp_chk ok|mem|cpu

zabbix_get -s 127.0.0.1 -k lnmp_chk[ok|mem|cpu]

web添加模板、主机群、应用集、监控项、触发器、图形...
</code></pre>
