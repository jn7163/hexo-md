---
title: CentOS7 进入单用户模式
date: 2016-11-28 19:23:00
categories:
- linux
- centos&rhel-7
tags:
- linux
- centos&rhel-7单用户模式
keywords: CentOS/RHEL7, 单用户模式, 修改root密码
---
> CentOS/RHEL6 开机进入单用户模式很容易，但是在 CentOS/RHEL7 中发生了变化，引导换成了grub2，相应的进入单用户模式的方式也变了

<!-- more -->

### 进入单用户模式
<pre><code class="language-bash line-numbers">在 grub2 引导界面选中启动内核， 按 E 键编辑
找到 'ro' ，改成 'rw init=/sysroot/bin/sh'
按 Ctrl+X 启动

chroot /sysroot/
passwd root

然后直接手动按物理电源键重启(虚拟机直接控制台重启)
因为reboot或init命令没用，按照网上的教程输入 touch /.autorelabel; exec /sbin/reboot 或 exec /sbin/init 3无效！
只有手动重启！！！
</code></pre>
