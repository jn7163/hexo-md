---
title: weblogic 12c 简明笔记
date: 2017-05-06 16:58:00
categories:
- 中间件
- weblogic
tags:
- 中间件
- weblogic
keywords: weblogic 12c 配置
---

> 
WebLogic是Oracle的主要产品之一，系购并得来，商业市场上主要的Java(J2EE)应用服务器软件之一;
是世界上第一个成功商业化的J2EE应用服务器，目前已推出到12c(12.1.1)版本

<!-- more -->

## java环境配置
[SunJDK for Linux 配置](https://www.zfl9.com/jdk.html)

## 解决启动缓慢问题
<pre><code class="language-bash line-numbers"><script type="text/plain">## 由于JDK的bug, 导致weblogic在linux下启动异常缓慢

vim $JAVA_HOME/jre/lib/security/java.security
"securerandom.source=file:/dev/./urandom"
</script></code></pre>

## weblogic 12c
<pre><code class="language-bash line-numbers"><script type="text/plain">## ssh_x11_forward 安装 weblogic
groupadd oinstall
useradd -g oinstall weblogic
echo 123456 | passwd --stdin weblogic
mkdir -p /opt/weblogic && chown weblogic:oinstall /opt/weblogic && chmod 775 /opt/weblogic

ssh -Y weblogic@127.0.0.1 java -jar /home/weblogic/wls.jar  # 安装weblogic
</script></code></pre>

## base_domain域
<pre><code class="language-bash line-numbers"><script type="text/plain">## config.sh 新建域
/opt/weblogic/oracle_common/common/bin/config.sh

## 开发模式与生产模式
开发模式启动自动部署，生产模式则关闭，生产模式启动时需要输入账号密码

## 修改config.xml监听地址
/opt/weblogic/user_projects/domains/base_domain/config/config.xml
"<listen-address>0.0.0.0</listen-address>"

## 启动admin_server
nohup /opt/weblogic/user_projects/domains/base_domain/startWeblogic.sh &>> /var/log/wls_admin.log &

访问http://ip:7001/console/  登录控制台，语言显示看浏览器的首选语言，设置为简中就行

## 创建boot.properties文件，免去输入帐号密码(生产模式) adminserver,managedserver同样的方式配置
cd /opt/weblogic/user_projects/domains/base_domain/servers/AdminServer/
mkdir security && cd security
--- boot.properties ---
username=weblogic
password=weblogic

## 配置nodemanager (每台Machine有且只有一个nodemanager,用于console远程控制ManagedServer)
cd /opt/weblogic/oracle_common/common/nodemanager/
cp -af /opt/weblogic/wlserver/server/bin/startNodeManager.sh .
--- startNodeManager.sh ---
NODEMGR_HOME="/opt/weblogic/oracle_common/common/nodemanager"
export NODEMGR_HOME

--- nodemanager.properties ---
ListenAddress=0.0.0.0
ListenPort=5556
SecureListener=false

. /opt/weblogic/wlserver/server/bin/setWLSEnv.sh

java weblogic.WLST
connect('weblogic', 'weblogic', 't3://ip:7001')
nmEnroll('/opt/weblogic/user_projects/domains/base_domain', '/opt/weblogic/oracle_common/common/nodemanager')

nohup /opt/weblogic/oracle_common/common/nodemanager/startNodeManager.sh &>> /var/log/wls_nm.log &

## console 创建machine,managedserver,并把server加入至对应machine中

## 手动启动ManagedServer(也可以通过nodemanager在console启动)
nohup /opt/weblogic/user_projects/domains/base_domain/bin/startManagedServer.sh ManagedServer_NAME &>> /var/log/wls_xx.log
</script></code></pre>

## 错误页 https跳转
同tomcat    [自定义错误页](https://www.zfl9.com/tomcat.html#自定义错误页)   [http跳转https](https://www.zfl9.com/tomcat.html#https相关)

## weblogic.xml
<pre><code class="language-bash line-numbers"><script type="text/plain"># 默认web项目的访问路径不是根目录，通过weblogic.xml可以指定
--- $webapp/WEB-INF/weblogic.xml ---
<?xml version="1.0" encoding="UTF-8"?>
<weblogic-web-app>
    <context-root>/</context-root>
</weblogic-web-app>
</script></code></pre>

## 基于域名的虚拟主机
> 
console -> 环境 -> 虚拟主机 -> 添加 -> 填写域名 -> 设置目标(Server)
console -> 部署 -> ... -> 目标选择虚拟主机 ...

## https 相关
<pre><code class="language-bash line-numbers"><script type="text/plain">## keytool 创建证书 同tomcat

## console配置
console -> 环境 -> 服务器 -> 选择你要启用https的server -> 启用ssl监听443端口 -> 密钥库
'定制标识和JAVA标准信任'
'weblogic.jks路径'
'JKS'
'jks密码'

-> SSL
'密钥库' 'alias别名' 'key密码'

-> 激活更改
</script></code></pre>

## 忘记console密码的解决方法
<pre><code class="language-bash line-numbers"><script type="text/plain">1. 备份$DOMAIN_HOME/security/DefaultAuthenticatorInit.ldift

2. 进入$DOMAIN_HOME/security/
java -classpath /opt/weblogic/wlserver/server/lib/weblogic.jar weblogic.security.utils.AdminAccount weblogic weblogic .

3. $DOMAIN_HOME/servers/AdminServer/data    重命名为data_old

4. 修改$DOMAIN_HOME/servers/AdminServer/security/boot.properties
username=weblogic
password=weblogic
</script></code></pre>

## 集群部署
<pre><code class="language-bash line-numbers"><script type="text/plain">## console新建集群，proxy服务器
console -> 环境 -> 集群 -> 新建集群 -> 单点传送 -> 服务器 -> 添加服务器101,102(服务器须先关闭)
console -> 环境 -> 服务器 -> 新建Proxy -> 192.168.255.109:80 -> 启动

## 创建proxy web程序
--- proxy/WEB-INF/web.xml ---
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://java.sun.com/xml/ns/javaee   http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
 <servlet>
  <servlet-name>HttpClusterServlet</servlet-name>
  <servlet-class>weblogic.servlet.proxy.HttpClusterServlet</servlet-class>
  <init-param>
   <param-name>WebLogicCluster</param-name>
   <param-value>192.168.255.101:80|192.168.255.102:80</param-value>
  </init-param>
  <!--
   以下是证书相关配置，暂时注释 <init-param> <param-name>KeyStore</param-name>
   <param-value>/mykeystore</param-value> </init-param> <init-param>
   <param-name>KeyStoreType</param-name> <param-value>jks</param-value>
   </init-param> <init-param> <param-name>PrivateKeyAlias</param-name>
   <param-value>passalias</param-value> </init-param> <init-param>
   <param-name>KeyStorePasswordProperties</param-name>
   <param-value>mykeystore.properties</param-value> </init-param>
  -->
 </servlet>
 <servlet-mapping><!-- 这里是说哪些URL需要交给SERVLET来处理 -->
  <servlet-name>HttpClusterServlet</servlet-name>
  <url-pattern>/</url-pattern>
 </servlet-mapping>
 <servlet-mapping>
  <servlet-name>HttpClusterServlet</servlet-name><!-- 可以用*表示所有 -->
  <url-pattern>*.jsp</url-pattern>
 </servlet-mapping>
 <servlet-mapping>
  <servlet-name>HttpClusterServlet</servlet-name>
  <url-pattern>*.htm</url-pattern>
 </servlet-mapping>
 <servlet-mapping>
  <servlet-name>HttpClusterServlet</servlet-name>
  <url-pattern>*.html</url-pattern>
 </servlet-mapping>
</web-app>


--- proxy/WEB-INF/weblogic.xml ---
<?xml version="1.0" encoding="UTF-8"?>
<weblogic-web-app>
 <context-root>/</context-root>
</weblogic-web-app>


## webapps启用session复制  基于内存，文件，数据库jdbc
--- $集群web项目/WEB-INF/weblogic.xml ---
<?xml version="1.0" encoding="UTF-8"?>
<weblogic-web-app>
 <context-root>/</context-root>
 <session-descriptor>
  <persistent-store-type>replicated_if_clustered</persistent-store-type>
 </session-descriptor>
</weblogic-web-app>

<!--
<url-rewriting-enabled>true</url-rewriting-enabled>
<persistent-store-type>file</persistent-store-type>
<persistent-store-dir>F:/weblogic/sessionStore</persistent-store-dir>

<url-rewriting-enabled>true</url-rewriting-enabled>
<persistent-store-type>jdbc</persistent-store-type>
<persistent-store-pool>jdbc/dev</persistent-store-pool>
-->


## 部署web应用程序
console -> 部署 -> proxy_app -> proxyserver
console -> 部署 -> cluster_app -> cluster中的所有服务器

## 访问http://proxy_server_ip:port/
</script></code></pre>
