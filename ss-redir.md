---
title: ss-redir透明代理
date: 2017-04-29 12:04:00
categories:
- 科学上网
tags:
- 科学上网
- ss-redir透明代理
keywords: ss-redir透明代理, ss-redir iptables 全局代理, ss-tunnel dns污染
---
> 
[shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev) 提供了 `ss-local` `ss-redir` `ss-tunnel` `ss-server`;
其中`ss-local`是ss的客户端，提供了socks5代理，但是一般只有浏览器才能用socks5代理，要想"充当"全局代理可以通过`privoxy`
注意，这里说的是"充当"全局代理，因为`ss-local + privoxy`方式并不是真正意义上的全局代理，只不过是设置了终端的`http/https`代理罢了
那有没有办法做到真正的全局透明代理？那就是今天的主角：`ss-redir + iptables`，`ss-redir`目前只有shadowsocks-libev版本才有，python和go版暂时没有

<!-- more -->

## ss-local、ss-redir、ss-tunnel、ss-server
这里简要说明一下ss-local、ss-redir、ss-tunnel、ss-server这四者的区别：
- `ss-server`：
很明显，这是用来作shadowsocks服务端的一个程序，没什么好说的
- `ss-local`：
local，顾名思义，本地运行的socks5客户端，是一个提供socks5代理的程序
注意：这个socks5代理和http代理在概念上没有什么两样
只不过对于http代理，目前绝大多数的软件和系统都完美的支持了
你可以随便找个软件，都可以找到一个http_proxy的代理选项，windows和linux系统也都支持http代理
但是对于socks5代理，目前只有部分浏览器支持和少数软件支持，最典型的就是浏览器，可以直接使用此种代理
不过，我们可以在我们的应用软件和socks5代理(也就是ss-local)之间再添加一层代理，那这层代理的作用是什么呢？
很简单，就是使用类似privoxy这样的代理软件，将来自应用程序的普通http代理转发到提供socks5代理的ss-local上
这样我们就可以在应用软件上使用http代理，将其proxy_server设置为privoxy的地址和端口，然后再配置privoxy将其转发至ss-local
如果你对这种代理方式有兴趣，那么你可以看我之前的这篇博文 - **[ss-local + privoxy终端代理](https://www.zfl9.com/ss-local.html)**
- `ss-redir`：
redirect，也就是重定向，也是在本地运行的程序，不过它可以提供透明代理！
先来解释一下ss-redir的透明代理：
1. 首先通过iptables将本机的数据包全部重定向至ss-redir的监听地址
2. 当上层应用发出数据包后，首先被iptables重定向至ss-redir，ss-redir将数据包原原本本的打包好，并加密，通过与ss服务器建立的ss通道，将其发送至ss服务器
3. 然后ss-server接收到此包，先解密，然后根据数据包中的目标ip地址，将其发送至目标服务器
4. 目标服务器做出应答后，将应答数据返回给ss-server，然后ss-server通过ss通道，将其传回ss-redir，最后ss-redir返回上层应用

从上面可以看出，发出的数据包的目标ip地址和端口port始终没有变，始终还是上层应用指定的目标ip和端口port
- `ss-tunnel`：
这个和ssh的本地端口转发颇为相似，首先ss-tunnel在本地监听一个端口（假设为127.0.0.1:1053）
那么发往127.0.0.1:1053的数据包，ss-tunnel会通过与ss服务器建立的ss通道，将其发往ss服务器
ss服务器根据ss-tunnel设置的远程地址和端口（比如8.8.8.8:53），将数据包发往8.8.8.8:53
8.8.8.8:53响应后，返回数据给ss服务器，ss服务器再将它返回给ss-tunnel，最后ss-tunnel将其返回给上层应用

对比ss-redir，ss-tunnel的目标地址其实是发生了改变的，开始的目的地址是127.0.0.1:1053
然后经过ss-tunnel发往ss服务器后，目的地址变为了8.8.8.8:53，然后再将此数据包发给8.8.8.8，最后返回

**ss-local和ss-redir优缺点对比**
ss-local配合privoxy可以做到shell终端方式的http、https代理；
再配合gfw.action，可以只代理被墙的网站，而其他正常的网站走直连；
最主要的一点是：ss-local不存在dns污染问题，因为dns默认是远程解析，即ss服务器帮你解析dns

而ss-redir配合iptables、ipset，也能做到只代理国外的地址，大陆的直连；
并且，不只是支持http、https这两种协议的代理，理论能够代理所有TCP、UDP的流量
不过，ss-redir相比ss-local的缺点是，ss-redir需要额外解决dns污染问题

**再谈谈dns污染问题如何解决**
第一种：ss-redir转发tcp、udp流量(包括dns)，这时`系统dns设置为8.8.8.8`等国外公共dns，dns查询走ss通道，由于dns还是走的udp(非明文)，在某些网络环境下udp丢包会较严重(比如我大电信)

第二种：ss-redir转发tcp、udp流量(除了dns)，用ss-tunnel来建立udp转发，将dns通过ss通道，转发至8.8.8.8:53等上游公共dns，此时的dns解析依旧是udp方式(非明文)，某些网络环境下udp丢包会较严重`(系统dns设置为127.0.0.1)`

第三种：ss-redir转发tcp、udp流量(除了dns)，通过pdnsd之类的软件，将udp/53方式的dns查询转换为tcp方式，然后再将该tcp方式的dns查询数据通过ss-redir的tcp转发，将其发送至目标dns服务器，完成解析`(系统dns设置为127.0.0.1)`

第四种：ss-redir转发tcp、udp流量(除了dns)，通过dnscrypt之类的软件，将udp明文方式的dns查询加密，然后再发送给支持该加密方式的上游公共dns服务器，完成解析，这种udp方式的dns查询和传统的明文方式查询的区别，就如同http和https的区别`(系统dns设置为127.0.0.1)`

具体选择哪种方式，随你自己，求速度就用udp方式查询dns，注意该udp方式的dns查询是加密了的哦，不是普通的udp/53明文方式，明文方式已经被gfw玩坏了，只能使用加密的！不过某些网络下，udp丢包较严重，还有一点注意，如果是通过ss转发udp的dns，记得检查ss服务商有没有开启udp转发，一般付费的服务商都是带有udp转发的，免费的话就少了
如果你不是那么的追求速度，而是想保障dns的查询可靠度，那么就选择tcp方式的dns查询吧！

## ss-redir环境部署说明
本文使用的是：
`ss-redir`转发tcp和除了dns的udp、`ss-tunnel`作上游dns完成dns解析，`chinadns`用来分流大陆和国外dns解析，大陆的直接向114.114.114.114此类dns查询，国外的向上游的`ss-tunnel`查询dns，做到智能分流

我使用的系统是 CentOS7.3(vps) 和 ArchLinux for armv7(树莓派3b)

## 编译shadowsocks-libev
<pre><code class="language-bash line-numbers">git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
git submodule update --init --recursive

# shadowsocks-libev依赖libev
yum -y install libevent libevent-devel libev libev-devel
yum -y install gcc gettext autoconf libtool automake make pcre-devel asciidoc xmlto udns-devel

# 安装 Libsodium
export LIBSODIUM_VER=1.0.12
wget https://download.libsodium.org/libsodium/releases/libsodium-$LIBSODIUM_VER.tar.gz
tar xvf libsodium-$LIBSODIUM_VER.tar.gz
pushd libsodium-$LIBSODIUM_VER
./configure --prefix=/usr && make
sudo make install
popd
sudo ldconfig

# 安装 MbedTLS
export MBEDTLS_VER=2.4.2
wget https://tls.mbed.org/download/mbedtls-$MBEDTLS_VER-gpl.tgz
tar xvf mbedtls-$MBEDTLS_VER-gpl.tgz
pushd mbedtls-$MBEDTLS_VER
make SHARED=1 CFLAGS=-fPIC
sudo make DESTDIR=/usr install
popd
sudo ldconfig

# 编译shadowsocks-libev
./autogen.sh && ./configure && make
sudo make install
</code></pre>

## 启动ss-redir
<pre><code class="language-bash line-numbers">nohup ss-redir -s 'server_ip' -p 'server_port' -m 'method' -k 'passwd' -u -b 'bind_ip' -l 'listen_port' -f 'pid_file' -v &>> /var/log/ss-redir.log &
-s ss服务器ip
-p ss服务器端口
-m 加密方式
-k 密码
-u 启用udp转发
-b 本地监听ip
-l 本地监听端口
-f pid_file
-v verbose info

# 如:
ss-redir -s ss.org -p 8989 -m rc4-md5 -k 'password' -u -b 0.0.0.0 -l 1080 -f '/run/ss-redir.pid' -v
</code></pre>

> ss-redir是起来了，但是还要配置iptables规则

## 配置iptables
<pre><code class="language-bash line-numbers"># 获取大陆ip地址段
curl -sL http://f.ip.cn/rt/chnroutes.txt | egrep -v '^$|^#' > cidr_cn

# ipset
yum -y install ipset ipset-libs

ipset -N cidr_cn hash:net
for i in `cat cidr_cn`; do echo ipset -A cidr_cn $i >> ipset.sh; done
chmod +x ipset.sh && ./ipset.sh
ipset -S > /etc/sysconfig/ipset.cidr_cn # 持久化ipset地址列表至文件

# 新建一条链 shadowsocks
iptables -t nat -N shadowsocks

# 保留地址、私有地址、回环地址 不走代理
iptables -t nat -A shadowsocks -d 0/8 -j RETURN
iptables -t nat -A shadowsocks -d 127/8 -j RETURN
iptables -t nat -A shadowsocks -d 10/8 -j RETURN
iptables -t nat -A shadowsocks -d 169.254/16 -j RETURN
iptables -t nat -A shadowsocks -d 172.16/12 -j RETURN
iptables -t nat -A shadowsocks -d 192.168/16 -j RETURN
iptables -t nat -A shadowsocks -d 224/4 -j RETURN
iptables -t nat -A shadowsocks -d 240/4 -j RETURN

# 发往ss服务器的数据不走代理，否则陷入死循环
iptables -t nat -A shadowsocks -d SOCKS_SERVER -j RETURN    # 替换SOCKS_SERVER为你的ss服务器ip/域名

# 大陆地址不走代理
iptables -t nat -A shadowsocks -m set --match-set cidr_cn dst -j RETURN

# 其余的全部重定向至ss-redir监听端口1080(端口号随意,统一就行)
iptables -t nat -A shadowsocks ! -p icmp -j REDIRECT --to-ports 1080

# OUTPUT链添加一条规则，重定向至shadowsocks链
iptables -t nat -A OUTPUT ! -p icmp -j shadowsocks

# 持久化iptables规则到文件
iptables-save > /etc/sysconfig/iptables.shadowsocks
</code></pre>


注意，如果你需要使用 ss-redir 充当路由器网关，那么请再添加这几条 iptables 规则：
在后面的一键脚本中，没有添加这些规则，因为笔者使用的环境中，并不需要ss-redir当网关，如果你需要，那么也请自行添加！
<pre><code class="language-bash line-numbers"><script type="text/plain"># 内网流量流经 shadowsocks 规则链
iptables -t nat -A PREROUTING -s 192.168/16 -j shadowsocks

# 开启内核ipv4的ip_forward转发
## 查看是否已开启：
cat /proc/sys/net/ipv4/ip_forward   # 如果值为1，那么表示已开启转发功能
## 若为0，则进行此步骤：
echo 1 > /proc/sys/net/ipv4/ip_forward  # 添加到开机脚本中，或者在/etc/sysctl.conf中添加对应的条目

# 内网流量源NAT
iptables -t nat -A POSTROUTING -s 192.168/16 -j MASQUERADE
</script></code></pre>

## dns污染
> 
dns被gfw污染了，只要检测到特殊照顾对象，就dns投毒，直接返回一个错误的ip给你
所以我们还需要解决dns污染问题，我们选择最简单的`ss-tunnel`
`ss-tunnel`需要ss服务器开启udp转发，一般付费的ss服务商都是带udp转发的
`ss-tunnel`其实和`ssh local_port forward`类似，了解过ssh的本地端口转发的同学应该很容易理解其原理
就是在本地监听一个端口(比如53)，ss服务器通过shadowsocks链路，将来自本地53端口的数据转发到指定的国外dns，免受gfw影响

<pre><code class="language-bash line-numbers">nohup ss-tunnel -s 'server_ip' -p 'server_port' -m 'method' -k 'passwd' -u -b 'bind_ip' -l 'listen_ip' -L '目标dns服务器' -f 'pid_file' -v &>> /var/log/ss-tunnel.log &
# 其中最重要的是 -L 参数指定的dns服务器，建立起ss-tunnel隧道后，只要将本机dns地址指向127.0.0.1
# dns查询数据包就会被ss-tunnel转发到这个指定的dns服务器进行域名解析，然后通过ss-tunnel返回结果

# 如:
ss-tunnel -s ss.org -p 8989 -m rc4-md5 -k 'passwd' -u -b 0.0.0.0 -l 53 -L '8.8.8.8:53' -f '/run/ss-tunnel.pid' -v

然后修改本机dns服务器，指向127.0.0.1
/etc/resolv.conf
nameserver 127.0.0.1

# 这里说一个关于dns的坑，如果你的ss-redir或ss-tunnel中的ss服务器填的是域名而不是ip地址
# 那么，请注意，在dns文件上临时加上 nameserver 114.114.114.114，不然你的ss-redir和ss-tunnel将启动失败，因为无法解析到ip
# 在ss-redir和ss-tunnel启动完成后，方可删除该项，如果你觉得太麻烦，那么请全部使用ip，不要用域名！
# 当然也可以将ss服务器的域名及ip地址加入本机host文件

# 再进行测试
dig www.google.com
dig www.google.co.jp
dig www.twitter.com
dig www.facebook.com

curl -sL https://www.google.com
curl -sL https://www.google.com.hk
curl -sL https://www.facebook.com
curl -sL https://www.baidu.com

没出意外应该是可以解析的
</code></pre>

## chinadns
> 
虽然解决了dns污染，不过这样的话，所有的dns请求都被转发到8.8.8.8去解析，对于国内的域名又是饶了一大圈
能不能在ss-tunnel转发dns之前把dns解析请求进行分流，大陆的直连114.114.114.114，其他的走8.8.8.8，做到速度的兼顾?
[chinadns](https://github.com/shadowsocks/ChinaDNS)就是这样一个东西，大致原理就是：
首先`chinadns`需要配置至少一个国内dns服务器，至少一个国外dns服务器
然后当dns请求到chinadns这里时，chinadns同时向两个dns服务器发送解析请求
如果国内dns服务器返回的ip不是大陆地址段(chinadns内置一张大陆地址段ip表)，则说明该域名很可能受到污染，这时就选择国外dns服务器返回的ip地址为准，反之则以国内dns服务器返回的地址为解析结果

<pre><code class="language-bash line-numbers"># 编译chinadns
wget https://github.com/shadowsocks/ChinaDNS/releases/download/1.3.2/chinadns-1.3.2.tar.gz
tar xf chinadns-1.3.2.tar.gz
cd chinadns-1.3.2
./configure && make
cp -af src/chinadns /usr/local/bin/chinadns
mkdir -p /etc/chinadns/
cp -af chnroute.txt /etc/chinadns/
cp -af iplist.txt /etc/chinadns/

# 我们先把ss-tunnel的监听端口改为1053
ss-tunnel -s ss.org -p 8989 -m rc4-md5 -k 'passwd' -u -b 0.0.0.0 -l 1053 -L '8.8.8.8:53' -f '/run/ss-tunnel.pid' -v

# 启动chinadns
chinadns -c /etc/chinadns/chnroute.txt -l /etc/chinadns/iplist.txt -b 0.0.0.0 -p 53 -s '114.114.114.114,127.0.0.1:1053' -d -v
指定国内dns服务器: 114.114.114.114
指定ss-tunnel国外dns服务器: 127.0.0.1:1053 (8.8.8.8:53)
这样就做到了dns的分流

# 测试
dig www.baidu.com
dig www.google.com

然后查看chinadns的日志，可以发现
解析www.baidu.com时，chinadns会把127.0.0.1:1053的返回结果给filter过滤掉
而解析www.google.com时，chinadns会把114.114.114.114的解析结果给filter

# 这个时候还可以进一步优化，使用dnsmasq进行dns缓存，进一步加快dns解析速度，篇幅关系就不详细说了
# 有兴趣的可以试试
# 其实openwrt等路由器中的智能代理也是通过 ss-redir + iptables + chinadns/pdnsd/dnsmasq 实现的
</code></pre>

## 总结
> 
如果你觉得前面的篇幅太长，不想看，你也可以直接使用这里提供的shell一键脚本

### 安装shadowsocks-libev
- RHEL、CentOS用户请参考前面的[编译安装shadowsocks-libev](#编译shadowsocks-libev)
- ArchLinux用户请直接使用`pacman -S shadowsocks-libev`
- 其他Linux系统请参考[github_shadowsocks-libev安装方法](https://github.com/shadowsocks/shadowsocks-libev#installation)

### 安装chinadns
<pre><code class="language-bash line-numbers"><script type="text/plain">wget https://github.com/shadowsocks/ChinaDNS/releases/download/1.3.2/chinadns-1.3.2.tar.gz
tar xf chinadns-1.3.2.tar.gz
cd chinadns-1.3.2
./configure && make
cp -af src/chinadns /usr/local/bin/chinadns
mkdir -p /etc/chinadns/
cp -af chnroute.txt /etc/chinadns/
cp -af iplist.txt /etc/chinadns/
</script></code></pre>

### shell脚本
**ipset_chinaip** `chmod +x ipset_chinaip`(如果没有安装ipset，请先安装ipset)
<pre><code class="language-bash line-numbers"><script type="text/plain">#!/bin/bash

cd $(cd $(dirname $0); pwd)
rm -fr ipset.chinaip.*

echo 'create chinaip hash:net' > ipset.chinaip.`date +%F_%T`

for ip in `curl -sL http://f.ip.cn/rt/chnroutes.txt | egrep -v '^\s*$|^\s*#'`; do
    echo "add chinaip $ip" >> ipset.chinaip.*
done
</script></code></pre>

**iptables_shadowsocks** `chmod +x iptables_shadowsocks`
<pre><code class="language-bash line-numbers"><script type="text/plain">#!/bin/bash

cd $(cd $(dirname $0); pwd)
rm -fr iptables.shadowsocks.*

cat << EOF > iptables.shadowsocks.`date +%F_%T`
*nat
:shadowsocks -
-A shadowsocks -d 0/8 -j RETURN
-A shadowsocks -d 127/8 -j RETURN
-A shadowsocks -d 10/8 -j RETURN
-A shadowsocks -d 169.254/16 -j RETURN
-A shadowsocks -d 172.16/12 -j RETURN
-A shadowsocks -d 192.168/16 -j RETURN
-A shadowsocks -d 224/4 -j RETURN
-A shadowsocks -d 240/4 -j RETURN
-A shadowsocks -d $1 -j RETURN
-A shadowsocks -m set --match-set chinaip dst -j RETURN
-A shadowsocks ! -p icmp -j REDIRECT --to-ports $2
-A OUTPUT ! -p icmp -j shadowsocks
COMMIT
EOF

if [ ! -e iptables.nat.clean ]; then
    echo -e "*nat\nCOMMIT" > iptables.nat.clean
fi
</script></code></pre>

**shadowsocks.sh** `chmod +x shadowsocks.sh`
请注意在脚本中填写你的ss服务商的相关信息！
<pre><code class="language-bash line-numbers"><script type="text/plain">#!/bin/bash

ss_server=SS服务器地址
ss_port=SS服务器端口
ss_method=SS服务器加密方式
ss_pass=SS服务器密码
listen_port=1080
dns_listen_port=1053
dns_upstream=114.114.114.114,127.0.0.1:${dns_listen_port}

case $1 in
start)
    nohup ss-redir -s ${ss_server} -p ${ss_port} -m ${ss_method} -k ${ss_pass} -u -b '0.0.0.0' -l ${listen_port} -v &>> /var/log/ss-redir.log &
    nohup ss-tunnel -s ${ss_server} -p ${ss_port} -m ${ss_method} -k ${ss_pass} -u -b '0.0.0.0' -l ${dns_listen_port} -L '8.8.8.8:53' -v &>> /var/log/ss-tunnel.log &
    nohup chinadns -c /etc/chinadns/chnroute.txt -l /etc/chinadns/iplist.txt -b 0.0.0.0 -p 53 -s ${dns_upstream} -d -v &>> /var/log/chinadns.log &
    ipset -R < $(cd $(dirname $0); pwd)/ipset.chinaip.*
    iptables-restore < $(cd $(dirname $0); pwd)/iptables.shadowsocks.*
    ;;
stop)
    iptables-restore < $(cd $(dirname $0); pwd)/iptables.nat.clean
    ipset -X chinaip &> /dev/null
    kill -9 `ps -ef | egrep '\schinadns\s' | egrep -v 'grep' | awk '{print $2}'` &> /dev/null
    kill -9 `ps -ef | egrep '\sss-redir\s' | egrep -v 'grep' | awk '{print $2}'` &> /dev/null
    kill -9 `ps -ef | egrep '\sss-tunnel\s' | egrep -v 'grep' | awk '{print $2}'` &> /dev/null
    ;;
*)
    echo "usage: $(cd $(dirname $0); pwd)/shadowsocks.sh start|stop"
    exit 1
    ;;
esac
</script></code></pre>

### 用法
1. 首次运行请先执行`./ipset_chinaip`、`./iptables_shadowsocks SS_IP LISTEN_PORT`(如: ./iptables_shadowsocks 1.2.3.4 1080)
端口号建议统一：ss-redir:1080、ss-tunnel:1053、chinadns:53
2. 设置dns解析，`/etc/resolv.conf`，dns为`127.0.0.1`，注意某些系统在重启后会自动更改dns设置，请注意该问题！
3. 启用shadowsocks：`./shadowsocks.sh start`
4. 关闭shadowsocks：`./shadowsocks.sh stop`
5. 相关日志：`/var/log/ss-redir.log`、`/var/log/ss-tunnel.log`、`/var/log/chinadns.log`
6. 测试是否全局代理成功：`curl -sL www.google.com`
7. 查看当前IP：`curl -sL ip.cn`
8. 更新大陆ip段：`./ipset_chinaip`，然后重启shadowsocks：`./shadowsocks.sh stop && ./shadowsocks.sh start`
9. 更新iptables规则：`./iptables_shadowsocks SS_IP LISTEN_PORT`，然后重启shadowsocks：`./shadowsocks.sh stop && ./shadowsocks.sh start`

关于第2点，设置dns的，请尽量在ss-redir和ss-tunnel中填写ss服务器的ip，不要填域名！
不然会因为解析不到ss服务器的ip，导致ss-redir和ss-tunnel启动失败！

如果你不得已使用域名，那么请在启动ss-redir和ss-tunnel之前，添加一个国内的公共dns服务器，完成ss服务器的ip解析
等ss-redir和ss-tunnel启动完毕后，再注释掉该国内公共dns服务器；
或者直接将ss服务器的域名及ip映射信息添加到本机的host文件；
