---
title: ntp时间同步
date: 2016-12-07 13:27:00
categories:
- linux
- 时间同步
- ntp
tags:
- linux
- 时间同步
- ntp
keywords: ntp, ntpdate, ntpd, 时间同步
---
> 服务器的时间同步是很重要的！不然很多服务和操作都会出现这样那样的问题，全部乱套。下面用ntpd实现Linux间的时间同步

<!-- more -->

## ntp协议

> 网络时间协议（英语：Network Time Protocol，NTP）是以分组交换把两台电脑的时钟同步化的网络传输协议。NTP使用UDP端口123作为传输层。它是用作抵销可变延迟的影响。
NTP是仍在使用中的最古老的网络传输协议之一（在1985年前开始）。NTP最初由特拉华大学的Dave Mills设计，他与一群志愿者仍在维护NTP。

## 实验环境
内网ntp服务器 192.168.255.103 (准确说应该是ntp_relay)
内网ntp客户端 192.168.255.104

## ntpd_server(192.168.255.103)

### 安装ntp
<pre><code class="language-bash line-numbers">yum -y install ntp
</code></pre>

### 配置ntp
<pre><code class="language-bash line-numbers">## ntp.conf ##
vim /etc/ntp.conf

--- ntp.conf ---
## 允许192.168.255.0/24的主机从本机同步时间
restrict 192.168.255.0 mask 255.255.255.0 nomodify notrap
## 上游公共ntp服务器
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
## 国内常用ntp服务器
time.windows.com
cn.pool.ntp.org
tw.pool.ntp.org

## 日志
logfile /var/log/ntp.log
--- ntp.conf ---

## ntpd ##
vim /etc/sysconfig/ntpd

--- ntpd ---
SYNC_HWCLOCK=yes # 将系统时间写入BIOS
--- ntpd ---
</code></pre>

### ntpdate手动同步时间
<pre><code class="language-bash line-numbers">ntpdate -u cn.pool.ntp.org
hwclock -w
</code></pre>

### 启动ntpd服务
<pre><code class="language-bash line-numbers">chkconfig ntpd on
service ntpd start
</code></pre>

### 查看ntpd状态
<pre><code class="language-bash line-numbers">ntpq -p # 查看ntp服务器状态

ntpstat # 查看ntpd同步状态，一般5-10分钟才能成功与服务器同步
## 刚启动的状态：
unsynchronised
   polling server every 64 s

## 成功同步的状态：
synchronised to NTP server (202.118.1.130) at stratum 3 
   time correct to within 8011 ms
   polling server every 64 s
</code></pre>

## ntpd_client(192.168.255.104)

### 安装ntp
<pre><code class="language-bash line-numbers">yum -y install ntp
</code></pre>

### 配置ntp
<pre><code class="language-bash line-numbers">## ntp.conf ##
vim /etc/ntp.conf

--- ntp.conf ---
## 上游公共ntp服务器
server 192.168.255.103 iburst # 其他的server全部注释
## 日志
logfile /var/log/ntp.log
--- ntp.conf ---

## ntpd ##
vim /etc/sysconfig/ntpd

--- ntpd ---
SYNC_HWCLOCK=yes # 将系统时间写入BIOS
--- ntpd --- 
</code></pre>


### ntpdate手动同步时间
<pre><code class="language-bash line-numbers">ntpdate -u cn.pool.ntp.org
hwclock -w
</code></pre>

### 启动ntpd服务
<pre><code class="language-bash line-numbers">chkconfig ntpd on
service ntpd start
</code></pre>

### 查看ntpd状态
<pre><code class="language-bash line-numbers">ntpq -p # 查看ntp服务器状态

ntpstat # 查看ntpd同步状态，因为是内网所以很快就能同步
</code></pre>
