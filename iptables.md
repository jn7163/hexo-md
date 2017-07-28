---
title: iptables详解
date: 2016-11-03 17:29:22
categories:
- linux
- 防火墙
- iptables
tags:
- linux
- 防火墙
- iptables
keywords: iptables, 防火墙
---
> 
`netfilter/iptables`包过滤系统被称为单个实体，但它实际上由两个组件`netfilter`和`iptables`组成。
`netfilter`组件也称为内核空间(kernelspace)，是内核的一部分，由一些信息包过滤表组成，这些表包含内核用来控制信息包过滤处理的规则集;
`iptables`组件是一种工具，也称为用户空间(userspace)，它使插入、修改和除去信息包过滤表中的规则变得容易

<!-- more -->

## netfilter/iptables
netfilter选取了五个位置进行数据包操作
`PREROUTING`    外界数据包路由前
`INPUT`         外界数据包流入用户空间
`FORWARD`       网卡之间数据包的转发(内核允许转发的情况下)
`OUTPUT`        自身应用程序产生的数据包从用户空间流出
`POSTROUTING`   数据包路由后

## 数据包流向
![iptables](images/iptables.png)
1. 数据包进入网卡时，它首先进入PREROUTING链，内核根据数据包目的IP判断是否需要转发出去。

2. 如果数据包就是进入本机的，它就会到达INPUT链。数据包到了INPUT链后，任何进程都会收到它。本机上运行的程序可以发送数据包，这些数据包会经 过OUTPUT链，然后到达POSTROUTING链输出。

3. 如果数据包是要转发出去的，且内核允许转发，数据包就会经过 FORWARD链，然后到达POSTROUTING链输出。

## 数据包流向实例
1. 入站数据流向(外部访问本机提供的http服务)
从外界到达防火墙的数据包，先被PREROUTING规则链处理，之后会进行路由选择，发现数据包的目标主机是防火墙本机。
那么内核将其传递给INPUT链进行处理，通过以后再交给系统上层的应用程序(如Nginx守护进程)进行响应。

2. 转发数据流向(内网主机访问外部网络)
来自内网主机的数据包到达防火墙后，首先被PREROUTING规则链处理，之后会进行路由选择，发现数据包的目标地址是外部网络地址。
则内核将其传递给FORWARD链进行处理，然后再交给POSTROUTING规则链进行SNAT等处理再发往外部网络。

3. 出站数据流向(本机产生的数据包,如请求DNS解析)
防火墙本机向外部网络发送的数据包，首先被OUTPUT规则链处理，之后进行路由选择，然后传递给POSTROUTING规则链进行处理再发往外部网络。

## 应用规则及优先级
### 表的优先级
从左到右优先级依次降低 `raw->mangle->nat->filter`
`raw`       高级功能，如：网址过滤
`mangle`    数据包修改（QOS），用于实现服务质量
`nat`       地址转换，用于网关路由器
`filter`    包过滤，用于防火墙规则

|表名|---|---|---|---|---|
|:--:|:-:|:-:|:-:|:-:|:-:|
|raw|PREROUTING|-|-|OUTPUT|-|
|mangle|PREROUTING|INPUT|FORWARD|OUTPUT|POSTROUTING|
|nat|PREROUTING|INPUT|-|OUTPUT|POSTROUTING|
|filter|-|INPUT|FORWARD|OUTPUT|-|

### 链中规则的应用顺序
按照表的优先级，先处理优先级高的
表中的规则按照从上至下的顺序依次匹配，
如果命中了一条匹配则执行相应的动作
如果没有命中任何匹配或者表中规则为空，则根据表的默认策略进行允许或拒绝操作
然后继续匹配下一个优先级的表的规则
直到所有表的规则匹配完成

raw表只使用在PREROUTING链和OUTPUT链上,因为优先级最高，从而可以对收到的数据包在连接跟踪前进行处理。
一但使用了raw表,在某个链上,raw表处理完后,将跳过nat表和ip_conntrack处理,即不再做地址转换和数据包的链接跟踪处理。

## 参数详解
<pre><code class="language-bash line-numbers"># iptables命令格式
iptables -t 表名 -A/I/D/R 链名 [规则号] 匹配规则 -j 动作


# iptables-save,iptables-restore
# 规则备份和恢复,默认保存在/etc/sysconfig/iptables
# iptables服务启动时会加载/etc/sysconfig/iptables的规则
iptables-save > /etc/sysconfig/iptables
iptables-restore < /etc/sysconfig/iptables
iptables-save > /etc/sysconfig/iptables.20161103
iptables-restore < /etc/sysconfig/iptables.20161103


# iptables参数
-t  指定要操作的表(默认filter表)
-A  向规则链中添加条目
-D  从规则链中删除条目
-I  向规则链中插入条目
-R  替换规则链中的条目
-L  显示规则链中已有的条目
-F  清空规则链中已有的条目
-Z  清空规则链中的数据包计算器和字节计数器
-N  创建新的用户自定义规则链
-E  重命名自定义链
-X  删除自定义链
-P  定义规则链中的默认策略 (注:nat表不允许设置默认策略为DROP!)
-S  显示规则表的规则
-v  显示规则链中条目详细信息
--line-numbers  显示条目序号


