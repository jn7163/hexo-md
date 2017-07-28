---
title: ssh端口转发和ssh_tunnel科学上网
date: 2017-04-22 20:59:00
categories:
- linux
- 基础命令
- ssh
tags:
- linux
- ssh
- 基础命令
- ssh端口转发
- 科学上网
keywords: ssh, ssh_forward, 科学上网, ssh_tunnel代理, ssh端口转发
---
> 
我也是最近才知道ssh还可以用来科学上网的，以前一直以为ssh只是一个linux登录用的工具而已，太年轻

<!-- more -->

### 本地端口转发
<pre><code class="language-bash line-numbers"><script type="text/plain"># 命令格式
ssh -L <local_ip>:<local_port>:<dst_ip>:<dst_port> <user>@<remote_host>
如:
ssh -L 192.168.1.101:80:ip.cn:80 root@zfl9.com
ssh -L 80:ip.cn:80 root@zfl9.com        监听127.0.0.1 [::1]
ssh -L :80:ip.cn:80 root@zfl9.com       监听0.0.0.0 [::]
ssh -L *:80:ip.cn:80 root@zfl9.com      同上
ssh -L 0.0.0.0:ip.cn:80 root@zfl9.com   监听0.0.0.0
ssh -L \[::\]:80:ip.cn:80 root@zfl9.com 监听[::]

作用: 所有发往 local_ip:local_port 的数据都会经由 remote_host 转发至 dst_ip:dst_port
</script></code></pre>

### 远程端口转发(多用于穿透NAT，防火墙)
<pre><code class="language-bash line-numbers"><script type="text/plain"># 命令格式
ssh -R <remote_host_port>:<dst_ip>:<dst_port> <user>@<remote_host>
# 监听remote_host的 127.0.0.1 [::1] 地址

作用: remote_host监听127.0.0.1:port ::1:port，所有发往其监听地址的数据经由 本机 转发至 dst_ip:dst_port

举个最简单的应用:
我在公网有一台服务器zfl9.com
我的笔记本接的路由器(WLAN、有线) ip是内网ip，访问公网的方式是通过路由器的NAT进行的
我还有一台台式机也是接的路由器，http服务器，地址是192.168.1.111:80, 不能上外网

现在问题来了，我想在 zfl9.com 这台服务器上访问内网台式机的http服务，能实现吗?
1. 我在笔记本上使用ssh远程端口转发就行, ssh -R 8080:192.168.1.111:80 root@zfl9.com
2. 上面的命令的意思是，让zfl9.com监听本地回环口8080, 发往其8080端口的数据由我的笔记本来转发到192.168.1.111:80
3. 那么我只需要在 zfl9.com 上执行 curl http://127.0.0.1:8080 就能访问192.168.1.111:80的http服务了！
</script></code></pre>

### 动态端口转发(socks4/5代理)
<pre><code class="language-bash line-numbers"><script type="text/plain"># 命令格式
ssh -D <local_ip>:<local_port> <user>@<remote_host>
如:
ssh -D 80:ip.cn:80 root@zfl9.com        监听127.0.0.1 [::1]
ssh -D :80:ip.cn:80 root@zfl9.com       监听0.0.0.0 [::]
ssh -D *:80:ip.cn:80 root@zfl9.com      同上
ssh -D 0.0.0.0:ip.cn:80 root@zfl9.com   监听0.0.0.0
ssh -D \[::\]:80:ip.cn:80 root@zfl9.com 监听[::]

# 简单的socks5代理
ssh -D 9999 root@zfl9.com
解释: 本机监听端口9999, 然后chrome或者firefox通过代理插件 配置socks5代理，127.0.0.1:9999 即可科学上网
</script></code></pre>
