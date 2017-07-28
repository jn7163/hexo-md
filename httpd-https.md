---
title: apache启用全站https
date: 2016-10-13 15:43:49
categories:
- linux
- 网络服务
- apache
tags:
- linux
- 网络服务
- apache
- https
keywords: apache https, apache全站https
---
> 
之前说了nginx开启https，不过除了nginx，还有老牌的apache，今天说说如何开启apache的https；

<!-- more -->

<pre><code class="language-bash line-numbers">yum -y install mod_ssl
</code></pre>

### 开启https
> [腾讯云ssl证书](https://console.qcloud.com/ssl)
> 下载ssl证书，放在 `/etc/pki/apache/`

<pre><code class="language-apacheconf line-numbers">cd /etc/httpd/conf.d/

--- www.conf ---
listen 443
&lt;VirtualHost *:443&gt;
    SSLEngine on
    SSLCertificateFile /etc/pki/apache/server.crt
    SSLCertificatekeyFile /etc/pki/apache/server.key
    ServerName www.zfl9.com
    DocumentRoot /var/www/html
&lt;/VirtualHost&gt;
&lt;VirtualHost *:80&gt;
    ServerName www.zfl9.com
    RewriteEngine on
    RewriteRule ^(.*)$ https://www.zfl9.com$1 [R=301,L] # 301重定向至https
&lt;/VirtualHost&gt;
</code></pre>
