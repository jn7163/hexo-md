---
title: nginx rewrite详解
date: 2016-12-08 18:20:00
categories:
- linux
- 网络服务
- nginx
tags:
- linux
- 网络服务
- nginx
- rewrite
keywords: nginx, rewrite
---
> 重写URL是非常有用的一个功能，因为它可以让你提高搜索引擎阅读和索引你的网站的能力；而且在你改变了自己的网站结构后，无需要求用户修改他们的书签，无需其他网站修改它们的友情链接；它还可以提高你的网站的安全性

<!-- more -->

## 内置变量
<pre><code class="language-nginx line-numbers">$content_length     HTTP报头"Content-Length"

$content_type       HTTP报头"Content-Type"

$document_root      请求的根路径设置值

$document_uri       同$uri

$uri                请求的URI，不带参数

$host               HTTP报头"Host"

$limit_rate         对连接速率的限制

$request            HTTP报头请求信息，如"GET / HTTP/1.1"

$request_method     HTTP报头"Method"，如"GET","POST"

$remote_addr        客户端IP地址

$remote_port        客户端端口号

$remote_user        客户端(认证)用户名

$request_filename   请求的文件路径

$request_body_file  客户端请求主体信息的临时文件名

$request_uri        请求的URI，带查询参数

$query_string       同$args

$args               HTTP请求中的参数

$scheme             协议类型，http/https

$server_protocol    HTTP协议版本，"HTTP/1.0"或"HTTP/1.1"

$server_addr        服务器IP地址

$server_name        请求的服务器名

$server_port        请求的服务器端口号

$status             请求的响应状态码，如:200

$http_user_agent    客户端UA信息

$http_cookie        客户端cookie信息

$cookie_COOKIE      COOKIE变量的值
</code></pre>

## 语法规则
**rewrite写在server、location、if字段中**
<pre><code class="language-nginx line-numbers">~           正则匹配

~*          正则匹配(不区分大小写)

!~          正则匹配(取反)

!~*         正则匹配(取反，不区分大小写)

-f和!-f     判断是否为文件

-d和!-d     判断是否为目录

-e和!-e     判断文件(夹)是否存在

-x和!-x     判断文件是否可执行

set         设置变量

return      返回状态码

last        完成rewrite

break       终止匹配

redirect    返回302临时重定向

permanent   返回301永久重定向


### last break 区别 ###

last
    重新将rewrite后的地址在server标签中执行
break
    将rewrite后的地址在当前location标签中执行

所以在server标签中的last和break没区别；
而当在location标签中时: last将返回server标签重新执行一遍；break则只在当前location标签中执行一遍;
</code></pre>

## 实例
<pre><code class="language-nginx line-numbers"><script type="text/plain">## 合并多个域名，防止被搜索引擎判定为重复内容
## 使用301重定向还可以防止分散权重
server {
    listen 80;
    server_name www.zfl9.com zfl9.com;
    root /usr/share/nginx/html/www.zfl9.com;
    index index.html index.php;
    if ($host !~* ^www.zfl9.com$) {
        return 301 http://www.zfl9.com$request_uri;
    }
}

## 网页重命名，用301重定向，传递权重
server {
    listen 80;
    server_name www.zfl9.com;
    root /usr/share/nginx/html/www.zfl9.com;
    index index.html index.php;
    rewrite ^/RegExp\.html /regexp.html permanent;
    rewrite ^/python3-1\.html /python3-basic.html permanent;
}

## 全站https
server {
    listen       443 ssl;
    server_name  zfl9.com www.zfl9.com;
    if ($host !~* ^www.zfl9.com$) {
        return 301 https://www.zfl9.com$request_uri;
    }
    root         /usr/share/nginx/html/www.zfl9.com;
    index    index.html;
    ssl      on;
    ssl_certificate "/etc/pki/zfl9.com/server.crt";
    ssl_certificate_key "/etc/pki/zfl9.com/server.key";
}
server {
    listen 80;
    server_name zfl9.com www.zfl9.com;
    return 301 https://www.zfl9.com$request_uri;
}

## return http_code 返回http状态码
server {
    listen 80;
    server_name www.zfl9.com;
    root /usr/share/nginx/html/www.zfl9.com;
    index index.html;
    if ($request_uri ~* ^/admin/) {
        return 403;
    }
}
</script></code></pre>
