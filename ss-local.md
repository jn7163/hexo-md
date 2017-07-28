---
title: Linux使用shadowsocks代理(科学上网)
date: 2016-11-28 15:26:00
categories:
- 科学上网
tags:
- ss-local
- 科学上网
keywords: socks5, sslocal, privoxy, 全局代理, ss, ssr, 翻墙, 科学上网, gfw.action, gfwlist2privoxy
---
> 
在Linux上使用`shadowsocks`代理上网，主要有两种方式：
第一种：使用`ss-local + privoxy`的方式，然后设置shell的`http/https_proxy`环境变量即可，这种属于shell终端代理方式，优点是没有dns污染问题，但是要注意，有些应用不会遵循shell的proxy设置
第二种：使用`ss-redir + iptables`方式，然后用`ss-tunnel`、`pdnsd`等缓解dns污染，这种属于全局透明代理方式，而不是仅仅支持`http/https`这样的应用层级别的代理
今天介绍的是第一种，也就是shell终端形式的代理，支持http、https代理

<!-- more -->

## ss-redir透明代理(推荐)
在使用`ss-redir + iptables透明代理`之前，我一直以为`ss-local + privoxy`是所谓的全局代理，真是年轻啊
这种代理方式是通过shell的proxy设置来实现所谓的科学上网的，这种代理方式并不完美，很多软件都不听话，不走shell终端代理
所以我后来又写了**[ss-redir + iptables 全局透明代理](https://www.zfl9.com/ss-redir.html)**这篇博客
所以，如果你需要良好的科学上网体验，请猛戳**[ss-redir传送门](https://www.zfl9.com/ss-redir.html)**
如果你被dns污染所烦恼，请选择使用`ss-local + privoxy`方式，因为ss-local是直接将域名解析都交给ss服务器做的，不存在dns污染问题！

## 安装ss-local
shadowsocks有c语言版、go版、python版
这里推荐c、python这两者，如果你已经有了python环境，那么直接使用pip在线安装shadowsocks即可；
如果你希望shadowsocks的性能好，稳定，占用内存小，那么请用c语言版，并且这个版本更新非常频繁
<pre><code class="language-bash line-numbers">## python版本 - shadowsocks
# CentOS
yum -y install epel-release
yum -y install python-pip
pip install shadowsocks

# ArchLinux
pacman -S python-pip
pip install shadowsocks

## c版本 - shadowsocks-libev
# CentOS
需要编译安装：请参考 https://www.zfl9.com/ss-redir.html#编译shadowsocks-libev

# ArchLinux
pacman -S shadowsocks-libev
</code></pre>

## ss-local.json
<pre><code class="language-bash line-numbers">vim /etc/ss-local.json
{
    "server":"SERVER-IP",
    "server_port":PORT,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"PASSWORD",
    "timeout":300,
    "method":"aes-128-cfb",
    "fast_open": false,
    "workers": 1
}

## 配置说明：
{
    "server":"SERVER-IP",           # 服务器地址
    "server_port":PORT,             # 服务器端口
    "local_address": "127.0.0.1",   # 本地监听地址
    "local_port":1080,              # 本地监听端口
    "password":"PASSWORD",          # 密码
    "timeout":300,                  # 超时
    "method":"aes-128-cfb",         # 加密方式
    "fast_open": false,             # tcp_fastopen
    "workers": 1                    # worker数量
}
</code></pre>

## 启动ss-local
<pre><code class="language-bash line-numbers">## python版本
nohup sslocal -c /etc/ss-local.json &>> /var/log/ss-local.log &

## c版本
nohup ss-local -c /etc/ss-local.json &>> /var/log/ss-local.log &
</code></pre>

> 虽然socks5代理起来了，但是还不能直接用，需要privoxy转发一下

## 安装privoxy
<pre><code class="language-bash line-numbers">## CentOS
yum -y install privoxy

## ArchLinux
pacman -S --need privoxy
</code></pre>

## privoxy-全局代理
> 
所谓全局，就是全部流量走ss-local，不分国内国外
如果你需要这样的代理方式，那么请看下面的配置；
如果你仅仅需要被墙的网站走ss-local代理，没被墙的走直连，那么请跳过这段，看下一节`privoxy-gfwlist`

<pre><code class="language-bash line-numbers">## 配置
echo 'forward-socks5 / 127.0.0.1:1080 .' >> /etc/privoxy/config

## 启动
systemctl start privoxy
systemctl -l status privoxy
</code></pre>

## privoxy-gfwlist
> 
其实很多时候，我们只是想让那些被墙的网站走ss-local代理，而其他的不走代理，直连
这时候就需要借助gfwlist，简单的说，gfwlist里面记录的是几乎所有被墙的网站列表

**gfwlist2privoxy**
gfwList是由AutoProxy官方维护，由众多网民收集整理的一个中国大陆防火长城的屏蔽列表
因为这是Firefox浏览器直接使用的一种格式，要想用在privoxy上，就必须进行相应的转换
这里我提供一个shell自动转换的脚本，测试可用，我已经放在了github上-[gfwlist2privoxy](https://github.com/zfl9/gfwlist2privoxy/)
<pre><code class="language-bash line-numbers"><script type="text/plain">wget https://raw.github.com/zfl9/gfwlist2privoxy/master/gfwlist2privoxy && bash gfwlist2privoxy
# 然后输入127.0.0.1:1080，也就是你的ss-local监听地址和端口号

cp -af gfw.action /etc/privoxy/
echo 'actionsfile gfw.action' >> /etc/privoxy/config
systemctl start privoxy
systemctl -l status privoxy
</script></code></pre>

## 设置http/https_proxy
<pre><code class="language-bash line-numbers"># privoxy默认监听端口为8118
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
export no_proxy=localhost

# no_proxy是不经过privoxy代理的地址
只支持填写具体的host/ip，不能填网段、通配符，根据自己需要修改
</code></pre>

## 测试代理
<pre><code class="language-bash line-numbers">curl www.google.com
## 如果出现下面这段输出则代理成功！
&lt;HTML&gt;&lt;HEAD&gt;&lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html;charset=utf-8&quot;&gt;
&lt;TITLE&gt;302 Moved&lt;/TITLE&gt;&lt;/HEAD&gt;&lt;BODY&gt;
&lt;H1&gt;302 Moved&lt;/H1&gt;
The document has moved
&lt;A HREF=&quot;http://www.google.com.hk/url?sa=p&amp;amp;hl=zh-CN&amp;amp;pref=hkredirect&amp;amp;pval=yes&amp;amp;q=http://www.google.com.hk/%3Fgws_rd%3Dcr&amp;amp;ust=1480320257875871&amp;amp;usg=AFQjCNHg9F5zMg83aD2KKHHHf-yecq0nfQ&quot;&gt;here&lt;/A&gt;.
&lt;/BODY&gt;&lt;/HTML&gt;

curl ip.cn      # 查看ip：如果是全局代理，那么该ip应为ss服务器的ip; 如果是gfwlist方式，该ip则为本机ip
</code></pre>

## ss-local服务脚本
因为这样一条一条命令太麻烦，重启后得重新启动ss-local和设置相应的环境变量
所以这里提供一个ss-local的服务脚本，一键启动、关闭ss-local、privoxy

在进行下面的步骤之前，请确保你已安装ss-local并正确配置了ss-local.json文件、privoxy配置文件

**python版本 - sslocal**
**myss.sh** `chmod +x myss.sh`、`ln -sf $(pwd)/myss.sh /usr/local/bin/myss`
<pre><code class="language-bash line-numbers">#!/bin/bash

case $1 in
start)
    nohup sslocal -c /etc/ss-local.json &>> /var/log/ss-local.log &
    systemctl start privoxy
    export http_proxy=http://127.0.0.1:8118
    export https_proxy=http://127.0.0.1:8118
    export no_proxy=localhost
    ;;
stop)
    unset http_proxy https_proxy no_proxy
    systemctl stop privoxy
    pkill sslocal
    ;;
reload)
    pkill sslocal
    nohup sslocal -c /etc/ss-local.json &>> /var/log/ss-local.log &
    ;;
