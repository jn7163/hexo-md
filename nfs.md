---
title: CentOS NFS配置
date: 2016-10-13 15:43:49
categories:
- linux
- 网络服务
- nfs
tags:
- linux
- 网络服务
- nfs
- linux文件共享
keywords: Linux, NFS, Linux文件共享, CentOS, 网络服务
---
> 我们知道windows主机和linux主机之间是使用samba来进行文件共享的，使用的是smb协议。那linux与linux之间呢，我们可以使用nfs进行文件共享，nfs配置起来也很简单

<!-- more -->

## nfs软件包
<pre><code class="language-bash line-numbers">yum -y install nfs-utils rpcbind    # 客户端也要安装，未安装可能会导致无法挂载nfs
</code></pre>

## 共享配置
<pre><code class="language-bash line-numbers">vim /etc/exports
--- exports ---
/share *(rw,sync,no_root_squash,no_subtree_check)

*               代表所有主机都可以访问nfs。也可以写具体的ip，网段，域名
/share          共享目录的绝对路径
rw              读写权限
sync            数据实时同步写入磁盘
no_root_squash      root用户具有根目录的完全管理访问权限
no_subtree_check    不检查父目录权限 

## nfs的常用参数：
ro      只读访问 
rw      读写访问 
sync    所有数据在请求时写入共享 
async   NFS在写入数据前可以相应请求 
secure  NFS通过1024以下的安全TCP/IP端口发送 
insecure    NFS通过1024以上的端口发送 
wdelay      如果多个用户要写入NFS目录，则归组写入（默认） 
no_wdelay   如果多个用户要写入NFS目录，则立即写入，当使用async时，无需此设置。 
hide        在NFS共享目录中不共享其子目录 
no_hide     共享NFS目录的子目录 
subtree_check       如果共享/usr/bin之类的子目录时，强制NFS检查父目录的权限（默认） 
no_subtree_check    和上面相对，不检查父目录权限 
all_squash          共享文件的UID和GID映射匿名用户anonymous，适合公用目录。 
no_all_squash       保留共享文件的UID和GID（默认） 
root_squash root    用户的所有请求映射成如anonymous用户一样的权限（默认） 
no_root_squash root 用户具有根目录的完全管理访问权限 
anonuid=xxx         指定NFS服务器/etc/passwd文件中匿名用户的UID 
anongid=xxx         指定NFS服务器/etc/passwd文件中匿名用户的GID


## nfs相关服务端口
portmap(rpcbind) (tcp/udp/111)、nfs (tcp/udp/2049)、其他动态调用小于1024未使用的端口

## 如果修改了/etc/exports文件, 不需要重新激活nfs，只要重新扫描一次/etc/exports文件，并且重新将设定加载即可：
exportfs [-aruv]
参数说明:
-a  全部挂载（或卸载）/etc/exports文件内的设定
-r  重新挂载/etc/exports中的设置，此外同步更新/etc/exports及/var/lib/nfs/xtab中的内容
-u  卸载某一目录
-v  在export时将共享的目录显示在屏幕上
</code></pre>

## 运行nfs服务
### CentOS 6.x
<pre><code class="language-bash line-numbers">chkconfig rpcbind on
chkconfig nfs on
service rpcbind start   # 先运行rpcbind！
service nfs start
</code></pre>

### CentOS 7.x
<pre><code class="language-bash line-numbers">systemctl enable rpcbind
systemctl enable nfs-server # 注意是 "nfs-server"
systemctl start rpcbind
systemctl start nfs-server
</code></pre>


## 挂载nfs
<pre><code class="language-bash line-numbers">mount -t nfs 127.0.0.1:/share /mnt
</code></pre>

## 相关
<pre><code class="language-bash line-numbers">showmount -e 127.0.0.1  # 查看服务器的共享目录

# 开机自动挂载nfs共享，加入fstab文件即可
192.168.1.100:/share /mnt nfs defaults 0 0
</code></pre>
