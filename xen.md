---
title: xen虚拟化 安装配置
date: 2017-01-25 15:55:00
categories:
- 虚拟化
- xen
tags:
- 虚拟化
- xen
keywords: xen, xen4, 虚拟化, pv
---
> Xen是一个开放源代码虚拟机监视器，由XenProject开发。它打算在单个计算机上运行多达128个有完全功能的操作系统。在旧（无虚拟硬件）的处理器上执行Xen，操作系统必须进行显式地修改（“移植”）以在Xen上运行（但是提供对用户应用的兼容性）。这使得Xen无需特殊硬件支持，就能达到高性能的虚拟化

<!-- more -->

## 环境
- CentOS 6.8 final (最好安装桌面环境)
- **CentOS 7 需要编译安装xen,kernel (在VMware里面安装的Xen一直有问题，想直接在笔记本上安装CentOS7，嫌麻烦，暂时没去弄)**

## 安装xen4

<pre><code class="language-bash line-numbers">yum -y install centos-release-xen
yum -y install xen vnc libvirt
yum -y update

reboot
</code></pre>

## 准备环境

<pre><code class="language-bash line-numbers">### pv半虚拟化 ###

chkconfig libvirtd on
service libvirtd start

yum -y install httpd

chkconfig httpd on
service httpd start

cd /opt
mkdir iso img src

cp -af /mnt/centos.iso iso/

dd if=/dev/zero of=/opt/img/centos.img bs=1M count=1 seek=10239 # 创建镜像

mkdir /var/www/html/centos

--- /etc/fstab ---
/opt/iso/centos.iso /var/www/html/centos iso9660 defaults,loop 0 0
--- /etc/fstab ---

mount -a

cp -af /var/www/html/centos/isolinux/{vmlinuz,initrd.img} /opt/src/
</code></pre>

## pv配置文件

<pre><code class="language-bash line-numbers">--- /etc/xen/centos.conf ---
name = "centos"
kernel = "/opt/src/vmlinuz"
ramdisk = "/opt/src/initrd.img"
#bootloader = "pygrub"
memory = 1024
maxmem = 1024
vcpus = 1
vif = [ 'bridge=virbr0' ]    # 可配置多个网卡，逗号隔开
vfb = [ 'vnclisten=0.0.0.0, vncdisplay=1' ]
disk = [ '/opt/img/centos.img, raw, xvda, rw' ]    # 可配置多个硬盘，逗号隔开
--- /etc/xen/centos.conf ---
</code></pre>

## 创建虚拟机

<pre><code class="language-bash line-numbers">cd /etc/xen/

xl create -V centos.conf

安装完成关机，修改配置文件

xl destroy centos

--- centos.conf ---
...
#kernel
#ramdisk
bootloader = "pygrub"
...
--- centos.conf ---

启动虚拟机

xl create -V centos.conf
</code></pre>

## xl命令

<pre><code class="language-bash line-numbers">xl create centos.conf       # 创建虚拟机
xl create -c centos.conf    # 创建虚拟机，console连接
xl create -V centos.conf    # 创建虚拟机，vncview连接

xl info     # 查看xen相关信息

xl list     # 列出当前虚拟机

xl shutdown centos  # 关闭虚拟机

xl destroy centos   # 强制关闭虚拟机

xl reboot centos    # 重启虚拟机

xl vncview centos   # vnc连接虚拟机
</code></pre>
