---
title: nginx反向代理
date: 2016-10-17 19:16:33
categories:
- linux
- 网络服务
- nginx
tags:
- linux
- 网络服务
- nginx
- 反向代理
keywords: nginx反向代理
---
> 
nginx不仅仅是web服务器，也是强大的反向代理服务器。反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

<!-- more -->

## 正向代理
> 
例子：国内不能访问Google，假设我在境外有一台服务器，他可以访问Google
我们就可以通过某些代理软件(ss、vpn)连接我们的代理服务器，
然后告诉它，我要访问Google，我们的代理服务器就会帮我们去请求访问Google，
然后代理服务器再将Google返回的内容返回给我们，这样就达到了曲线救国的目的。
而对于Google来说这只是一个客户端是代理服务器ip的一个普通http请求，
Google并不知道我们的真实ip（这取决于我们的代理服务器的设置）
这种代理方式就叫做`正向代理`，这种代理方式对于Google(目标服务器)是`透明的`,因为他并不需要知道我使用了代理

## 反向代理
> 
例子：我们请求一个网站，目标网站的前端响应服务器(nginx)并没有这个请求的内容，
但是前端服务器(nginx)会去请求他的后端服务器(群)，后端服务器将网站内容返回给前端服务器，然后前端服务器再返回内容给我们
这时候前端服务器就叫做`反向代理服务器`，这种代理方式就叫`反向代理`
反向代理对于我们客户端来说是不知情的，也就是对于客户端是`透明的`，
我们并不知道我们访问网站后，服务器做了什么代理动作，也并不需要知道
反向代理的作用有: 缓解服务器压力，负载均衡，高可用

## nginx反向代理
### 配置nginx(反向代理服务器)
<pre><code class="language-nginx line-numbers">vim /etc/nginx/conf.d/m.conf
upstream httpd {
    server 127.0.0.1:8080;  # 指定上游服务器，命名为httpd
}

server {
    listen       80;
    server_name  m.zfl9.com;
    location / {
        proxy_pass          http://httpd;
        proxy_redirect      off;
        proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
</code></pre>

### 配置apache(后端服务器)
<pre><code class="language-apacheconf line-numbers">vim /etc/httpd/conf/httpd.conf
Listen 127.0.0.1:8080
LogFormat "%{X-Real-IP}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
# 把%h改为%{X-Real-IP}i     让apache可以获取到客户端的真实ip


vim /etc/httpd/conf.d/m.conf
&lt;VirtualHost *&gt;
    DocumentRoot /var/www/html
    DirectoryIndex index.html
&lt;/VirtualHost&gt;
</code></pre>

### 启动服务
<pre><code class="language-bash line-numbers">service httpd reload
service nginx reload
ss -lnp|egrep 'nginx|httpd'
</code></pre>

> 然后访问 `http://m.zfl9.com`

## nginx负载均衡
### 配置nginx
<pre><code class="language-nginx line-numbers">vim /etc/nginx/conf.d/m.conf
upstream httpd {
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
}
server {
    listen       80;
    server_name  m.zfl9.com;
    location / {
        proxy_pass          http://httpd;
        proxy_redirect      off;
        proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
</code></pre>

### 配置apache
<pre><code class="language-apacheconf line-numbers">vim /etc/httpd/conf/httpd.conf
Listen 127.0.0.1:8081


vim /etc/httpd/conf.d/m.conf
&lt;VirtualHost *:8080&gt;
    DocumentRoot /var/www/html/8080
    DirectoryIndex index.html
&lt;/VirtualHost&gt;

&lt;VirtualHost *:8081&gt;
    DocumentRoot /var/www/html/8081
    DirectoryIndex index.html
&lt;/VirtualHost&gt;


## 模拟两个后端服务器
</code></pre>

### 启动服务
<pre><code class="language-bash line-numbers">service httpd reload
service nginx reload
ss -lnp|egrep 'nginx|httpd'
</code></pre>

### 测试
> 
为了方便实验我们新建两个站点8080 8081，8080里面填写8080, 8081里面填写8081
然后浏览器访问，看第一次是不是8080(或者8081)，然后再刷新一下呢。是不是变成8081(或者8080)了?
这就是负载均衡的效果，不过这是为了做实验好看效果，生产中这些内容都是一样的!
