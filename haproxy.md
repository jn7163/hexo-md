---
title: haproxy 负载均衡
date: 2016-12-10 18:52:00
categories:
- linux
- 网络服务
- haproxy
tags:
- linux
- 网络服务
- haproxy
keywords: haproxy, 负载均衡, http代理, 健康检查
---
> 前面说了nginx, lvs负载均衡的配置，今天说说另一个软件级负载均衡器 - haproxy

<!-- more -->

### 安装haproxy
<pre><code class="language-bash line-numbers">yum -y install haproxy
</code></pre>

### 配置haproxy
<pre><code class="language-bash line-numbers">vim /etc/haproxy/haproxy.cfg

--- haproxy.cfg ---
frontend  main *:80 # 监听端口
    default_backend     http

backend http
    balance     roundrobin # 加权轮询
    server  server1 192.168.255.103:80 check
    server  server2 192.168.255.104:80 check
--- haproxy.cfg ---
</code></pre>

### 备份服务器
<pre><code class="language-bash line-numbers">backend http
    ...
    server  backup  192.168.255.104:8080 backup
    ## backup服务器一般都是在主备服务器全部无法提供服务的时候，进行信息通告的，只有当全部服务器down了才会启动！
    ...
</code></pre>

### session保持
<pre><code class="language-bash line-numbers">backend http
    ...
    cookie SERVERID insert nocache
    ...
</code></pre>

### 启用haproxy_web状态页
<pre><code class="language-bash line-numbers">listen stats
    mode http
    bind *:9999
    stats enable
    stats hide-version
    stats uri /haproxyadmin?stats
    stats realm  "hello\ proxy"
    stats auth  hapadmin:hapadmin # 认证用户:密码
    stats admin if TRUE

useradd hapadmin -s /sbin/nologin

访问http://Your_IP:9999/haproxyadmin?stats 输入用户名密码即可
</code></pre>

### 动静资源分离
<pre><code class="language-bash line-numbers">html js css images 等静态文件交给后端的static服务器处理
php cgi asp jsp 等动态页面交给后端的dynamic服务器处理

frontend  main *:80
    acl url_static       path_beg       -i /static /images /javascript /stylesheets /img /mp3 /mp4 /js /css
    acl url_static       path_end       -i .jpg .gif .png .css .js .html .xml
    acl url_dynamic      path_end       -i .php
    use_backend static          if url_static
    use_backend dynamic         if url_dynamic
    default_backend             static

backend static
    balance     roundrobin
    server      static  192.168.255.103:80 check

backend dynamic
    balance     roundrobin
    server      dynamic 192.168.255.104:80 check
</code></pre>

### 健康状态检查
<pre><code class="language-bash line-numbers">backend http
    ...
    option httpchk GET /index.html # uri检测，GET页面
    ...
</code></pre>
