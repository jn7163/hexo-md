---
title: vsftpd被动模式
date: 2016-10-21 15:31:22
categories:
- linux
- 网络服务
- vsftpd
tags:
- linux
- 网络服务
- vsftpd
- 被动模式
- 主动模式
keywords: vsftpd, 被动模式, 主动模式
---
> 
默认flashfxp连接ftp服务器使用的是被动模式，什么是ftp的主动模式和被动模式？这两种模式有什么区别？以及如何开启vsftpd的被动模式？

<!-- more -->

## 主动模式和被动模式
> 
`主动模式(port)`
port方式的连接过程是：客户端向服务器的ftp端口(默认是21)发送连接请求，服务器接受连接，建立一条命令链路。
当需要传送数据时，客户端在命令链路上用port命令告诉服务器："我打开了xxxx端口，你过来连接我"。
于是服务器从20端口向客户端的xxxx端口发送连接请求，建立一条数据链路来传送数据

> 
`被动模式(pasv)`
客户端向服务器的ftp端口(默认是21)发送连接请求，服务器接受连接，建立一条命令链路。
当需要传送数据时，服务器在命令链路上用pasv命令告诉客户端："我打开了xxxx端口，你过来连接我"。
于是客户端向服务器的xxxx端口发送连接请求，建立一条数据链路来传送数据


## 开启vsftpd被动模式
<pre><code class="language-bash line-numbers">vim /etc/vsftpd/vsftpd.conf
pasv_enable=YES
pasv_min_port=12000     # pasv端口起始号
pasv_max_port=12199     # pasv端口结束号


service vsftpd restart
</code></pre>

如果服务器开启了防火墙，请放行12000-12199端口
