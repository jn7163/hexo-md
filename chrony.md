---
title: chrony时间同步
date: 2016-12-07 17:35:00
categories:
- linux
- 时间同步
- chrony
tags:
- linux
- 时间同步
- chrony
keywords: chrony, ntp, centos7, 时间同步
---
> Linux用户对于时间同步，基本上是使用ntpdate和ntpd这两个工具实现的，但是这两个工具已经很古老了，CentOS/RHEL7 已经将chrony作为默认时间同步工具

<!-- more -->

## chrony简介

> chrony是redhat开发的，它是网络时间协议的 (NTP) 的另一种实现。
centos7/rhel7默认的时间同步工具，在centos6.8之后，老的centos和rhel6系列也添加上了这个工具。
Chrony可以同时做为ntp服务的客户端和服务端。
默认安装完后有两个程序chronyd和chronyc 。
chronyd是一个在系统后台运行的守护进程，chronyc是用来监控chronyd性能和配置其参数程序。

## 实验环境

OS:   CentOS 7.2 (1511)
chrony_server(relay):   192.168.255.103
chrony_client:          192.168.255.104

## chrony_server

### 安装chrony
<pre><code class="language-bash line-numbers">yum -y install chrony
</code></pre>

### 配置chrony
<pre><code class="language-bash line-numbers">vim /etc/chrony.conf

--- chrony.conf ---
## 上游公共ntp服务器
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

## 允许192.168.255.0/24内网主机同步时间
allow 192.168.255/24
--- chrony.conf ---
</code></pre>

### 运行chrony
<pre><code class="language-bash line-numbers">systemctl enable chronyd
systemctl start chronyd
</code></pre>

### timedatectl
<pre><code class="language-bash line-numbers">## 查看时间
timedatectl

## 开启ntp时间同步
timedatectl set-ntp true
</code></pre>

### chronyc
<pre><code class="language-bash line-numbers">## 查看ntp_servers状态
chronyc sources -v

## 查看ntp_sync状态
chronyc sourcestats -v

## 查看ntp_servers 是否在线
chronyc activity -v

## 查看ntp时间详细信息
chronyc tracking -v
</code></pre>

## chrony_client

### 安装chrony
<pre><code class="language-bash line-numbers">yum -y install chrony
</code></pre>

### 配置chrony
<pre><code class="language-bash line-numbers">vim /etc/chrony.conf 

--- chrony.conf ---
## 内网ntp_server IP
server 192.168.255.103 iburst

## 允许内网ntp_server进行时间同步
allow 192.168.255.103
--- chrony.conf ---
</code></pre>

### 运行chrony
<pre><code class="language-bash line-numbers">systemctl enable chronyd
systemctl start chronyd
</code></pre>

### timedatectl
<pre><code class="language-bash line-numbers">## 查看时间
timedatectl

## 开启ntp时间同步
timedatectl set-ntp true
</code></pre>

### chronyc
<pre><code class="language-bash line-numbers">## 查看ntp_servers状态
chronyc sources -v

## 查看ntp_sync状态
chronyc sourcestats -v

## 查看ntp_servers 是否在线
chronyc activity -v

## 查看ntp时间详细信息
chronyc tracking -v
</code></pre>
