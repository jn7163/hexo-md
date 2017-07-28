---
title: rsync 数据同步备份
date: 2016-11-17 18:20:00
categories:
- linux
- 数据备份
tags:
- linux
- 数据备份
- rsync
keywords: rsync, 数据同步, 数据备份
---
> 在实际生产中，数据备份与同步是非常重要的事情！比如网站的一些重要文件，MySQL数据库的热备等等。今天聊聊rsync的同步艺术

<!-- more -->

## rsync
> 
rsync是linux自带的远程数据备份同步工具，使用及配置都很简单。
要使用rsync，首先我们要配置rsync服务器，并且客户端和服务器都要安装rsync软件包

## 配置rsync服务器
<pre><code class="language-bash line-numbers">## 安装rpm包
yum -y install rsync

## rsyncd.conf
vim /etc/rsyncd.conf # rsync主配置文件
uid = root
gid = root
use chroot = no
secrets file = /etc/rsyncd.secrets # 认证口令文件(自行新建)
# 其余的默认参数即可
# 默认 read only = yes 
# 表示只读 客户端不能修改服务器的文件 有需要的自行添加 read only = no
[test]
       path = /root/test
       comment = test
       auth users = root # 认证的用户

## rsyncd.secrets
vim /etc/rsyncd.secrets
root:123456

## 更改权限为600
chmod 600 /etc/rsyncd.secrets

## 启动 rsyncd 服务
systemctl enable rsyncd
systemctl start rsyncd
</code></pre>

## rsync命令
<pre><code class="language-bash line-numbers">格式：
rsync [option] src dst
参数：
-a 归档
-z 压缩
-v verbose
-P 进度等信息
--delete 删除src没有而dst上有的文件
--exclude 排除某些文件
--exclude-from 从指定文件读取排除参数
--password-file 指定密码文件(密码文件需设置权限为600！)

rsync -azvP root@192.168.255.105::test /tmp/test

# 指定密码文件
echo '123456' > /root/.rsync.passwd && chmod 600 /root/.rsync.passwd
rsync -azvP --password-file=/root/.rsync.passwd root@192.168.255.105::test /tmp/test

# 删除本地没有而服务器上有的文件
rsync -azvP --delete --password-file=/root/.rsync.passwd /tmp/test/ root@192.168.255.105::test

# 排除以bak结尾的文件
rsync -azvP --exclude=*.bak --password-file=/root/.rsync.passwd root@192.168.255.105::test /tmp/test

# 从指定文件中读取排除参数
echo '*.bak' > /root/.exclude.rsync
rsync -azvP --exclude-from=/root/.exclude.rsync --password-file=/root/.rsync.passwd root@192.168.255.105::test /tmp/test

# 从本地上传数据到服务器
把 src 和 dst 互换即可

# 查看目标的所有文件
rsync -v rsync://root@192.168.255.105::test
</code></pre>
