---
title: nginx执行cgi程序-实践
date: 2016-12-03 12:16:00
categories:
- linux
- 网络服务
- nginx
tags:
- linux
- 网络服务
- nginx
- spawn-fcgi
- fcgiwrap
- cgi
- fastcgi
keywords: nginx, spawn-fcgi, fcgiwrap, fcgi
---
> nginx本身是不能执行外部程序的，nginx处理PHP也是通过PHP的fcgi管理器进行处理然后nginx将结果返回给用户。所以如果我们需要通过cgi程序来编写网站后台的话，就需要spawn-fcgi和fcgiwrap帮助nginx处理cgi

<!-- more -->

## 安装epel源
<pre><code class="language-nginx line-numbers">yum -y install epel-release
</code></pre>

## 安装fcgi
<pre><code class="language-nginx line-numbers">yum -y install fcgi fcgi-devel
</code></pre>

## 安装spawn-fcgi
<pre><code class="language-nginx line-numbers">yum -y install spawn-fcgi
</code></pre>

## 安装fcgiwrap
<pre><code class="language-nginx line-numbers">git clone git://github.com/gnosek/fcgiwrap.git
cd fcgiwrap
autoreconf -i
./configure
make
make install
</code></pre>

## 配置spawn-fcgi
<pre><code class="language-nginx line-numbers">vim /etc/sysconfig/spawn-fcgi

--- spawn-fcgi ---
FCGI_SOCKET=/var/run/fcgiwrap.socket
FCGI_PROGRAM=/usr/local/sbin/fcgiwrap
FCGI_USER=nginx
FCGI_GROUP=nginx
FCGI_EXTRA_OPTIONS="-M 0700"
OPTIONS="-u $FCGI_USER -g $FCGI_GROUP -s $FCGI_SOCKET -S $FCGI_EXTRA_OPTIONS -F 1 -P /var/run/spawn-fcgi.pid -- $FCGI_PROGRAM"

## -F 1 表示启动一个fcgiwrap进程，有需要的可以多启动几个
--- spawn-fcgi ---
</code></pre>

## 配置nginx
<pre><code class="language-nginx line-numbers">server {
    root /usr/share/nginx/html;
    index index.html;
    location /cgi-bin/ {
        fastcgi_pass unix:/var/run/fcgiwrap.socket;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}

### 配置优化
http {
    client_max_body_size 1G;
    fastcgi_connect_timeout 120;
    fastcgi_send_timeout 120;
    fastcgi_read_timeout 120;
    fastcgi_buffer_size 256k;
    fastcgi_buffers 4 256k;
    fastcgi_busy_buffers_size 256k;
}
</code></pre>

## 创建cgi-bin目录
<pre><code class="language-nginx line-numbers">cd /usr/share/nginx/html
mkdir cgi-bin
chown nginx:nginx cgi-bin

## 新建一个cgi程序
vim hello.py

--- hello.py ---
#!/usr/local/python-3.5.2/bin/python3.5
print('Content-type: text/html\r\n')
print(&lt;h1&gt;-------------&gt; Hello, world! &lt;---------------&lt;/h1&gt;)
--- hello.py ---

## 修改权限
chown nginx:nginx hello.py
chmod +x hello.py
</code></pre>

## 启动服务
<pre><code class="language-nginx line-numbers">service spawn-fcgi start
service nginx restart
</code></pre>

## 测试
![nginx-fcgi](images/fcgi.png)
