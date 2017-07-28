---
title: jdk安装及配置
date: 2017-04-05 14:35:00
categories:
- linux
- 基础命令
- jdk
tags:
- linux
- 基础命令
- jdk
- jdk环境变量
keywords: jdk, jdk环境变量, java环境变量
---
> CentOS有些默认安装了openjdk，需要先卸载，然后安装Sun的jdk

<!-- more -->

### jdk下载
<pre><code class="language-bash line-numbers">jdk1.8下载页面:     http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

jdk-8u121-linux-x64.tar.gz      # 一般选择tar.gz，比较通用
</code></pre>

### jdk安装及配置
<pre><code class="language-bash line-numbers">### 安装到/usr/java
mkdir -p /usr/java
tar xf jdk-8u121-linux-x64.tar.gz -C /usr/java/

### 配置java环境变量
vim /etc/profile.d/jdk.sh
--- jdk.sh ---
export JAVA_HOME=/usr/java/jdk1.8.0_121     # 填写jdk解压的路径就行
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

. /etc/profile      # 生效jdk环境变量

### 测试是否设置成功
java
java -version
javac
</code></pre>
