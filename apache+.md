---
title: apache 配置
date: 2016-12-10 20:26:00
categories:
- linux
- 网络服务
- apache
tags:
- linux
- 网络服务
- apache
keywords: apache, httpd
---
> 本文记录了Apache一系列常用的配置，包括rewrite, cgi, 自定义错误页, gzip, 缓存设置等

<!-- more -->

> 若未作特别声明，均在/etc/httpd/conf.d/目录配置

### 虚拟主机
<pre><code class="language-apacheconf line-numbers">&lt;VirtualHost *:80&gt;
    ServerName www.zfl.com
    DocumentRoot /var/www/html
    DirectoryIndex index.html
&lt;/VirtualHost&gt;

## 绑定多个域名
ServerName zfl.com
ServerAlias www.zfl.com m.zfl.com
</code></pre>

### 目录浏览
<pre><code class="language-apacheconf line-numbers">--- /opt/www/httpd/conf/httpd.conf ---
LoadModule autoindex_module modules/mod_autoindex.so
LoadModule dir_module modules/mod_dir.so
Include conf/extra/httpd-autoindex.conf
Include conf/extra/httpd-vhosts.conf
--- /opt/www/httpd/conf/httpd.conf ---

--- /opt/www/httpd/conf/extra/httpd-vhosts.conf ---
&lt;VirtualHost *:80&gt;
    ServerName www.zfl.com
    DocumentRoot /opt/www/httpd/htdocs
    DirectoryIndex index.html index.php index.htm
    &lt;Directory /opt/www/httpd/htdocs&gt;
        Options +Indexes
    &lt;/Directory&gt;
&lt;/VirtualHost&gt;
--- /opt/www/httpd/conf/extra/httpd-vhosts.conf ---
</code></pre>

### 隐藏Apache信息
<pre><code class="language-apacheconf line-numbers">--- conf/httpd.conf ---
Include conf/extra/httpd-default.conf
--- conf/httpd.conf ---

--- conf/extra/httpd-default.conf ---
ServerTokens Prod
ServerSignature Off
--- conf/extra/httpd-default.conf ---
</code></pre>

### apache mpm模式 2.4默认event
<pre><code class="language-apacheconf line-numbers">--- httpd.conf ---
LoadModule mpm_event_module modules/mod_mpm_event.so
Include conf/extra/httpd-mpm.conf
--- httpd.conf ---
</code></pre>

### apache 添加模块(编译安装)
<pre><code class="language-apacheconf line-numbers">cd /usr/local/src/httpd/modules/metadata/
/opt/www/httpd/bin/apxs -c -i -a mod_expires.c
</code></pre>

### AllowOverride, Options
<pre><code class="language-apacheconf line-numbers">## AllowOverride ##
AllowOverride参数就是指明Apache服务器是否去找.htacess文件作为配置文件，
如果设置为None,那么服务器将忽略.htacess文件，
如果设置为All,那么所有在.htaccess文件里有的指令都将被重写

## Options ##
允许目录浏览，执行CGI程序，允许跟随符号链接等


&lt;VirtualHost *:80&gt;
    ServerName www.zfl.com
    DocumentRoot /var/www/html
    DirectoryIndex index.html
    &lt;Directory /var/www/html&gt;
        AllowOverride None
        Options None
    &lt;/Directory&gt;
&lt;/VirtualHost&gt;

## AllowOverride:
AuthConfig  允许使用所有的权限指令，他们包括AuthDBMGroupFile AuthDBMUserFile  AuthGroupFile  AuthName AuthTypeAuthUserFile和Require
FileInfo    允许使用文件控制类型的指令。它们包括AddEncoding AddLanguage  AddType  DEfaultType ErrorDocument LanguagePriority
Indexes     允许使用目录控制类型的指令。它们包括AddDescription  AddIcon  AddIconByEncoding AddIconByType  DefaultIcon  DirectoryIndex  FancyIndexing  HeaderName  IndexIgnore  IndexOptions ReadmeName
Limit       允许使用权限控制指令。它们包括Allow Deny和Order
Options     允许使用控制目录特征的指令.他们包括Options 和XBitHack

