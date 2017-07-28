---
title: ip-tunnel隧道及nmcli命令
date: 2016-11-28 19:39:00
categories:
- linux
- 基础命令
- ip-tunnel
tags:
- linux
- 基础命令
- ip-tunnel
- ipip
- gre
keywords: ip-tunnel, ipip, gre, ip隧道, tun网卡
---
> TAP 等同于一个以太网设备，它操作第二层数据包如以太网数据帧。TUN模拟了网络层设备，操作第三层数据包比如IP数据封包。tun/tap是Linux内核中一套虚拟网络驱动软件，主要有ipip，gre两种常见的隧道

<!-- more -->

### 本地主机
<pre><code class="language-bash line-numbers">## tunctl 也可以 ##

# 新建一张 tun 网卡，模式为 ipip (gre模式只需把ipip改为gre即可)
# local指定本地物理网卡IP，remote指定目标主机外网IP
ip tunnel add tun0 mode ipip local 192.168.255.101 remote 192.168.255.102 ttl 255
# 设置 tun0 网卡IP
ip addr add 10.0.0.1/32 dev tun0
# 激活 tun0 网卡
ip link set tun0 up
# 添加一条静态路由
ip route add 10.0.0.2/32 dev tun0
</code></pre>

### 远程主机
<pre><code class="language-bash line-numbers"># 新建一张 tun 网卡，模式为 ipip (gre模式只需把ipip改为gre即可)
# local指定本地物理网卡IP，remote指定目标主机外网IP
ip tunnel add tun0 mode ipip local 192.168.255.102 remote 192.168.255.101 ttl 255
# 设置 tun0 网卡IP
ip addr add 10.0.0.2/32 dev tun0
# 激活 tun0 网卡
ip link set tun0 up
# 添加一条静态路由
ip route add 10.0.0.1/32 dev tun0
</code></pre>

### 使用 tun0 网卡
<pre><code class="language-bash line-numbers">ping 10.0.0.2 # 本地，发现可通

ping 10.0.0.1 # 远程，发现可通
</code></pre>

### nmcli 命令
<pre><code class="language-bash line-numbers">nmcli gen status # 网络总体状态

nmcli con show # 打印所有连接

nmcli con show -a # 打印活动连接

nmcli dev status # 打印所有网络设备

nmcli con add type eth con-name test_static ifname ens33 ip4 192.168.255.100 gw4 192.168.255.254 ipv4.dns '192.168.255.254 114.114.114.114 8.8.8.8' # 添加静态IP连接

nmcli con add type eth con-name test_dhcp ifname ens33 # 添加DHCP连接

nmcli con mod ens33 +ipv4.address '192.168.1.1/24'  # 添加ip

nmcli con mod ens33 -ipv4.address '192.168.2.1/24'  # 移除ip

### bridge --- networkmanager(推荐)
nmcli con add type bridge con-name br0 ifname br0 stp no ip4 192.168.255.101/24 gw4 192.168.255.254 ipv4.dns 192.168.255.254
nmcli con add type bridge-slave con-name br0-ens33 ifname ens33 master br0

### bridge --- iproute2
ip link add br0 type bridge
ip link set br0 up

ip link set ens33 promisc on
ip link set ens33 up

ip link set ens33 master br0

bridge link show

# 删除br0
ip link set ens33 promisc off
ip link set ens33 down
ip link set ens33 nomaster

ip link del br0

## 修改配置文件
[root@localhost network-scripts]# cat ifcfg-br0
DEVICE=br0
BOOTPROTO=none
#HWADDR=00:0c:29:11:6a:09
IPV6INIT=yes
NM_CONTROLLED=yes
ONBOOT=yes
TYPE=Bridge
#UUID="2ecfcd74-4ab0-4d2b-a92a-9306ef8fdd58"
IPADDR=192.168.255.102
NETMASK=255.255.255.0
GATEWAY=192.168.255.254
DNS1=192.168.255.254
USERCTL=no
PEERDNS=yes

[root@localhost network-scripts]# cat ifcfg-eth0
DEVICE=eth0
BOOTPROTO=none
HWADDR=00:0c:29:11:6a:09
IPV6INIT=yes
NM_CONTROLLED=yes
ONBOOT=yes
TYPE=Ethernet
#UUID="2ecfcd74-4ab0-4d2b-a92a-9306ef8fdd58"
IPADDR=192.168.255.102
NETMASK=255.255.255.0
GATEWAY=192.168.255.254
DNS1=192.168.255.254
USERCTL=no
PEERDNS=yes
BRIDGE=br0

service NetworkManager restart
service network restart
</code></pre>
