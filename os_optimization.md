---
title: CentOS调优(基础服务，内核参数)
date: 2017-04-05 14:54:00
categories:
- linux
- 性能优化
tags:
- linux
- 性能优化
- 内核调优
keywords: 内核调优, ssh优化, 内核参数调整
---
> 系统默认的一些参数都是比较保守的，所以我们可以通过调整系统参数来提高系统内存、CPU、内核资源的占用，通过禁用不必要的服务、端口，来提高系统的安全性，更好的发挥系统的可用性

<!-- more -->

### yum常用源汇总
> 
yum的出现方便了对rpm包的管理，一些常用的第三方yum源也是必不可少的

**[常用yum源整理](https://www.zfl9.com/yum-repo.html)**

### 编译环境配置
> 
基本的linux工具，基础开发环境，编译环境

**[CentOS & RHEL 编译环境配置](https://www.zfl9.com/dev-tools.html)**

### sshd 服务优化

**[ssh 密钥登录](https://www.zfl9.com/ssh.html)**
**[ssh 安全加固](https://www.zfl9.com/ssh_security.html)**

<pre><code class="language-bash line-numbers">--- /etc/ssh/sshd_config ---
PermitEmptyPasswords no     # 禁止空密码登录

UseDNS no                   # 不解析目标主机的PTR记录

PermitRootLogin yes         # 允许root用户以任何认证方式(密码认证/RSA公钥)登录 ** 不建议
PermitRootLogin without-password    #只允许root用SSH公钥认证方式登录 ** 建议此项
PermitRootLogin no          # 不允许root用户以任何认证方式登录

PasswordAuthentication no   # 不允许所有用户使用密码认证的方式登录

GSSAPIAuthentication no     # 关闭，没什么影响，可以加快登录速度
GSSAPICleanupCredentials no
GSSAPIStrictAcceptorCheck no
GSSAPIKeyExchange no
GSSAPIEnablek5users no

X11Forwarding yes       # 启用X11转发，看你需求
X11DisplayOffset 10
X11UseLocalhost yes
</code></pre>

### 文件描述符
<pre><code class="language-bash line-numbers">### 用户限制
--- /etc/security/limits.conf ---
* soft nofile 10240
* hard nofile 10240
--- /etc/security/limits.conf ---

### 内核限制
--- /etc/sysctl.conf ---
fs.file-max = 1000000
--- /etc/sysctl.conf ---

sysctl -p

### 查看
ulimit -n       # 查看当前用户的最大文件描述符数
sysctl -a | grep file-max   # 查看内核限制的最大文件描述符数
</code></pre>

### sudo
<pre><code class="language-bash line-numbers">visudo
用户 主机=[(切换到哪些用户或用户组)] [是否需要密码验证] 命令1, [(切换到哪些用户或用户组)] [是否需要密码验证] [命令2]...
USER ALL=(ALL) NOPASSWD:ALL
</code></pre>

### 关闭三键重启CentOS6
<pre><code class="language-bash line-numbers">--- /etc/init/control-alt-delete.conf ---
#exec /sbin/shutdown -r now "Control-Alt-Deletepressed"
--- /etc/init/control-alt-delete.conf ---
</code></pre>

### 隐藏系统相关信息
<pre><code class="language-bash line-numbers">echo "Welcome to Server" > /etc/issue 
echo "Welcome to Server" > /etc/centos-release
</code></pre>

### 命令历史记录
<pre><code class="language-bash line-numbers">--- /etc/profile ---
export HISTSIZE=10000
export HISTCONTROL=ignoredups   # 忽略重复的命令记录
--- /etc/profile ---
</code></pre>

### ntp时间同步
<pre><code class="language-bash line-numbers">ntpd | chrony | ntpdate

国内常用的ntp服务器
time.windows.com
cn.pool.ntp.org
tw.pool.ntp.org

手动更新时间，ntpdate -u time.windows.com

设置时区为上海:
CentOS6: ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
CentOS7: timedatectl set-timezone Asia/Shanghai
</code></pre>

### 内核参数优化
<pre><code class="language-bash line-numbers">### 我自己的CentOS7 VPS服务器的配置(1G内存)
--- /etc/sysctl.conf ---
net.ipv6.conf.all.accept_ra = 2
net.ipv6.conf.eth0.accept_ra = 2
net.ipv4.ip_forward = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.ip_local_port_range = 4096 65000
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_max_syn_backlog = 4096
net.core.netdev_max_backlog =  10240
net.core.somaxconn = 2048
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_mem = 177945 216076 254208
net.ipv4.tcp_fastopen = 3
fs.file-max = 50000000
------------------------

### 参数详解
net.ipv4.tcp_syncookies = 1         #1是开启SYN Cookies，当出现SYN等待队列溢出时，启用Cookies来处理，可防范少量SYN攻击，默认是0关闭

net.ipv4.tcp_tw_reuse = 1           #1是开启重用，允许将TIME_AIT sockets重新用于新的TCP连接，默认是0关闭

net.ipv4.tcp_tw_recycle = 1         #TCP失败重传次数，默认是15，减少次数可释放内核资源

net.ipv4.ip_local_port_range = 4096 65000   #应用程序可使用的端口范围

net.ipv4.tcp_max_tw_buckets = 5000  #系统同时保持TIME_WAIT套接字的最大数量，如果超出这个数字，TIME_WATI套接字将立刻被清除并打印警告信息，默认180000

net.ipv4.tcp_max_syn_backlog = 4096 #进入SYN包的最大请求队列，默认是1024

net.core.netdev_max_backlog =  10240#允许送到队列的数据包最大设备队列，默认300

net.core.somaxconn = 2048           #listen挂起请求的最大数量，默认128

net.core.wmem_default = 8388608     #发送缓存区大小的缺省值(单位: byte)

net.core.rmem_default = 8388608     #接受套接字缓冲区大小的缺省值(单位: byte)

net.core.rmem_max = 16777216        #最大接收缓冲区大小的最大值(单位: byte)

net.core.wmem_max = 16777216        #发送缓冲区大小的最大值(单位: byte)

net.ipv4.tcp_synack_retries = 2     #SYN-ACK握手状态重试次数，默认5

net.ipv4.tcp_syn_retries = 2        #向外SYN握手重试次数，默认4

net.ipv4.tcp_tw_recycle = 1         #开启TCP连接中TIME_WAIT sockets的快速回收，默认是0关闭

net.ipv4.tcp_max_orphans = 3276800  #系统中最多有多少个TCP套接字不被关联到任何一个用户文件句柄上，如果超出这个数字，孤儿连接将立即复位并打印警告信息

net.ipv4.tcp_mem = 94500000 915000000 927000000

net.ipv4.tcp_mem[0]: 低于此值，TCP没有内存压力;    # 80% Mem
net.ipv4.tcp_mem[1]: 在此值下，进入内存压力阶段;   # 90% Mem
net.ipv4.tcp_mem[2]: 高于此值，TCP拒绝分配socket;  # 100% Mem

内存单位是页(1页=4kb)，可根据物理内存大小进行调整，如果内存足够大的话，可适当往上调.

net.ipv4.tcp_fastopen = 3           # 3.7版本以上的内核才支持，须客户端，服务器同时开启; 多用于shadowsocks
</code></pre>
