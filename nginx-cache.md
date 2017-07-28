---
title: nginx 缓存配置
date: 2016-12-08 20:19:00
categories:
- linux
- 网络服务
- nginx
tags:
- linux
- 网络服务
- nginx
- cache
- 缓存
keywords: nginx, cache, 缓存
---
> 配置nginx、设置 HTTP 头部过期时间，用 Cache-Control 中的 max-age 标记为静态文件（比如图片、 CSS 和 Javascript 文件）设置一个时间，这样用户的浏览器就会缓存这些文件。这样能节省带宽，并且在访问你的网站时会显得更快些（如果用户第二次访问你的网站，将会使用浏览器缓存中的静态文件）；这是nginx的静态资源缓存设置，还有一种就是当nginx的角色是反向代理服务器时：当用户第一次访问页面时，由于nginx的缓存中没有，会访问upstream服务器相应的文件，第二次再访问的时候，由于已经缓存在了nginx的proxy_cache中，Nginx当接收到请求之后就不会将请求传送到upstream服务器里面了；

<!-- more -->

## 静态资源缓存
<pre><code class="language-nginx line-numbers"># 图片、css、js设置过期时间 15days
--- nginx.conf ---
location ~*  \.(jpg|jpeg|png|gif|ico|css|js)$ {
   expires 15d;
}
------------------

# 其他时间单位
ms  毫秒
s   秒
m   分钟
h   小时
d   天
w   星期
M   月(30d)
y   年(365d)

eg: 1h30m 表示1小时30分钟，1y6M 表示1年6个月

# Chrome F12，可以看到http头部加了 Expires: xxx 头
</code></pre>

## upstream缓存
<pre><code class="language-nginx line-numbers">proxy_cache_path /usr/share/nginx/html/cache levels=1:2 keys_zone=cache:10m max_size=10g inactive=60m
use_temp_path=off;

server {
...
    location / {
        ...
        proxy_cache         cache;
        proxy_cache_use_stale error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_cache_valid   200 304 12h;
        proxy_cache_valid   any 10m;
        proxy_cache_key     $host$uri$is_args$args;
        add_header          Nginx-Cache "$upstream_cache_status";
    }
}


## levels=1:2       两级层次结构的目录
## keys_zone        缓存区命名
## max_size         缓存上限值
## inactive         缓存过期时长
## use_temp_path=off 将在缓存这些文件时将它们写入同一个目录下
## proxy_cache_use_stale 当出现相应的错误时,不返回错误信息,而是返回已缓存的内容
## proxy_cache_valid 指定状态的缓存过期时长
## proxy_cache_key  根据key映射成一个hash值,然后保存至本地文件
</code></pre>
