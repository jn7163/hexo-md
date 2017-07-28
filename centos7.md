---
title: CentOS/RHEL7 变化汇总
date: 2016-10-23 13:50:11
categories:
- linux
- centos&rhel-7
tags:
- linux
- centos&rhel-7变化
keywords: CentOS/RHEL7, 变化汇总
---
> CentOS/RHEL7 带来了很多系统配置和管理上的变化，虽然有些不习惯，但个人认为大家应该去适应它们，而不是因循守旧，因为变化总是有原因的，它们大致都代表了未来的趋势，你不可能一直停留在原地踏步

<!-- more -->

## Tab键支持补全命令参数
<pre><code class="language-bash line-numbers"># 最小化安装默认没有安装 bash-completion
yum -y install bash-completion # 安装完后重启
</code></pre>

## GRUB升级至2.0
bootloader的配置文件也调整至 `/boot/grub2/grub.cfg`
grub2开机进入`单用户模式`的方式与CentOS6有所不同，具体请戳[CentOS7.x 开机进入单用户模式](https://www.zfl9.com/centos7-init1.html)

## `/etc/inittab` 不再使用
`cat /etc/inittab` 发现
<pre><code class="language-bash line-numbers"># systemd uses 'targets' instead of runlevels. By default, there are two main targets: 
# 
# multi-user.target: analogous to runlevel 3 
# graphical.target: analogous to runlevel 5 
# 
# To view current default target, run: 
# systemctl get-default 
# 
# To set a default target, run: 
# systemctl set-default TARGET.target
</code></pre>
<pre><code class="language-bash line-numbers">systemctl get-default               # 查看当前运行级别
systemctl set-default TARGET.target # 设置默认运行级别
</code></pre>

## 主机名保存到了`/etc/hostname`文件中

## 全面由`systemd`管理服务
<pre><code class="language-bash line-numbers"># 常用systemctl命令
systemctl enable|disable firewalld.service              # (添加/移除)开机自启
systemctl start|stop|restart|status firewalld.service   # (启动|停止|重启|状态)服务
systemctl list-unit-files -t service                    # 查看所有开机自启的服务
systemctl list-unit-files -t service | grep iptables
journalctl -b                                           # 类似于dmesg
</code></pre>

## `/etc/rc.local`默认没有执行权限
建议创建自己的systemd services
如果需要使用`rc.local`,请自行添加执行权限
<pre><code class="language-bash line-numbers">chmod +x /etc/rc.local
chmod +x /etc/rc.d/rc.local
</code></pre>

## `/etc/sysctl.conf`中的默认配置移至`/usr/lib/sysctl.d/00-system.conf`
<pre><code class="language-bash line-numbers">cat /etc/sysctl.conf
# System default settings live in /usr/lib/sysctl.d/00-system.conf.
# To override those settings, enter new settings here, or in an /etc/sysctl.d/<name>.conf file
# 在这里设置配置会覆盖 00-system.conf 下的默认配置
# 在 /etc/sysctl.d/ 下创建一个 <name>.conf 的文件也可以
</code></pre>

## 新的网卡命名规则(不再是eth0)
- `en` for Ethernet             以太网
- `wl` for wireless LAN (WLAN)  无线局域网
- `ww` for wireless wide area network (WWAN)    无线广域网

## `/etc/udev/rules.d/70-persistent-net.rules`不存在了
用虚拟机的朋友可能还记得，每次虚拟机克隆都要改网卡配置文件和上面那个文件，很是头疼，CentOS/RHEL7取消了这个文件，以后复制虚拟机只需要修改网卡配置文件即可

## `ip/ss`替代`ifconfig|route|arp|netstat`
<pre><code class="language-bash line-numbers"># 若没有安装iproute软件包则自行安装
yum -y install iproute  # iproute全面代替之前的net-tools
ip addr                 # 查看网卡ip等信息
ip route                # 查看路由表
ss -lnp | grep named    # ss代替netstat 命令参数几乎没变
</code></pre>

## 旧的`network`脚本(service)和`ifcfg`文件
> CentOS/RHEL7 开始，网络由`NetworkManager`服务负责管理

需要注意的是：
1. 不建议`systemctl disable NetworkManager.service`
2. 因为旧的`network`脚本不兼容`ifcfg-*`文件里的新的配置项名称`IPADDR0/PREFIX0/GATEWAY0`
3. 除非把后面那个`0`去掉，否则开机是无法启动网卡的

## 网络配置变化
### `nmtui`
`nmtui`属于`curses-based text user interface`(文本用户界面),类似`Centos6`的`setup`(在CentOS/RHEL7 中`setup`已经没啥用了)工具
但只能编辑连接、启用/禁用连接、更改主机名。系统初装之后可以第一时间用nmtui配置网络，挺方便

### `nmcli`
[nmcli命令用法](https://www.zfl9.com/ip-tunnel.html#nmcli-命令)
`nmcli`是命令行形式的工具，功能要强大、复杂的多
`nmcli help`
`Usage: nmcli [OPTIONS] OBJECT { COMMAND | help }`
`OBJECT`和`COMMAND`可以用全称也可以用简称，最少可以只用一个字母，建议用头三个字母。`OBJECT`里面我们平时用的最多的就是`connection`和`device`，这里需要简单区分一下`connection`和`device` :
`device` 叫网络接口，是物理设备
`connection` 是连接，偏重于逻辑设置

多个`connection`可以应用到同一个`device`，但同一时间只能启用其中一个`connection`

这样的好处是针对一个网络接口，我们可以设置多个网络连接，比如静态IP和动态IP，再根据需要`up`相应的`connection`
我们可以用`nmtui`把两个连接改成我们熟悉的名字(nmcli也能，但比较麻烦)

### 重新载入配置文件
<pre><code class="language-bash line-numbers"># 在修改了配置文件后需要重新载入配置文件使其生效
nmcli con reload
nmcli dev con eno16777736
</code></pre>

`ip addr add`   临时添加ip,重启失效
`ifcfg-eth0:x`  这种方式已经失效
添加多ip等请在网卡配置文件添加字段`IPADDRx/PREFIXx/GATEWAYx`
`/etc/resolv.conf`不能直接修改，要修改dns请在网卡配置文件添加字段`DNS1/DNS2/DNS3`,重载网卡配置文件后自动覆盖`/etc/resolv.conf`
`/etc/hostname`保存主机名
`/etc/sysconfig/network`可配置全局默认网关(默认空文件)

## 静态路由配置
`ip route list`     查看当前路由表
`ip route add 192.168.1.0/24 via 192.168.2.254` 临时添加一条静态路由条目
`ip route add 172.16.0.0/16 dev eth0`
`ip route del 10.0.0.0/8`   临时删除一条静态路由条目

**永久静态路由配置**
`/etc/sysconfig/static-routes` 不再使用
需要自行新建`/etc/sysconfig/network-scripts/route-INTERFACE`(INTERFACE为网卡名称) 添加以下内容
<pre><code class="language-bash line-numbers">192.168.2.0/24 via 10.0.0.254
192.168.1.0/24 dev eth0
# 重载配置文件生效,或重启后生效
</code></pre>

## `iptables`换成`firewalld`
[netfilter/iptables 详解](https://www.zfl9.com/iptables.html)
在CentOS/RHEL7中,防火墙默认是`firewalld`,特点是引入了zone的概念,并且支持动态管理
但是底层还是用的netfilter模块，并且zone的概念好麻烦，还是喜欢`iptables`这种简洁风
> 
注意`netfilter`是linux内核模块，工作在`内核空间`，之前centos6用的`用户空间`管理工具是`iptables`,centos7只是换成了`firewalld`. 并没有太多的变化

<pre><code class="language-bash line-numbers">yum -y install iptables-services    # 安装iptables-services
systemctl enable iptables
systemctl disable firewalld
</code></pre>
