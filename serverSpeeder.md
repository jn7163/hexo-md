---
title: 锐速破解版(转自91yun)
date: 2016-10-23 16:08:22
categories:
- linux
- 网络服务
- 锐速
tags:
- linux
- 网络服务
- 锐速
- 更换内核
keywords: vps, 锐速破解版, 更换内核
---
> 
对于国外的vps，装了锐速和没装锐速的区别还是很大的，不吹不黑，我之前用的vultr没装锐速顶多4m网速，装了锐速之后平均7-8m，不过现在vultr又被玩坏了，丢包严重，延迟超高

<!-- more -->

## 安装锐速破解版(转自91yun)
[91yun](https://www.91yun.org) (2017-03-17更新)
**不支持OVZ虚拟技术的vps!**

<pre><code class="language-bash line-numbers">wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder.sh && bash serverspeeder.sh

### 卸载方法
chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f
</code></pre>

## 若提示内核不匹配请更换内核
### CentOS 6
<pre><code class="language-bash line-numbers"># CentOS6内核更换为：2.6.32-504.3.3.el6.x86_64
rpm -ivh http://soft.91yun.org/ISO/Linux/CentOS/kernel/kernel-firmware-2.6.32-504.3.3.el6.noarch.rpm
rpm -ivh http://soft.91yun.org/ISO/Linux/CentOS/kernel/kernel-2.6.32-504.3.3.el6.x86_64.rpm --force
</code></pre>

### CentOS 7
<pre><code class="language-bash line-numbers"># CentOS7内核更换为： 3.10.0-229.1.2.el7.x86_64
rpm -ivh http://soft.91yun.org/ISO/Linux/CentOS/kernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force
</code></pre>

**安装完后请重启**

`rpm -qa | grep kernel` 查看是否安装成功
