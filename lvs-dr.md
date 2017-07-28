---
title: lvs -> vs/dr 负载均衡配置
date: 2016-12-08 08:42:00
categories:
- linux
- lvs
tags:
- linux
- lvs
keywords: lvs, vs/dr, 负载均衡, 集群
---
> DR负载均衡模式数据分发过程中不修改IP地址，只修改mac地址，由于实际处理请求的真实物理IP地址和数据请求目的IP地址一致，所以不需要通过负载均衡服务器进行地址转换，可将响应数据包直接返回给用户浏览器，避免负载均衡服务器网卡带宽成为瓶颈。因此，DR模式具有较好的性能，也是目前大型网站使用最广泛的一种负载均衡手段

<!-- more -->

### vs/dr

> DR模式下需要LVS和RS集群绑定同一个VIP（RS通过将VIP绑定在loopback实现），
但与NAT的不同点在于：请求由LVS接受，由真实提供服务的服务器（RealServer, RS）直接返回给用户，返回的时候不经过LVS。
详细来看，一个请求过来时，LVS只需要将网络帧的MAC地址修改为某一台RS的MAC，该包就会被转发到相应的RS处理，注意此时的源IP和目标IP都没变，LVS只是做了一下移花接木。
RS收到LVS转发来的包时，链路层发现MAC是自己的，到上面的网络层，发现IP也是自己的，于是这个包被合法地接受，RS感知不到前面有LVS的存在。
而当RS返回响应时，只要直接向源IP（即用户的IP）返回即可，不再经过LVS。

### 实验环境
OS： CentOS 7.2
vip：192.168.255.111
dip：192.168.255.103
rip1: 192.168.255.104
rip2: 192.168.255.105

### vs(192.168.255.103)

#### 安装ipvsadm
<pre><code class="language-bash line-numbers">yum -y install ipvsadm
</code></pre>

#### 配置vs/dr脚本
<pre><code class="language-bash line-numbers">vim /etc/init.d/lvs_dr

--- lvs_dr ---
#!/bin/bash

. /etc/init.d/functions

vip=192.168.255.111
rip1=192.168.255.104
rip2=192.168.255.105

case $1 in
start)
    ip addr add $vip/32 broadcast $vip dev eno16777736
    ip route add $vip/32 dev eno16777736
    ipvsadm -C
    ipvsadm -At $vip:80 -s rr
    ipvsadm -at $vip:80 -r $rip1:80 -g
    ipvsadm -at $vip:80 -r $rip2:80 -g
    ;;
stop)
    ipvsadm -C
    ip addr del $vip/32 dev eno16777736
    ip route del $vip/32 dev eno16777736
    ;;
status)
    ipvsadm -ln
    ;;
*)
    echo "Usage: $0 {start|stop|status}"
    exit 1
    ;;
esac

exit 0
--- lvs_dr ---

chmod +x /etc/init.d/lvs_dr
</code></pre>

### rs(192.168.255.104/105)

#### 安装nginx
<pre><code class="language-bash line-numbers">yum -y install epel-release
yum -y install nginx
</code></pre>

#### 修改index.html
<pre><code class="language-bash line-numbers">### 192.168.255.104 ###
echo &#x27;&lt;h1&gt;104&lt;/h1&gt;&#x27; &gt; /usr/share/nginx/html/index.html

### 192.168.255.105 ###
echo &#x27;&lt;h1&gt;105&lt;/h1&gt;&#x27; &gt; /usr/share/nginx/html/index.html

## 为了实验方便，我们让两台rs的内容不一样，以便识别，实际生产中应该保持一致！
</code></pre>

#### 运行nginx
<pre><code class="language-bash line-numbers">systemctl start nginx
</code></pre>

#### 配置vs/dr脚本
<pre><code class="language-bash line-numbers">vim /etc/init.d/lvs_dr

--- lvs_dr ---
#!/bin/bash

. /etc/init.d/functions

vip=192.168.255.111

case $1 in
start)
    ip addr add $vip/32 broadcast $vip dev lo
    ip route add $vip/32 dev lo
    echo 1 >/proc/sys/net/ipv4/conf/lo/arp_ignore 
    echo 2 >/proc/sys/net/ipv4/conf/lo/arp_announce 
    echo 1 >/proc/sys/net/ipv4/conf/all/arp_ignore 
    echo 2 >/proc/sys/net/ipv4/conf/all/arp_announce
    ;;
stop)
    ip addr del $vip/32 dev lo
    ip route del $vip/32 dev lo
    echo 0 >/proc/sys/net/ipv4/conf/lo/arp_ignore 
    echo 0 >/proc/sys/net/ipv4/conf/lo/arp_announce 
    echo 0 >/proc/sys/net/ipv4/conf/all/arp_ignore 
    echo 0 >/proc/sys/net/ipv4/conf/all/arp_announce
    ;;
*)
    echo "Usage: $0 {start|stop}"
    exit 1
esac

exit 0
--- lvs_dr ---

chmod +x /etc/init.d/lvs_dr
</code></pre>

### 执行vs/dr脚本
<pre><code class="language-bash line-numbers">### vs: 192.168.255.103
service lvs_dr start

### rs1: 192.168.255.104
service lvs_dr start

### rs2: 192.168.255.105
service lvs_dr start
</code></pre>

### 测试
> 正常情况

![vs/dr](images/lvs_dr-1.png)

> 有服务器down机时 192.168.255.105 -> `systemctl stop nginx`

![vs/dr](images/lvs_dr-2.png)

> 恢复正常 192.168.255.105 -> `systemctl start nginx`

![vs/dr](images/lvs_dr-1.png)
