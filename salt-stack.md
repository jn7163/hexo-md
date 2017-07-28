---
title: saltstack 运维工具
date: 2016-11-28 19:56:00
categories:
- linux
- 运维工具
- saltstack
tags:
- linux
- 运维工具
- saltstack
keywords: saltstack, 自动运维
---
> 每天重复无聊的登录Linux服务器配置服务，修改文件，调试，让我们很烦。总是重复的干着无聊的事情，有时候真的很像解放自己，所以自动运维工具应运而生！而saltstack以其简单的安装部署及强大的功能深受python爱好者的喜爱

<!-- more -->

## saltstack

> saltstack是基于python开发的一款自动化运维工具，功能强大，容易上手

## 部署saltstack

### salt-master
<pre><code class="language-bash line-numbers">salt-master 是saltstack的管理端，以守护进程的方式运行

## 1. 安装epel源
yum -y install epel-release

## 2. 安装salt-master
yum -y install salt-master

## 3. 运行salt-master
service salt-master start # 开了防火墙需要放行4505 4506端口
</code></pre>

### salt-minion
<pre><code class="language-bash line-numbers">salt-minion 是saltstack的被管理端，以守护进程的方式运行

## 1. 安装epel源
yum -y install epel-release

## 2. 安装salt-minion
yum -y install salt-minion

## 3. 配置salt-minion
sed -i 's/#master:.*/master: MASTER_IP/' /etc/salt/minion # 修改为master的IP

## 4. 运行salt-minion
service salt-minion start
</code></pre>

### 授权
<pre><code class="language-bash line-numbers">首次连接需要授权
master端：

salt-key -L # 查看当前证书签证情况

salt-key -A -y # 同意当前所有未授权的minion
</code></pre>

### 简单实例
<pre><code class="language-bash line-numbers">## ping
salt '*' test.ping # *表示通配符，表示所有主机

## cmd.run
salt '*' cmd.run 'ls -al' # 运行shell命令并返回结果

## 默认salt使用通配符匹配主机
-E 选项 指定为正则表达式匹配
salt -E '192.168.1.\d{2}$' test.ping # 匹配最后一段ip为两位数的主机

-L 选项 指定为列表匹配
salt -L '192.168.255.101,192.168.255.222' test.ping # 匹配192.168.255.101,192.168.255.222

### 更多使用方法请查阅官方文档及帮助文件
</code></pre>
