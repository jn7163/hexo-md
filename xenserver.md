---
title: XenServer 7.1.0 安装及基础配置
date: 2017-04-06 09:14:00
categories:
- 虚拟化
- xenserver
tags:
- 虚拟化
- xenserver
keywords: xenserver, 虚拟化, xenserver7, xencenter
---
> Citrix Xenserver，思杰基于Linux的虚拟化服务器。Citrix XenServer是一种全面而易于管理的服务器虚拟化平台，基于强大的 Xen Hypervisor 程序之上。Xen技术被广泛看作是业界最快速、最安全的虚拟化软件。XenServer 是为了高效地管理 Windows(R) 和 Linux(R)虚拟服务器而设计的，可提供经济高效的服务器整合和业务连续性

<!-- more -->

### 下载安装XenServer7
<pre><code class="language-bash line-numbers">### 下载
下载ISO: http://downloadns.citrix.com.edgesuite.net/11988/XenServer-7.1.0-install-cd.iso
下载XenCenter(windows平台下的XenServer管理端): http://downloadns.citrix.com.edgesuite.net/12009/XenServer-7.1.0-XenCenterSetup-7.1.1.l10n.exe

### 安装
和安装普通的CentOS一样，插入光盘，启动安装就行，按照安装向导一步一步来
磁盘空间至少46G,推荐100G以上，内存最少2G,推荐4G以上

### 默认分区情况
GPT分区
18GB    XenServer主机控制域(dom0)分区
18GB    升级备份分区
4GB     日志分区
1GB     交换分区
5GB     UEFI引导分区

### 存储库(SR)、物理块设备(PBD)、虚拟磁盘映像(VDI)、虚拟块设备(VBD) 之间的关系  SR可用来存储ISO镜像文件，或者VDI
                            * 'S R' * 
XenServer主机 <--> PBD <--> *  VDI  * <--> VBD <--> VM
XenServer主机 <--> PBD <--> *  VDI  * <--> VBD <--> VM
XenServer主机 <--> PBD <--> *  VDI  * <--> VBD <--> VM
                            * 'S R' *
</code></pre>

### XenServer 基本配置流程
<pre><code class="language-bash line-numbers">1. 修改lvm配置，否则无法手动创建lvm逻辑卷
--- /etc/lvm/lvm.conf ---
...
metadata_read_only = 0
...
-------------------------

2. 准备另一块硬盘或者直接就在XenServer的系统分区(推荐另增一块硬盘)，或者是Windows文件共享(CIFS)，Linux文件共享(NFS)，用来存放ISO镜像启动文件

3. XenCenter 创建SR,类型为ISO库，连接上面创建的ISO库,点击刷新就可以看到你的iso镜像文件了

4. 新建虚拟机VM -> 控制台 -> 然后就可以安装你的虚拟机了
</code></pre>

### XenServer 基本命令
<pre><code class="language-bash line-numbers">### 新建SR
# ISO库
xe sr-create name-label=boot_iso type=iso content-type=iso device-config:location=/xenserver/iso device-config:legacy_mode=true
# VDI库
xe sr-create name-label="Local storage 2" type=lvm content-type=user device-config:device=/dev/xenserver/data host-uuid=uuid_host shared=false

xe vm-list  # 查看vm
xe host-list # 查看host_uuid
xe-toolstack-restart # 重启toolstack

### 删除SR
xe sr-list
xe sr-list name-label=boot_iso  # 查看sr的uuid

xe pbd-list sr-uuid=UUID_SR     # 查看pbd的uuid

xe pdb-unplug uuid=UUID_PBD     # 断开sr与pbd的连接

xe sr-forget uuid=UUID_SR       # 删除sr

xe sr-destroy uuid=UUID_SR      # 销毁sr

xe task-list # 查看后台任务

xe task-cancel uuid=... # 结束后台任务
</code></pre>
