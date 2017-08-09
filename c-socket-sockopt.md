---
title: c语言 - socket编程(三)
date: 2017-08-09 15:34:00
categories:
- c
tags:
- c
keywords: c语言 socket编程 setsockopt getsockopt
---

> 
c语言 - socket编程(三)，tcp_keepalive保持长连接，允许程序使用还在TIME_WAIT状态的端口

<!-- more -->

## getsockopt、setsockopt
`int getsockopt(int socket, int level, int optname, void *optval, socklen_t *optlen);`：获取套接字属性
- `socket`：输入参数，要操作的socket套接字
- `level`：输入参数，协议层：`SOL_SOCKET`通用套接字选项、`IPPROTO_IP`IP选项、`IPPROTO_TCP`TCP选项
- `optname`：输入参数，访问的选项名
- `optval`：输出参数，保存选项的值
- `optlen`：输入参数，指明optval的长度
- 返回值：成功返回0，失败返回-1，并设置errno

`int setsockopt(int socket, int level, int optname, const void *optval, socklen_t optlen);`：设置套接字属性
- `socket`：输入参数，要操作的socket套接字
- `level`：输入参数，协议层，同getsockopt()
- `optname`：输入参数，访问的选项名
- `optval`：输入参数，要设置的值
- `optlen`：输入参数，指明optval的长度
- 返回值：成功返回0，失败返回-1，并设置errno

常用的socket选项：
![socket选项](/images/sockopt_1.jpg)
![socket选项](/images/sockopt_2.jpg)

## 重用TIME_WAIT状态的端口
一般来说，一个端口释放后会等待两分钟之后才能再被使用，这段时间主要消耗在了`TIME_WAIT`状态上；
正常情况下只有当socket的状态为`CLOSED`的时候才能被重新利用，不过我们可以使用`SO_REUSEADDR`选项来重用TIME_WAIT的端口

打开端口重用选项：
`int reuseaddr = 1;`
`setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &reuseaddr, sizeof(reuseaddr));`

## tcp_keepalive保持长连接
tcp保持长连接主要有两种方式：
第一种，使用应用层面的心跳机制，定期发送一个心跳包，检测对方是否还活着
第二种，使用tcp协议自带的keepalive机制，让tcp自己发送心跳包，这里推荐这种方式，更清爽彻底
<pre><code class="language-c line-numbers"><script type="text/plain">// 打开tcp keepalive选项
int keepalive = 1;
if(setsockopt(sockfd, SOL_SOCKET, SO_KEEPALIVE, &keepalive, sizeof(keepalive)) < 0){
    perror("\033[1;35m[WARNING]\033[0m setsockopt_keepalive");
}

// 设置空闲时长(即多久不收发数据就开始触发心跳机制，发送心跳包)
int idle = 30;
if(setsockopt(sockfd, SOL_TCP, TCP_KEEPIDLE, &idle, sizeof(idle)) < 0){
    perror("\033[1;35m[WARNING]\033[0m setsockopt_keepidle");
}

// 设置心跳包发送时间间隔(即两个心跳包之间的发送时间间隔)
int interval = 60;
if(setsockopt(sockfd, SOL_TCP, TCP_KEEPINTVL, &interval, sizeof(interval)) < 0){
    perror("\033[1;35m[WARNING]\033[0m setsockopt_keepintvl");
}

// 设置允许探测失败的最大次数(即连续3次探测失败，说明对方已断开连接)
int cnt = 3;
if(setsockopt(sockfd, SOL_TCP, TCP_KEEPCNT, &cnt, sizeof(cnt)) < 0){
    perror("\033[1;35m[WARNING]\033[0m setsockopt_keepcnt");
}
</script></code></pre>