## Options:
All             准许以下除MultiViews以外所有功能
MultiViews      允许多重内容被浏览，如果你的目录下有一个叫做foo.txt的文件，那么你可以通过/foo来访问到它，这对于一个多语言内容的站点比较有用
Indexes         若该目录下无index文件，则准许显示该目录下的文件以供选择
IncludesNOEXEC  准许SSI，但不可使用#exec和#include功能
Includes        准许SSI
FollowSymLinks  在该目录中，服务器将跟踪符号链接。注意，即使服务器跟踪符号链接，它也不会改变用来匹配不同区域的路径名，如果在&lt;Local&gt;;标记内设置，该选项会被忽略
SymLinksIfOwnerMatch  在该目录中仅仅跟踪本站点内的链接
ExecCGI         在该目录下准许使用CGI

## 注意 ##
当 Options 参数前面有 +/- 号时(如：+ExecCGI)：
+   所有前面加有&quot;+&quot;号的可选项将强制覆盖当前的可选项设置
-   所有前面有&quot;-&quot;号的可选项将强制从当前可选项设置中去除
</code></pre>

### Order Deny Allow
<pre><code class="language-apacheconf line-numbers">## 用于访问控制，允许、拒绝 ##

&lt;VirtualHost *:80&gt;
    ServerName www.zfl.com
    DocumentRoot /var/www/html
    DirectoryIndex index.html
    &lt;Directory /var/www/html&gt;
        Deny from 192.168.255.103 # 禁止来自192.168.255.103的访问
        # Allow from All # 允许任何合法请求
    &lt;/Directory&gt;
&lt;/VirtualHost&gt;

## Order 定义优先级 ##

&lt;VirtualHost *:80&gt;
    ServerName www.zfl.com
    DocumentRoot /var/www/html
    DirectoryIndex index.html
    &lt;Directory /var/www/html&gt;
        Order Allow,Deny
        Allow from All
        Deny from 192.168.255.103
    &lt;/Directory&gt;
&lt;/VirtualHost&gt;

后面的优先级高，如上例：Deny优先级比Allow高
apache先匹配Allow,然后匹配Deny
当出现冲突项时，冲突部分以优先级高的为准！

所以：
apache 先匹配allow -&gt; 发现允许所有
然后匹配deny -&gt; 发现拒绝来自192.168.255.103的访问
deny优先级高，所以综合起来就是允许除了192.168.255.103外的所有访问

eg:
    Order Allow,Deny
    Deny from All
    Allow from All       # 拒绝所有访问
   
    Order Deny,Allow
    Deny from All
    Allow from All       # 允许所有访问
</code></pre>

### cgi
<pre><code class="language-apacheconf line-numbers">假设 /var/www/html/cgi-python 目录是存放cgi程序的

&lt;VirtualHost *:80&gt;
    ServerName www.zfl.com
    DocumentRoot /var/www/html
    DirectoryIndex index.html
    ScriptAlias /cgi-bin/ /var/www/html/cgi-python/
    &lt;Directory /var/www/html/cgi-python&gt;
        Options ExecCGI
        AddHandler cgi-script cgi py
    &lt;/Directory&gt;
&lt;/VirtualHost&gt;

##
cgi程序的第一行不要写成： #!/usr/bin/env python3
会报错，提示无法找到该文件! 要写绝对路径#!/usr/local/python3.6.0/bin/python3

apache 执行 cgi (python shell) 均无法打印中文字符！各种方法都试了，换nginx一切正常！
</code></pre>

### 错误页
<pre><code class="language-apacheconf line-numbers">&lt;VirtualHost *:80&gt;
    ServerName www.zfl.com
    DocumentRoot /var/www/html
    DirectoryIndex index.html
    ErrorDocument 404 /404.html
    ErrorDocument 403 /403.html
    ErrorDocument 500 /500.html
&lt;/VirtualHost&gt;
</code></pre>

### gzip
<pre><code class="language-apacheconf line-numbers">&lt;VirtualHost *:80&gt;
    ServerName www.zfl.com
    DocumentRoot /var/www/html
    DirectoryIndex index.html
    ErrorDocument 404 /404.html
    ErrorDocument 403 /403.html
    ErrorDocument 500 /500.html
    &lt;IfModule deflate_module&gt;
        SetOutputFilter DEFLATE
        SetEnvIfNoCase Request_URI .(?:gif|jpe?g|png)$ no-gzip dont-vary
        SetEnvIfNoCase Request_URI .(?:exe|t?gz|zip|bz2|sit|rar)$ no-gzip dont-vary
        SetEnvIfNoCase Request_URI .(?:pdf|doc|avi|mov|mp3|rm)$ no-gzip dont-vary
        AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css
        AddOutputFilterByType DEFLATE application/x-javascript application/x-httpd-php application/x-httpd-fastphp
    &lt;/IfModule&gt;
