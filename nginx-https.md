---
title: nginx启用全站https
date: 2016-10-11 18:05:37
categories:
- linux
- 网络服务
- nginx
tags:
- linux
- 网络服务
- nginx
- https
keywords: nginx https, 全站https
---
> 
有数据统计，国外网站https流量已经超过全网的50%，相比较而言国内安全意识较差，https普及率极低；
`https`是由网景公司设计的SSL协议，用于给http加密，其实就是在http的基础上增加了`安全套接层`；
同时浏览器显示的https小绿锁会让用户觉得更放心(事实也确实如此)

<!-- more -->

~~[startssl免费ssl证书 - 教程](http://www.laozuo.org/2823.html)~~

startssl已失效，目前唯一的希望就是腾讯云了，算老马有点良心  [腾讯云ssl证书](https://console.qcloud.com/ssl)

### nginx启用全站https
> 
下载腾讯云的ssl证书，只保留nginx文件夹就行，并重命名为`server.crt server.key` 放到 `/etc/pki/nginx/`

<pre><code class="language-nginx line-numbers">vim www.conf
server {
    listen       443;
    server_name  www.zfl9.com;
    root         /usr/share/nginx/html/www.zfl9.com;
    ssl          on;
    ssl_certificate "/etc/pki/nginx/server.crt";
    ssl_certificate_key "/etc/pki/nginx/server.key";
}
server {
    listen 80;
    server_name www.zfl9.com;
    rewrite ^(.*)$ https://$server_name$1 permanent; ## 301永久重定向,强制跳转至https
}

systemctl restart nginx
ss -lnp|grep 443    # 查看https端口是否开启
</code></pre>

> 访问 `http://www.zfl9.com` 301重定向至 `https://www.zfl9.com`

![nginx全站https](/images/nginx-https.png)
