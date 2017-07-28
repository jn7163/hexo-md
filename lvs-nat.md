---
title: lvs -> vs/nat 负载均衡配置
date: 2016-11-29 18:49:00
categories:
- linux
- lvs
tags:
- linux
- lvs
keywords: lvs, vs/nat 负载均衡
---
> lvs是Linux Virtual Server的简称，顾名思义：linux虚拟服务器。当然这是拿来做服务器集群 负载均衡的。lvs有三种IP负载均衡技术：(vs/nat, vs/tun, vs/dr)。我们先介绍第一种：vs/nat

<!-- more -->

### vs/nat

> Virtual Server via Network Address Translation（VS/NAT）
通过网络地址转换，调度器重写请求报文的目标地址，根据预设的调度算法，将请求分派给后端的真实服务器；真实服务器的响应报文通过调度器时，报文的源地址被重写，再返回给客户，完成整个负载调度过程。

### lvs 集群的设备地址命名

VIP：Virtual IP          LVS 面向用户请求的 IP 地址
RIP：Real server IP      后端服务器用于和 LVS 通信的 IP 地址
DIP：Director IP         LVS 用户和后端服务器通信的 IP 地址
CIP：Client IP           客户端 IP 地址

### 实验环境

vip: 10.0.0.1
dip: 192.168.255.101
rip1: 192.168.255.102
rip2: 192.168.255.103

### 安装 ipvsadm
<pre><code class="language-bash line-numbers">yum -y install ipvsadm
</code></pre>

### 配置vs(192.168.255.101)
<pre><code class="language-bash line-numbers">echo 1 > /porc/sys/net/ipv4/ip_forward  开启内核网卡转发
ipvsadm -C                              清空之前的转换表
ipvsadm -At 10.0.0.1:80 -s rr           指定带有调度算法转换的服务器
ipvsadm -at 10.0.0.1:80 -r 192.168.255.102:80 -m 增加一台真实服务器, -m是nat模式
ipvaadm -at 10.0.0.1:80 -r 192.168.255.103:80 -m
</code></pre>

### 配置rs1(192.168.255.102)
<pre><code class="language-bash line-numbers">为了方便测试，在rs1、2中配置nginx 模拟web服务集群
yum -y install nginx
service nginx start
cd /usr/share/nginx/html/
echo &#x27;&lt;h1&gt;RS1:192.168.255.102:80&lt;/h1&gt;&#x27; &gt; index.html
</code></pre>

### 配置rs2(192.168.255.103)
<pre><code class="language-bash line-numbers">yum -y install nginx
service nginx start
cd /usr/share/nginx/html/
echo &#x27;&lt;h1&gt;RS2:192.168.255.103:80&lt;/h1&gt;&#x27; &gt; index.html
</code></pre>

### 测试
<pre><code class="language-bash line-numbers">在vs(192.168.255.101)中测试：
curl 10.0.0.1
&lt;h1&gt;RS1:192.168.255.102:80&lt;/h1&gt;
curl 10.0.0.1
&lt;h1&gt;RS2:192.168.255.103:80&lt;/h1&gt;
curl 10.0.0.1
&lt;h1&gt;RS1:192.168.255.102:80&lt;/h1&gt;
curl 10.0.0.1
&lt;h1&gt;RS2:192.168.255.103:80&lt;/h1&gt;

发现在RS1/2之间轮询，说明负载均衡配置成功！
</code></pre>