&lt;/VirtualHost&gt;
</code></pre>

### http缓存
<pre><code class="language-apacheconf line-numbers">&lt;VirtualHost *:80&gt;
    ServerName www.zfl.com
    DocumentRoot /var/www/html
    DirectoryIndex index.html
    ErrorDocument 404 /404.html
    ErrorDocument 403 /403.html
    ErrorDocument 500 /500.html
    &lt;IfModule mod_expires.c&gt;
        ExpiresActive on
        ExpiresDefault A864000 # 10天
        ExpiresBytype text/css &quot;access plus 14 days&quot;
        ExpiresByType text/javascript &quot;access plus 14 days&quot;
        ExpiresByType application/x-javascript &quot;access plus 14 days&quot;
        ExpiresByType application/x-shockwave-flash &quot;access plus 14 days&quot;
        ExpiresByType image/* &quot;access plus 14 days&quot;
        ExpiresByType text/html &quot;access plus 14 days&quot;
        &lt;FilesMatch &quot;.(flv|ico|pdf|avi|mov|ppt|doc|mp3|wmv|wav|jpg|gif)$&quot;&gt;
            ExpiresDefault A864000
        &lt;/FilesMatch&gt;
    &lt;/IfModule&gt;
&lt;/VirtualHost&gt;
</code></pre>

### rewrite
<pre><code class="language-apacheconf line-numbers">## 和nginx差不多，都是正则匹配替换 ##

1. 当访问的uri匹配 ^/root/.*$ 时，rewrite至/403.html 页面
&lt;VirtualHost *:80&gt;
    ServerName www.zfl.com
    DocumentRoot /var/www/html
    DirectoryIndex index.html
    RewriteEngine on
    RewriteCond %{REQUEST_URI} ^/root/.*$ [NC] # 相当于nginx 的 if
    RewriteRule ^.*$ /403.html [NC,L]  # 相当于nginx 的 rewrite
&lt;/VirtualHost&gt;

2. 重定向http至https
&lt;VirtualHost *:80&gt;
    ServerName www.zfl9.com
    RewriteEngine on
    RewriteRule ^(.*)$ https://www.zfl9.com$1 [R=301,L]
&lt;/VirtualHost&gt;

## flag
R[=code](force redirect)            强制外部重定向(默认302)
F(force URL to be forbidden)        禁用URL,返回403HTTP状态码
G(force URL to be gone)             强制URL为GONE，返回410HTTP状态码
P(force proxy)                      强制使用代理转发
L(last rule)                        表明当前规则是最后一条规则，停止分析以后规则的重写
N(next round)                       重新从第一条规则开始运行重写过程
C(chained with next rule)           与下一条规则关联
T=MIME-type(force MIME type)        强制MIME类型
NS (used only if no internal sub-request)   只用于不是内部子请求
NC(no case)                         不区分大小写
QSA(query string append)            追加请求字符串
NE(no URI escaping of output)       不在输出转义特殊字符
PT(pass through to next handler)    传递给下一个处理

## Apache变量
HTTP头
    HTTP_USER_AGENT
    HTTP_REFERER
    HTTP_COOKIE
    HTTP_FORWARDED
    HTTP_HOST
    HTTP_PROXY_CONNECTION
    HTTP_ACCEPT
    
连接与请求
    REMOTE_ADDR
    REMOTE_HOST
    REMOTE_PORT
    REMOTE_USER
    REMOTE_IDENT
    REQUEST_METHOD
    SCRIPT_FILENAME
    PATH_INFO
    QUERY_STRING
    AUTH_TYPE
    
服务器本身
    DOCUMENT_ROOT
    SERVER_ADMIN
    SERVER_NAME
    SERVER_ADDR
    SERVER_PORT
    SERVER_PROTOCOL
    SERVER_SOFTWARE
    
日期与时间
    TIME_YEAR
    TIME_MON
    TIME_DAY
    TIME_HOUR
    TIME_MIN
    TIME_SEC
    TIME_WDAY
    TIME
    
其他
    API_VERSION
    THE_REQUEST
    REQUEST_URI
    REQUEST_FILENAME
    IS_SUBREQ
    HTTPS
</code></pre>
