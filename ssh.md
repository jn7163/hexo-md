---
title: ssh密钥登录
date: 2016-11-28 16:31:00
categories:
- linux
- 基础命令
- ssh
tags:
- linux
- 基础命令
- ssh
- ssh-copy-id
keywords: ssh, ssh-copy-id
---
> 通常我们登录linux都不是直接用用户名密码登录的，因为不安全，密码复杂度不够高，容易被暴力破解等等，那通过什么方法？ssh密钥对，非对称算法，而且可以免输密码登录，懒人党福音啊！

<!-- more -->

## 生成ssh密钥对
<pre><code class="language-bash line-numbers">ssh-keygen # 一路回车

将.ssh目录下的公钥复制到你要登陆的目标服务器上

### 1. ssh-copy-id 方式(推荐，会自动配置好相应的权限，避免各种权限问题)
ssh-copy-id root@192.168.255.254

### 2. scp 手动复制
scp .ssh/id_rsa.pub root@192.168.255.254:/root/.ssh/authorized_keys
# 修改权限
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 600 ~/.ssh/authorized_keys
</code></pre>

## 自动登录
<pre><code class="language-bash line-numbers">ssh root@192.168.255.254
</code></pre>

## ssh config配置
<pre><code class="language-bash line-numbers">发现每次都要输入用户及IP 还是不够方便
别担心，我们还有方法

vim .ssh/config
--- config ---
Host    CUST_NAME
    HostName    192.168.255.254
    Port        22
    User        root
    IdentityFile    ~/.ssh/id_rsa_http
--- config ---

ssh CUST_NAME # 快速登录成功！
</code></pre>

## ssh安全加固

**[ssh安全加固，防止暴力破解](https://www.zfl9.com/ssh_security.html)**

## ssh使用代理
有时候连接国外的vps时，因为某些原因，ssh连上了很卡，而且经常失去连接，如果你有https或者socks代理，我们就可以通过该代理连接ssh

`ssh -oProxyCommand="nc -X 5 -x 127.0.0.1:1080 %h %p" root@zfl9.com`    通过 socks5 代理
`ssh -oProxyCommand="nc -X connect -x 127.0.0.1:1080 %h %p" root@zfl9.com` 通过 http_connect 代理

同理，scp，sftp也能用这样的方法
`scp -Cpr -oProxyCommand="nc -X5 -x127.0.0.1:1080 %h %p" py3.7z root@zfl9.com:temp/`
`sftp -oProxyCommand="nc -X5 -x127.0.0.1:1080 %h %p" root@zfl9.com`

如果你使用的是 xshell5
也可以设置代理：`属性` -> `连接` -> `代理` -> `添加、选择代理服务器` -> `重新连接ssh`
