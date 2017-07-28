---
title: nginx gzip压缩配置
date: 2016-10-18 12:49:50
categories:
- linux
- 网络服务
- nginx
tags:
- linux
- 网络服务
- nginx
- gzip
keywords: nginx, gzip, 压缩
---
> gzip是GNUzip的缩写，它的主要作用就是用来减轻服务器的带宽问题。默认情况下，Nginx的gzip压缩是关闭的， gzip压缩功能就是可以让你节省不少带宽，但是会增加服务器CPU的开销，Nginx默认只对text/html进行压缩 ，如果要对html之外的内容进行压缩传输，我们需要手动来调gzip_types的值

<!-- more -->

## 开启gzip
<pre><code class="language-nginx line-numbers">vim /etc/nginx/nginx.conf
gzip on;
gzip_min_length 1k;     # 最小阀值
gzip_comp_level 4;      # 压缩比 1-9 级别越高占用cpu资源越多,推荐level_4
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png font/ttf font/otf image/svg+xml;
# 压缩文件类型,一般压缩静态文件，如图片，js，css，xml等
gzip_vary on;                   # 启用gzip压缩标识
gzip_disable "MSIE [1-6]\.";    # 禁用ie6以下的gzip压缩功能

### 其他gzip参数
gzip_buffers 4 8k;      # 缓冲
gzip_http_version 1.0;  # http协议版本 现在一般都是1.1了


### nginx作为反向代理服务器
gzip_proxied off|expired|no-cache|no-sotre|private|no_last_modified|no_etag|auth|any

# off       关闭所有的代理结果数据压缩
# expired   启用压缩，如果header中包含”Expires”头信息
# no-cache  启用压缩，如果header中包含”Cache-Control:no-cache”头信息
# no-store  启用压缩，如果header中包含”Cache-Control:no-store”头信息
# private   启用压缩，如果header中包含”Cache-Control:private”头信息
# no_last_modified  启用压缩，如果header中包含”Last_Modified”头信息
# no_etag   启用压缩，如果header中包含“ETag”头信息
# auth      启用压缩，如果header中包含“Authorization”头信息
# any       无条件压缩所有结果数据
</code></pre>
