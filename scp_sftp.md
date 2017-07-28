---
title: scp/sftp 命令
date: 2016-11-28 17:44:00
categories:
- linux
- 基础命令
- ssh
tags:
- linux
- 基础命令
- ssh
- scp
- sftp
keywords: scp, sftp
---
> 一般Win主机与Linux主机进行文件共享都是通过Samba或者vsftpd，Linux主机之间一般是NFS或者vsftpd。不过有时候我们只是临时复制一些文件而已，每次都要动用samba或ftp，nfs不免有些麻烦，下面介绍两个好用的命令

<!-- more -->

## scp 命令

> scp是 secure copy的缩写, scp是linux系统下基于ssh登陆进行安全的远程文件拷贝命令。
linux的scp命令可以在linux服务器之间复制文件和目录。

<pre><code class="language-bash line-numbers">## 复制文件
scp file root@192.168.255.101:/root/file # 后面接远程路径(如果路径开头不是/，则表示相对于$HOME的相对路径)
上面这条命令可以简写:
scp file root@192.168.255.101:file
scp file root@192.168.255.101:

## 复制目录
scp -r dir root@192.168.255.101:/root/dir
同样可以简写(同上)

### 对调 本地路径 和 远程路径 就是从远程下载文件(夹)

## 常用参数
-C  启用压缩
-p  保留文件的属性
-P  指定远程主机端口号
-r  递归
-q  不显示进度信息
</code></pre>

## sftp 命令

> sftp 与 scp 类似，也是通过ssh进行文件操作
不过 sftp 的功能比 scp 强大
不仅可以复制文件及目录，还可以与平时的FTP一样进行类似的操作

<pre><code class="language-bash line-numbers">sftp root@192.168.255.101
## 列出当前文件夹内容
ls

## 切换目录
cd /etc

## 打印当前目录
pwd

更多命令请使用 ? 查看！
</code></pre>

***前面一节我们讲了可以用 ssh别名 登录远程主机，在 scp&sftp 中通用！***
