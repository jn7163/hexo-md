---
title: tomcat8.5 简明笔记
date: 2017-05-06 15:30:00
categories:
- 中间件
- tomcat
tags:
- 中间件
- tomcat
keywords: tomcat8, tomcat8配置
---

> 
Tomcat是一个开放源代码、运行servlet和JSP Web应用软件的基于Java的Web应用软件容器;
随着Catalina Servlet引擎的出现，Tomcat第四版号的性能得到提升，使得它成为一个值得考虑的Servlet/JSP容器，因此目前许多WEB服务器都是采用Tomcat

<!-- more -->

## java环境配置
[SunJDK for Linux 配置](https://www.zfl9.com/jdk.html)

## tomcat8.5
<pre><code class="language-bash line-numbers"><script type="text/plain">## 下载 http://tomcat.apache.org/download-80.cgi

mkdir -p /opt/tomcat/
tar xf tomcat.tar.gz -C /opt/tomcat/

## 环境变量 /etc/profile.d/tomcat.sh
export CATALINA_HOME=/opt/tomcat
export PATH=$PATH:$CATALINA_HOME/bin

. /etc/profile

# tomcat启动、关闭
catalina.sh start|stop
or
startup.sh | shutdown.sh
</script></code></pre>

## 热部署
<pre><code class="language-bash line-numbers"><script type="text/plain"># webapps/  webapps目录
物理目录    访问url
ROOT        http://ip
zfl         http://ip/zfl/
test        http://ip/test/

# 三种方式热部署
1. 把web项目放置在webapps/  访问路径 http://ip/项目名/

2. conf/server.xml
<Engine name="Catalina" defaultHost="localhost">
  <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
    <Context debug="0" docBase="/opt/webapps/blog" path="/blog" privileged="true" reloadable="true"/>
  </Host>
</Engine>
# docBase若为相对路径是相对于webapps/   访问路径 http://ip/blog/

3. conf/Catalina/localhost/apps.xml
<?xml version="1.0" encoding="UTF-8"?>
<Context docBase="/opt/webapps/blog" reloadable="true" />
# 该方式配置时,访问路径为xml文件名
</script></code></pre>

## 基于域名的虚拟主机
<pre><code class="language-bash line-numbers"><script type="text/plain">## conf/server.xml
<Host name="www.zfl.com" appBase="/opt/webapps/www.zfl.com" unpackWARs="true" autoDeploy="true" xmlValidation="false" xmlNamespaceAware="false">
    <Context docBase="/opt/webapps/www.zfl.com/www" path="" reloadable="true" />
</Host>
<Host name="blog.zfl.com" appBase="/opt/webapps/blog.zfl.com" unpackWARs="true" autoDeploy="true" xmlValidation="false" xmlNamespaceAware="false">
    <Context docBase="/opt/webapps/blog.zfl.com/blog" path="" reloadable="true" />
</Host>

www.zfl.com     appbase目录 /opt/webapps/www.zfl.com/   web项目 www     http://www.zfl.com
blog.zfl.com    appbase目录 /opt/webapps/blog.zfl.com/  web项目 blog    http://blog.zfl.com
</script></code></pre>

## apache + tomcat
<pre><code class="language-bash line-numbers"><script type="text/plain">## 下载tomcat-connectors mod_jk
http://apache.mirror.iweb.ca/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.42-src.tar.gz

## 编译
cd /usr/local/src/mod_jk/native/
./configure --with-apxs=/opt/www/httpd/bin/apxs
make
libtool --finish /opt/www/httpd/modules/
make install

cd ../conf/
cp -af httpd-jk.conf /opt/www/httpd/conf/extra/
cp -af uriworkermap.properties /opt/www/httpd/conf/extra/
cp -af workers.properties /opt/www/httpd/conf/


## 配置Apache
--- conf/httpd.conf ---
Include conf/extra/httpd-jk.conf
AddType application/x-httpd-jsp .jsp


--- conf/extra/httpd-jk.conf ---
LoadModule jk_module modules/mod_jk.so
<IfModule jk_module>
    JkWorkersFile conf/workers.properties
    JkLogFile logs/mod_jk.log
    JkLogLevel info
    JkShmFile logs/mod_jk.shm
    # 添加这两行
    JkMount /*.jsp tomcat1
    JkMountCopy All
    #
    JkWatchdogInterval 60
    <Location /jk-status>
        JkMount jk-status
        Order deny,allow
        Deny from all
        Allow from 127.0.0.1
    </Location>
    <Location /jk-manager>
        JkMount jk-manager
        Order deny,allow
        Deny from all
        Allow from 127.0.0.1
    </Location>
</IfModule>


--- conf/workers.properties ---
worker.template.type=ajp13
worker.template.socket_connect_timeout=5000
worker.template.socket_keepalive=true
worker.template.ping_mode=A
worker.template.ping_timeout=10000
worker.template.connection_pool_minsize=0
worker.template.connection_pool_timeout=600
worker.template.reply_timeout=300000
worker.template.recovery_options=3
worker.list=tomcat1
worker.type=ajp3
worker.host=127.0.0.1
worker.port=8009


## 配置Tomcat
# docBase改为Apache网站根目录
--- conf/server.xml ---
<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
  <Context path="" docBase="/opt/www/httpd/htdocs" debug="0"/>
</Host>

## 测试
service httpd start
catalina.sh start

# test_page
--- /opt/www/httpd/htdocs/test.jsp ---
<%@page language="java" import="java.util.*" %>
Stay hungry Stay foolish !!!
Now the time is: <%out.println(new Date());%>
</script></code></pre>

## nginx + tomcat
<pre><code class="language-bash line-numbers"><script type="text/plain">## 配置nginx
--- www.conf ---
server {
    server_name www.zfl.com;
    root /opt/www/nginx/html;
    index index.jsp index.php index.html;
    location ~* \.php$ {
        fastcgi_pass    127.0.0.1:9000;
        fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include         fastcgi_params;
    }
    location ~* \.jsp$ {
        proxy_pass          http://127.0.0.1:8080;
        proxy_redirect      off;
        proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
</script></code></pre>

## 自定义错误页
<pre><code class="language-bash line-numbers"><script type="text/plain">--- conf/web.xml ---
<web-app>
...
<error-page>
<error-code>404</error-code>
<location>/404.html</location>
</error-page>
...
</web-app>
</script></code></pre>

## https相关
> 
[keytool用法](https://www.zfl9.com/keytool.html)

<pre><code class="language-bash line-numbers"><script type="text/plain">## keytool
keytool -genkey -keyalg RSA -validity 3650 -alias tomcat -keystore tomcat.jks -storepass 123456 -keypass 123456 -dname "cn=www.zfl.com,ou=otokaze,o=otokaze,l=gd,st=gz,c=cn"

## 启用https
--- conf/server.xml ---
...
<Connector port="80" protocol="HTTP/1.1"  connectionTimeout="20000"  redirectPort="443" />
...
...
<Connector port="443" protocol="org.apache.coyote.http11.Http11NioProtocol" maxThreads="150" SSLEnabled="true">
    <SSLHostConfig>
        <Certificate
            certificateKeystoreFile="/etc/pki/tomcat/tomcat.jks"
            certificateKeyAlias="tomcat"
            certificateKeystorePassword="123456"
            type="RSA" />
    </SSLHostConfig>
</Connector>
...
...
<Connector port="8009" protocol="AJP/1.3" redirectPort="443"/>
...

## 强制跳转至https
--- conf/web.xml ---
<web-app>
...
<security-constraint>
    <web-resource-collection >
        <web-resource-name >SSL</web-resource-name>
        <url-pattern>/*</url-pattern>
    </web-resource-collection>                             
    <user-data-constraint>
        <transport-guarantee>CONFIDENTIAL</transport-guarantee>
    </user-data-constraint>
</security-constraint>
...
</web-app>


## nginx 代理 tomcat (https)
nginx与client之间用https通信  nginx与tomcat之间用http通信
--- $nginx/conf.d/www.conf ---
server {
...
    location ... {
        proxy_pass http://127.0.0.1:8888;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
...
}

--- $tomcat/conf/server.xml ---
...
    <Host ...>
    ...
        <Valve className="org.apache.catalina.valves.RemoteIpValve"  
               remoteIpHeader="X-Forwarded-For"  
               protocolHeader="X-Forwarded-Proto"  
               protocolHeaderHttpsValue="https"/> 
    ...
    </Host>
...
</script></code></pre>