set)
    export http_proxy=http://127.0.0.1:8118
    export https_proxy=http://127.0.0.1:8118
    export no_proxy=localhost
    ;;
unset)
    unset http_proxy https_proxy no_proxy
    ;;
*)
    echo 'usage start|stop|reload|set|unset'
    exit 1
    ;;
esac
</code></pre>

**c版本 - ss-local**
**myss.sh** `chmod +x myss.sh`、`ln -sf $(pwd)/myss.sh /usr/local/bin/myss`
<pre><code class="language-bash line-numbers">#!/bin/bash

case $1 in
start)
    nohup ss-local -c /etc/ss-local.json &>> /var/log/ss-local.log &
    systemctl start privoxy
    export http_proxy=http://127.0.0.1:8118
    export https_proxy=http://127.0.0.1:8118
    export no_proxy=localhost
    ;;
stop)
    unset http_proxy https_proxy no_proxy
    systemctl stop privoxy
    pkill ss-local
    ;;
reload)
    pkill ss-local
    nohup ss-local -c /etc/ss-local.json &>> /var/log/ss-local.log &
    ;;
set)
    export http_proxy=http://127.0.0.1:8118
    export https_proxy=http://127.0.0.1:8118
    export no_proxy=localhost
    ;;
unset)
    unset http_proxy https_proxy no_proxy
    ;;
*)
    echo 'usage start|stop|reload|set|unset'
    exit 1
    ;;
esac
</code></pre>

**配置命令别名alias** `vim /etc/profile.d/myss.sh`
<pre><code class="language-bash line-numbers"><script type="text/plain">alias myss.start='. myss start'
alias myss.stop='. myss stop'
alias myss.set='. myss set'
alias myss.unset='. myss unset'
alias myss.reload='. myss reload'
</script></code></pre>

**加载alias别名文件** `source /etc/profile`

**用法**
`myss.start`    启动shadowsocks
`myss.stop`     停用shadowsocks
`myss.set`      设置http/https/ftp_proxy环境变量
`myss.unset`    取消http/https/ftp_proxy环境变量
`myss.reload`   重载ss-local.json配置文件

ss-local运行日志：`/var/log/ss-local.log`
