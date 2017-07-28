---
title: lvm raid 日常配置
date: 2017-01-03 12:04:00
categories:
- linux
- 磁盘管理
tags:
- linux
- 磁盘管理
keywords: lvm, raid, mdadm
---
> LVM 是一种可用在Linux内核的逻辑分卷管理器；RAID是一种把多块独立的硬盘（物理硬盘）按不同的方式组合起来形成一个硬盘组（逻辑硬盘），从而提供比单个硬盘更高的存储性能和提供数据备份技术

<!-- more -->

## lvm 逻辑卷管理

### 安装lvm

<pre><code class="language-bash line-numbers">yum -y install lvm*
</code></pre>

### pv

<pre><code class="language-bash line-numbers">pvcreate /dev/sd[b,c]
pvdisplay
</code></pre>

### vg

<pre><code class="language-bash line-numbers">vgcreate vg0 /dev/sd[b,c]
vgdisplay

vgchange -aay --sysinit     # 激活所有vg卷组
vgchange -ay vg_name        # 激活指定vg卷组
</code></pre>

### lv

<pre><code class="language-bash line-numbers">lvcreate -L 30G -n data vg0
lvdisplay
</code></pre>

### 在线扩展

<pre><code class="language-bash line-numbers">## 新增一块磁盘 /dev/sdf 20G

pvcreate /dev/sdf
vgextend vg0 /dev/sdf
lvextend -L +20476M /dev/vg0/data

## ext4 ##
resize2fs /dev/vg0/data

## xfs ##
xfs_growfs /dev/vg0/data

df -hT # 查看是否已经扩展成功
</code></pre>

### 缩小(离线)

<pre><code class="language-bash line-numbers">umount /dev/vg0/data

e2fsck -f /dev/vg0/data
resize2fs /dev/vg0/data 20G
lvreduce -L 20G /dev/vg0/data

mount /dev/vg0/data /mnt

df -hT # 查看是否已经缩小成功；注：xfs暂不支持缩小文件系统！
</code></pre>

## raid 磁盘阵列

<pre><code class="language-bash line-numbers"># raid0:    带区卷
# raid1:    镜像卷
# raid5:    raid5(常用)

## 创建
mdadm -C /dev/md0 -l0 -n2 /dev/sd[b,c]
mdadm -C /dev/md1 -l1 -n2 /dev/sd[b,c]
mdadm -C /dev/md5 -l5 -n3 /dev/sd[b,c,d] -x1 /dev/sde

## 状态
mdadm -D /dev/md0

## 停止
mdadm -S /dev/md0
mdadm --zero-superblock /dev/sd[b,c,d,e]

## 替换问题设备 raid5
mdadm /dev/md5 -f /dev/sdc  # 标记问题设备
mdadm /dev/md5 -r /dev/sdc  # 移除问题设备
mdadm /dev/md5 -a /dev/sdf  # 添加新设备
</code></pre>