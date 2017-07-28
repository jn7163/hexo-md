---
title: ssh安全加固
date: 2017-05-06 12:59:00
categories:
- linux
- 基础命令
- ssh
tags:
- linux
- 基础命令
- ssh安全加固
keywords: ssh安全加固, ssh安全配置
---

> 
服务器好久没登了，今天偶然查看了一下`/var/log/secure`日志，握日，什么情况？
日志文件大小`1.8M`！，有问题啊，果然，5000多条ssh认证失败的记录！
不过没有一个登录成功的，因为我一直都是只允许通过密钥登录的，想要破解ssh密钥还是有点难度的

<!-- more -->

## ssh密钥登录

**[生成ssh密钥，配置密钥登录，禁止密码方式认证](https://www.zfl9.com/ssh.html)**

## sshd_config
<pre><code class="language-bash line-numbers"><script type="text/plain">## 先从配置文件开始 /etc/ssh/sshd_config
Port 2333               # ssh监听端口，默认22，很不安全，建议改为高位端口
ListenAddress 0.0.0.0   # ssh监听地址
Protocol 2              # ssh协议版本，现在都是版本2了

SyslogFacility AUTHPRIV

PermitRootLogin without-password    # root用户只能通过ssh密钥登录(不过最好是禁止root登录)
MaxAuthTries 3                      # 认证重试次数；如果连续3次没认证成功，则锁定

PubkeyAuthentication yes                    # 开启ssh公钥认证(默认)
AuthorizedKeysFile  .ssh/authorized_keys    # ssh公钥存放路径
PermitEmptyPasswords no                     # 禁止空密码认证
PasswordAuthentication no                   # 禁止密码认证方式

AllowUsers root zfl9     # 只允许特定用户登录(空格分隔)

ChallengeResponseAuthentication no
GSSAPIAuthentication no
GSSAPICleanupCredentials no

UseDNS no
UsePAM yes

X11Forwarding no        # X11转发
X11UseLocalhost no

UsePrivilegeSeparation sandbox

Subsystem   sftp    /usr/libexec/openssh/sftp-server    # sftp子系统
</script></code></pre>

## iptables_recent
<pre><code class="language-bash line-numbers"><script type="text/plain">## 按照上面的配置，其实已经没什么大问题了，但是如果希望更稳固一点，可以用iptables的recent模块
iptables -A INPUT -p icmp --icmp-type 8 -m length --length 100 -m recent --name sshkey --rsource --set -j ACCEPT
iptables -A INPUT -p tcp --dport 2333 -m state --state NEW -m recent --name sshkey --rcheck --rsource --seconds 15 -j ACCEPT
iptables -A INPUT -p tcp --dport 2333 -m state --state NEW -j DROP

## recent模块
可以理解为一张记录IP地址的列表
'--set' '--remove'      添加、删除IP
'--rsource' '--rdest'   记录源地址(默认)、目标地址
'--rcheck' '--update'   检查地址是否在列表中
'--seconds'             时间条件
'--hitcount'            命中次数

## 放在上面的例子上：
1. 第一条，记录长度为128字节(100字节加上ip头,icmp头28字节)的icmp_request包的源地址，命名为sshkey
2. 第二条，允许sshkey列表中的地址 15秒内可以登录ssh
3. 第三条，其他的ssh握手连接全部丢弃

这样，普通的ssh连接就不能连接了，就算是密钥也不行，那要怎么连接呢？
我们要先给服务器打声招呼，也就是先ping一个128长度的包给服务器，这时我们就有15秒的时间可以登录了

## "sshkey开锁"
ping -c 1 -s 100 ip   # linux
ping -n 1 -l 100 ip   # windows

## 登录ssh
ssh -p 2333 user@ip     # 15s内

## rcheck update区别
# rcheck从第1个包开始计算时间;
# update在rcheck的基础上增加了从最近的DROP包开始计算阻断时间, 具有准许时间和阻断时间, 会更新last-seen时间戳;

- rcheck类似活动抽奖: 
每天有3次抽奖机会，随便你抽，抽完3次，抱歉，等明天;

- update类似网银: 
连续输错密码超过3次，不好意思，冻结24小时，24小时后解锁，如果解锁后的一个小时内又连续输错3次，又得等24小时了;
</script></code></pre>
