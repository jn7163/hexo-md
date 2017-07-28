---
title: keepalived + nginx 高可用配置
date: 2016-11-30 12:53:00
categories:
- linux
- 网络服务
- keepalived
tags:
- linux
- 网络服务
- keepalived
- HA高可用
keywords: HA, keepalived, nginx, 高可用, 集群
---
> Keepalived是一个基于VRRP协议来实现的服务高可用方案，可以利用其来避免IP单点故障，类似的工具还有heartbeat、corosync、pacemaker。但是它一般不会单独出现，而是与其它负载均衡技术（如lvs、haproxy、nginx）一起工作来达到集群的高可用。下面记录了keepalived+nginx 实现集群高可用

<!-- more -->

###  实验环境 CentOS 7
VIP1:192.168.255.111
VIP2:192.168.255.222
互为主备，实现高可用

### keepalived 安装
<pre><code class="language-bash line-numbers">yum -y install keepalived
</code></pre>

### keepalived 配置(server1)
<pre><code class="language-bash line-numbers">### 创建 nginx 检测脚本
cd /etc/keepalived/
vim nginx.sh
--- nginx.sh ---
#!/bin/bash
status=$(ss -lnp | grep -c 'nginx')
if [ ${status} == 0 ]; then
    systemctl nginx restart
    sleep 2
    status=$(ss -lnp | grep -c 'nginx')
    if [ ${status} == 0 ]; then
        systemctl stop keepalived
    fi
fi
--- nginx.sh ---

chmod +x nginx.sh

vim keepalived.conf
--- keepalived.conf ---
! Configuration File for keepalived

vrrp_script nginx {
    script "/etc/keepalived/nginx.sh"
    interval 2 # 检测间隔2s
    weight -5 # 若检测失败权重减低5
    fall 3 # 检测失败3次就定义为down状态
    rise 2 # 检测失败后,检测成功超过2次就定义为up状态
}

vrrp_instance VI_1 {
    state MASTER # master_server
    interface eno16777736
    virtual_router_id 51
    priority 100 # 权重值，值大的优先级高
    advert_int 2 # 检测时间间隔2s
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.255.111 # VIP
    }
    track_script {
       nginx # 检测脚本
    }
}

vrrp_instance VI_2 {
    state BACKUP # backup_server
    interface eno16777736
    virtual_router_id 51
    priority 98
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.255.222
    }
    track_script {
       nginx
    }
}
--- keepalived.conf ---
</code></pre>

### keepalived 配置(server2)
<pre><code class="language-bash line-numbers">### 创建 nginx 检测脚本
cd /etc/keepalived/
vim nginx.sh
--- nginx.sh ---
#!/bin/bash
status=$(ss -lnp | grep -c 'nginx')
if [ ${status} == 0 ]; then
    systemctl nginx restart
    sleep 2
    status=$(ss -lnp | grep -c 'nginx')
    if [ ${status} == 0 ]; then
        systemctl stop keepalived
    fi
fi
--- nginx.sh ---

chmod +x nginx.sh

vim keepalived.conf
--- keepalived.conf ---
! Configuration File for keepalived

vrrp_script nginx {
    script "/etc/keepalived/nginx.sh"
    interval 2 # 检测间隔2s
    weight -5 # 若检测失败权重减低5
    fall 3 # 检测失败3次就定义为down状态
    rise 2 # 检测失败后,检测成功超过2次就定义为up状态
}

vrrp_instance VI_1 {
    state BACKUP
    interface eno16777736
    virtual_router_id 51
    priority 98 # 权重值，值大的优先级高
    advert_int 2 # 检测时间间隔2s
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.255.111 # VIP
    }
    track_script {
       nginx # 检测脚本
    }
}

vrrp_instance VI_2 {
    state MASTER
    interface eno16777736
    virtual_router_id 51
    priority 100
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.255.222
    }
    track_script {
       nginx
    }
}
--- keepalived.conf ---
</code></pre>

### 运行keepalived
<pre><code class="language-bash line-numbers">systemctl start keepalived

ip addr # 查看虚拟IP
</code></pre>

### 浏览器测试

##### 正常情况下
<pre><code class="language-bash line-numbers">192.168.255.111
192.168.255.222

均可访问，且后端web服务器负载均衡正常
</code></pre>

##### 模拟down机
<pre><code class="language-bash line-numbers">server2 --> systemctl stop keepalived

发现依旧可以访问，且后端web服务器负载均衡正常
</code></pre>
