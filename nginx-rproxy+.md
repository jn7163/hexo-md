---
title: nginx负载均衡详解
date: 2016-10-18 15:22:13
categories:
- linux
- 网络服务
- nginx
tags:
- linux
- 网络服务
- nginx
- 负载均衡
- 反向代理
keywords: nginx, 负载均衡
---
> 这节内容详细介绍nginx的负载均衡相关配置 upstream是Nginx的HTTP Upstream模块，这个模块通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡

<!-- more -->

## 基本的负载均衡配置
实验环境：同一台CentOS主机中测试，运行apache和nginx
nginx作为负载均衡
apache中我配置了两个虚拟主机 配置文件分别为8080.conf和8081.conf
8080端口的虚拟主机index.html内容为` <h1>m.zfl9.com:8080</h1> `
8081端口的虚拟主机index.html内容为` <h1>m.zfl9.com:8081</h1> `

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

# upstream后接name，名字自己定义，location中可调用
</code></pre>

## 负载均衡的调度算法
### 调度算法
> 
`轮询(默认)`
每个请求按时间顺序逐一分配到不同的后端服务器，
如果后端某台服务器宕机，故障系统被自动剔除，使用户访问不受影响。
weight指定轮询权值，weight值越大，分配到的访问机率越高，主要用于后端每个服务器性能不均的情况下


> 
`ip_hash`
每个请求按访问IP的hash结果分配，这样来自同一个IP的访客固定访问一个后端服务器，有效解决了动态网页存在的session共享问题


> 
`fair`
这是比上面两个更加智能的负载均衡算法。
此种算法可以依据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。nginx本身是不支持fair的，如果需要使用这种调度算法，必须下载nginx的`upstream_fair`模块


> 
`url_hash`
此方法按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率。
nginx本身是不支持url_hash的，如果需要使用这种调度算法，必须安装nginx的`hash`软件包

### 状态参数
> 
`down`          表示当前的server暂时不参与负载均衡
`backup`        预留的备份机器. 当其他所有的非backup机器出现故障或者忙的时候，才会请求backup
`max_fails`     允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream模块定义的错误
`fail_timeout`  在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用

注意：当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是weight和backup

## 配置负载均衡
### 轮询 weight
<pre><code class="language-nginx line-numbers">upstream httpd {
    server 127.0.0.1:8080 weight=1;
    server 127.0.0.1:8081 weight=1;
}
server {
    listen       80;
    server_name  m.zfl9.com;
    location / {
        proxy_pass          https://httpd;
        proxy_redirect      off;
        proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
</code></pre>

浏览器刷新测试，发现在8080和8081之间循环

### 健康状态检查
<pre><code class="language-nginx line-numbers">upstream httpd {
        server 127.0.0.1:8080 weight=1 max_fails=2 fail_timeout=2;
        server 127.0.0.1:8081 weight=1 max_fails=2 fail_timeout=2;
}
# max_fails     允许请求失败的次数，默认为1. 当超过最大次数时，返回proxy_next_upstream模块定义的错误
# fail_timeout  在经历了max_fails次失败后，暂停服务的时间. max_fails可以和fail_timeout一起使用，进行健康状态检查


mv /etc/httpd/conf.d/8081.conf /etc/httpd/conf.d/8081.conf.bak
service httpd reload    # 模拟其中一台服务器down了
</code></pre>

浏览器刷新测试，发现只有8080了
然后我们再开启8081

<pre><code class="language-nginx line-numbers">mv /etc/httpd/conf.d/8081.conf.bak /etc/httpd/conf.d/8081.conf
service httpd reload
</code></pre>

浏览器刷新测试，发现恢复正常了
说明nginx健康状态检查生效了，但是如果全部服务器都down了怎么办呢，这时我们可以配置backup服务器

<pre><code class="language-nginx line-numbers">upstream httpd {
        server 127.0.0.1:8080 weight=1 max_fails=2 fail_timeout=2;
        server 127.0.0.1:8081 weight=1 max_fails=2 fail_timeout=2;
        server 127.0.0.1:8082 backup;
}
</code></pre>
backup服务器平时并不会启用，只有当全部服务器都down状态了才会启用backup(一般用作通知公告)

### ip_hash
每个请求按访问IP的hash结果分配，这样来自同一个IP的访客固定访问一个后端服务器，有效解决了动态网页存在的session共享问题(一般电子商务网站用的比较多)

<pre><code class="language-nginx line-numbers">upstream httpd {
        ip_hash;
        server 127.0.0.1:8080 weight=1 max_fails=2 fail_timeout=2;
        server 127.0.0.1:8081 weight=1 max_fails=2 fail_timeout=2;
        #server 127.0.0.1:8082 backup;
}
</code></pre>

当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能有backup
为什么呢？大家想啊，如果负载均衡把你分配到backup服务器上，你能访问到页面吗?

这时再刷新浏览器，发现一直都是同一台服务器，说明ip_hash配置成功
然后换个ip访问，测试发现也是固定服务器
