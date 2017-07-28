---
title: tcpdump 详解
date: 2016-12-08 13:58:00
categories:
- linux
- 基础命令
- tcpdump
tags:
- linux
- 基础命令
- tcpdump
keywords: tcpdump
---
> 通俗的说，tcpdump是一个抓包工具，用于抓取互联网上传输的数据包。形象的说，tcpdump就好比是国家海关，驻扎在出入境的咽喉要道，凡是要入境和出境的集装箱，海关人员总要打开箱子，看看里面都装了点啥。学术的说，tcpdump是一种嗅探器（sniffer），利用以太网的特性，通过将网卡适配器（NIC）置于混杂模式（promiscuous）来获取传输在网络中的信息包

<!-- more -->

## 常用参数
<pre><code class="language-bash line-numbers">-i  指定监视的网卡(any/lo/eth0...),默认监视第1张网卡
-nn 显示协议/端口号而不显示相关协议/端口名称
-v  显示较详细的信息
-vv 显示更详细的信息
-A  以ASCII编码显示抓取的内容
-X  以16进制和ASCII编码显示原始信息
-XX 从以太网部分开始显示抓取的内容，而非从网络层开始
-c  抓取指定个数据包
-e  显示头部信息
-l  将输出变为行缓冲
-D  显示所有可监视的网卡
-w  将数据包保存至文件
-r  从文件读取其数据包内容
</code></pre>

## 实例
<pre><code class="language-bash line-numbers">## 监视eno16777736
tcpdump -i eno16777736

## 监视所有网卡any
tcpdump -i any

## 抓包内容保存到文件
tcpdump -i eno16777736 -w eno16777736.cap

## 回放抓包文件
tcpdump -i eno16777736 -r eno16777736.cap

## 抓取tcp数据包
tcpdump -i eno16777736 -Annvl 'tcp'

## 抓取udp数据包
tcpdump -i eno16777736 -Annvl 'udp'

## 抓取icmp数据包
tcpdump -i eno16777736 -Annvl 'icmp'

## 抓取ip数据包
tcpdump -i eno16777736 -Annvl 'ip'

## 抓取arp数据包
tcpdump -i eno16777736 -Annvl 'arp'

## 抓取rarp数据包
tcpdump -i eno16777736 -Annvl 'rarp'

## 抓取dns解析
tcpdump -i eno16777736 -Annvl 'port 53'

## 抓取http数据
tcpdump -i eno16777736 -Annvl 'port 80'

## 抓取192.168.255.101主机接收/发送的数据包
tcpdump -i eno16777736 -Annvl 'host 192.168.255.101'

## 抓取发往192.168.255.101的数据包
tcpdump -i eno16777736 -Annvl 'dst host 192.168.255.101'

## 抓取发往192.168.255.101的dns解析请求
tcpdump -i eno16777736 -Annvl 'port 53 and dst host 192.168.255.101'

## 抓取从192.168.255.101发来的数据包
tcpdump -i eno16777736 -Annvl 'src host 192.168.255.101'

## 抓取192.168.255.0/24网段的数据包
tcpdump -i eno16777736 -Annvl 'net 192.168.255.0/24'

## and、or、not逻辑连接符
tcpdump -i eno16777736 -Annvl 'port 53 and host 192.168.255.101'
tcpdump -i eno16777736 -Annvl 'port 53 or 80'
tcpdump -i eno16777736 -Annvl 'not port 53'
tcpdump -i eno16777736 -Annvl '! port 53' # 同not
tcpdump -i eno16777736 -Annvl 'src host 192.168.255.103 and dst host (192.168.255.104 or 192.168.255.105)'

## and、or、not 同 &&、||、!
</code></pre>