# 匹配规则
## 通用匹配
-s/d    源/目的地址.前面加!取反(! -s/d 1.1.1.1 下同)
-i/o    数据包流入/流出网卡,+表示通配(如: eth+)
-p      协议,tcp/udp/icmp/all


## 扩展匹配
### 隐式扩展(-p协议扩展)
tcp
--dport/sport 80        # 目的端口/源端口 80
--dport/sport 80:100    # 目的端口/源端口范围 80-100

udp
--dport/sport 80        # 同上

icmp
--icmp-type 8/0         # request/reply包


### 显式扩展(-m 加载module)
multiport   # 多端口
-m multiport --dports/sports/port(s) 67,68/80:100

owner       # 所属用户(组)
-m owner --uid/gid-owner 0

state       # 连接状态,tcp/udp/icmp均有效
-m state --state NEW,ESTABLISHED
NEW         # 新连接
ESTABLISHED # 已建立连接
RELATED     # 类似ftp多端口有关联的连接
INVALID     # 无效的连接

iprange     # ip范围
-m iprange --src-range 1.1.1.1-1.1.1.100 # 源
-m iprange --dst-range 1.1.1.1-1.1.1.100 # 目的

mac         # 源mac地址
-m mac --mac-source 00:00:00:00:00:00

time        # 时间
-m time --timestart/stop 00:00
-m time --datestart/stop 2000-10-10T10:10:10
-m time --weekdays Mon,Tue,Wed,Thu,Fri,Sat,Sun

string      # 应用层数据关键字
-m string --algo mp/kmp --string "string"


# -j 动作
ACCEPT      # 接收数据包
DROP        # 丢弃数据包
REJECT      # 拒绝并返回信息
REDIRECT    # 重定向(端口)
SNAT        # 源地址转换
DNAT        # 目标地址转换
MASQUERADE  # 源地址转换,用于公网IP非固定的情况下,如DHCP获取方式,自动判断公网IP
Cust-Name   # 跳到自定义链
RETURN      # 返回,返回父链.在自定义链执行完毕后返回父链
LOG         # 记录log /var/log/messages

-j REDIRECT --to-ports 8080 # 用于端口重定向 PREROUTING/OUTPUT(nat表)
SNAT # INPUT/POSTROUTING    # (nat表)
DNAT # OUTPUT/PREROUTING    # (nat表)
MASQUERADE # POSTROUTING    # (nat表)

-j SNAT --to-source 1.1.1.1
-j DNAT --to-destination 10.0.0.1
-j MASQUERADE
-j MASQUERADE --to-ports 10000-10999

-j LOG --log-prefix 'xxx'

## return 动作
如果当前链为子链：则返回至父链的调用处，继续匹配父链的下一个规则
如果当前链不是子链：则按照当前链的默认策略执行相应动作
</code></pre>

## 实例
<pre><code class="language-bash line-numbers"># 允许eth网卡转发数据包
iptables -A FORWARD -i eth+ -j ACCEPT
iptables -A FORWARD -o eth+ -j ACCEPT


# 放行回环接口lo
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT


# 拒绝本机接收icmp数据包
iptables -A INPUT -p icmp -j DROP


# 放行ssh
iptables -A INPUT -p tcp --dport 22 -j ACCEPT


# 拒绝192.168.3.0/24主机ping和traceroute本机
iptables -A INPUT -s 192.168.3.0/24 -p icmp -j DROP


# 实现内网主机访问公网
iptables -t nat -A POSTROUTING -j MASQUERADE


# 查看nat表明细
iptables -t nat -nvL
iptables -t nat -nvL --line-numbers # 显示行号


# 查看nat表规则
iptables -t nat -S
</code></pre>

## ipset
<pre><code class="language-bash line-numbers">使用ipset提高 iptables 性能

yum -y install ipset

[root@zfl ~]# ipset -N myset hash:net
[root@zfl ~]# ipset -A myset 127.0.0.0/8
[root@zfl ~]# ipset -A myset 172.16.0.0/12
[root@zfl ~]# ipset -A myset 192.168.0.0/16

[root@zfl ~]# ipset -S > /etc/sysconfig/ipset # 持久化至配置文件
[root@zfl ~]# ipset -R < /etc/sysconfig/ipset # 从配置文件恢复

[root@zfl ~]# iptables -A INPUT -m set --match-set myset src -j ACCEPT # src/dst

[root@zfl ~]# ipset -X myset
ipset v6.19: Set cannot be destroyed: it is in use by a kernel component
[root@zfl ~]# iptables -F
[root@zfl ~]# ipset -X myset # 清空
[root@zfl ~]# ipset -L
</code></pre>

## recent模块
> 
很实用的iptables模块[iptables_recent](https://www.zfl9.com/ssh_security.html#iptables-recent)
可以有效抵御ssh暴力破解，cc攻击...
