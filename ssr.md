---
title: SSR一键安装脚本(转自91yun)
date: 2016-10-23 16:22:22
categories:
- 科学上网
tags:
- 科学上网
- shadowsocksR
keywords: ssr, vps, 科学上网, ss, 翻墙, 代理, socks5
---
> 
Shadowsocks原版在更新到 v2.5.8 之后被“相关部门”约谈喝茶了，于是就停止了更新
但是应网友要求，另一个开发者把 v2.5.8 的一些严重BUG修复了更新为 v3.0，更新一度停滞.......
而ShadowsocksR是在原版作者喝茶前，由另一个程序员 [breakwa11](https://github.com/breakwa11) 制作的第三方版本，主要特点是增加了一些人性化功能，比如服务器连接统计、连接管理、协议转换、多重代理等

<!-- more -->

### linux下使用ss全局代理
**[`ss-local + privoxy` 终端http/https代理](https://www.zfl9.com/ss-local.html)**
**[`ss-redir + iptables` 全局透明代理](https://www.zfl9.com/ss-redir.html)**

### 安装SSR(转自91yun)
[91yun](https://www.91yun.org) (2017-02-25更新)

<pre><code class="language-bash line-numbers"># 默认加密为 "chacha20"
# 默认混淆协议为 "auth_sha1_v4"
# 默认混淆方式为 "tls1.2_ticket_auth"
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/shadowsocks_install/master/shadowsocksR.sh && bash shadowsocksR.sh
</code></pre>

### shadowsocks.json
<pre><code class="language-bash line-numbers">vim /etc/shadowsocks.json
{
    "server": "0.0.0.0",
    "server_ipv6": "::",
    "server_port": 8989,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "password",
    "timeout": 120,
    "udp_timeout": 60,
    "method": "chacha20",
    "protocol": "origin",
    "protocol_param": "",
    "obfs": "plain",
    "obfs_param": "",
    "dns_ipv6": false,
    "connect_verbose_info": 0,
    "redirect": "",
    "fast_open": false,
    "workers": 1
}

## 配置说明：
{
    "server": "0.0.0.0",            # 监听地址ipv4
    "server_ipv6": "::",            # 监听地址ipv6
    "server_port": 8989,            # 监听端口
    "local_address": "127.0.0.1",   # 本地监听地址
    "local_port": 1080,             # 本地监听端口
    "password": "password",         # password密码
    "timeout": 120,                 # 超时
    "udp_timeout": 60,              # udp超时
    "method": "chacha20",           # 加密方式
    "protocol": "origin",           # 协议插件,origin为不混淆
    "protocol_param": "",           # 协议插件参数
    "obfs": "plain",                # 混淆插件,plain为不混淆
    "obfs_param": "",               # 混淆插件参数
    "dns_ipv6": false,              # 启用ipv6的dns
    "connect_verbose_info": 0,      # verbose信息
    "redirect": "",                 # redirect
    "fast_open": false,             # 需要内核支持(3.7+),在tcp握手的同时也交换数据,显著降低延迟
    "workers": 1                    # worker数量
}

## 多用户配置：
1. 删除password行
2. 替换为：
"port_password":{
"80":"password1",
"443":"password2"
},
</code></pre>

### ssr混淆
[github-obfs-wiki](https://github.com/breakwa11/shadowsocks-rss/wiki/obfs)

<pre><code class="language-bash line-numbers">### 工作原理
C -> S 方向
浏览器请求(socks5协议) -> ssr客户端 -> 协议插件(转为指定协议) -> 加密 -> 混淆插件(转为表面上看起来像http/tls) ->
ssr服务端 -> 混淆插件(分离出加密数据) -> 解密 -> 协议插件(转为原协议) -> 转发目标服务器

其中，协议插件主要用于增加数据完整性校验，增强安全性，包长度混淆等
混淆插件主要用于伪装为其它协议

### 推荐
如不考虑原版的情况下，推荐使用的协议，只有auth_sha1_v4和auth_aes128_md5和auth_aes128_sha1
推荐使用的混淆只有plain,http_simple,http_post,tls1.2_ticket_auth
</code></pre>
