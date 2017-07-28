---
title: kvm虚拟化 安装配置
date: 2017-01-01 16:06:00
categories:
- 虚拟化
- kvm
tags:
- 虚拟化
- kvm
keywords: kvm, 虚拟化
---
> centos 安装及配置kvm虚拟化，包括：安装配置kvm环境；设置网桥；虚拟机的日常管理；虚拟机快照；虚拟机克隆；虚拟机迁移

<!-- more -->

### 检查宿主机是否支持cpu虚拟化

<pre><code class="language-bash line-numbers">egrep 'vmx|svm' /proc/cpuinfo
</code></pre>

### 安装kvm

<pre><code class="language-bash line-numbers">yum -y install kvm virt-viewer virt-manager qemu-kvm libvirt python-virtinst bridge-utils
</code></pre>

### 查看kvm模块是否加载

<pre><code class="language-bash line-numbers">lsmod | grep kvm
</code></pre>

### 启动libvirtd服务

<pre><code class="language-bash line-numbers">## centos6
chkconfig libvirtd on
service libvirtd start

## centos7
systemctl enable libvirtd
systemctl start libvirtd
</code></pre>

### 配置网桥

> kvm有两种网络连接方式：nat、bridge；默认是nat方式；两种连接方式的区别类似VMware；

<pre><code class="language-bash line-numbers">cd /etc/sysconfig/network-scripts/

--- ifcfg-ens38 ---
HWADDR=00:0C:29:0C:7B:71
TYPE=Ethernet
BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens38
UUID=675a7816-5057-3f6e-87c3-7117cfd12ae8
ONBOOT=yes
AUTOCONNECT_PRIORITY=-999
DEVICE=ens38
BRIDGE=br0  # 添加此行
--- ifcfg-ens38 ---

--- ifcfg-br0 ---
#HWADDR=00:0C:29:0C:7B:71
TYPE=Bridge # 修改为Bridge
BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=br0    # 修改为br0
#UUID=675a7816-5057-3f6e-87c3-7117cfd12ae8
ONBOOT=yes
AUTOCONNECT_PRIORITY=-999
DEVICE=br0  # 修改为br0
STP=no      # stp；生成树协议；
--- ifcfg-br0 ---

systemctl restart NetworkManager
systemctl restart network   ## 可能会报错，多重启几次就好了

ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:0c:7b:67 brd ff:ff:ff:ff:ff:ff
    inet 192.168.255.105/24 brd 192.168.255.255 scope global dynamic ens33
       valid_lft 1749sec preferred_lft 1749sec
    inet6 fd15:4ba5:5a2b:1008:add5:e02a:98c9:6dd3/64 scope global noprefixroute dynamic 
       valid_lft 86375sec preferred_lft 14375sec
    inet6 fe80::9321:a09f:3dd2:d7ed/64 scope link 
       valid_lft forever preferred_lft forever
3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br0 state UP qlen 1000
    link/ether 00:0c:29:0c:7b:71 brd ff:ff:ff:ff:ff:ff
4: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether 00:0c:29:0c:7b:71 brd ff:ff:ff:ff:ff:ff
    inet 192.168.255.110/24 brd 192.168.255.255 scope global dynamic br0
       valid_lft 1749sec preferred_lft 1749sec
    inet6 fd15:4ba5:5a2b:1008:d24b:e09f:7b1a:7ccd/64 scope global noprefixroute dynamic 
       valid_lft 86375sec preferred_lft 14375sec
    inet6 fe80::8eb2:707c:38a7:b2f9/64 scope link 
       valid_lft forever preferred_lft forever
5: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:9e:d7:0d brd ff:ff:ff:ff:ff:ff
    inet 192.168.254.254/24 brd 192.168.254.255 scope global virbr0
       valid_lft forever preferred_lft forever
6: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:9e:d7:0d brd ff:ff:ff:ff:ff:ff
</code></pre>

### 创建虚拟机

<pre><code class="language-bash line-numbers">## 桥接模式

virt-install \
-n centos7 \
-r 1024 \
--vcpus=1 \
--boot cdrom,hd,network,menu=on \
--disk path=/opt/img/centos7.img,size=20,format=qcow2,bus=virtio \
-c /opt/iso/centos7.iso \
--network bridge=virbr0,model=virtio \
--vnc --vnclisten=0.0.0.0 --vncport=10001 \
--os-type linux \
--os-variant=rhel7 \
--accelerate \
--noautoconsole


## NAT模式(默认)
virt-install \
-n centos7 \
-r 1024 \
--vcpus=1 \
--boot cdrom,hd,network,menu=on \
--disk path=/opt/img/centos7.img,size=20,format=qcow2,bus=virtio \
-c /opt/iso/centos7.iso \
--network network=default,model=virtio \
--vnc --vnclisten=0.0.0.0 --vncport=10001 \
--os-type linux \
--os-variant=rhel7 \
--accelerate \
--noautoconsole


### 参数解释

-n centos7  # 虚拟机名称
-r 1024     # 虚拟机内存大小
--vcpus=1   # 虚拟机cpu数量
--boot cdrom,hd,network,menu=on # 启动顺序
--disk path=/opt/img/centos7.img,size=20,format=qcow2,bus=virtio    # 硬盘
-c /opt/iso/centos7.iso # 系统安装镜像
--network network=default,model=virtio  # 网络模式
--vnc --vnclisten=0.0.0.0 --vncport=10001   # vnc
--os-type linux     # 系统类型
--os-variant=rhel7  # 系统名
--accelerate        # kvm加速器
--noautoconsole     # 取消自动控制
</code></pre>

### 修复kvm虚拟机鼠标不同步问题

<pre><code class="language-bash line-numbers">--- /etc/libvirt/qemu/VM_Name.xml ---

&lt;devices&gt;
...
&lt;input type=&#x27;tablet&#x27; bus=&#x27;usb&#x27;/&gt;
...
&lt;/devices&gt;

--- /etc/libvirt/qemu/VM_Name.xml ---

virsh shutdown VM_Name
virsh define /etc/libvirt/qemu/VM_Name.xml
virsh start VM_Name


## 目前个人发现 centos6 有这个问题，centos7 无问题；
</code></pre>

### virsh 参数

<pre><code class="language-bash line-numbers">virsh -c qemu+ssh://root@192.168.255.101/system   # 连接libvirtd

virsh list          # 查看虚拟机运行状态
virsh list --all    # 查看所有虚拟机状态

virsh start test    # 启动 虚拟机test
virsh shutdown test # 关闭 虚拟机test
virsh reboot test   # 重启 虚拟机test
virsh destroy test  # 强制关闭 虚拟机test

vitsh define /etc/libvirt/qemu/test.xml # 加载配置文件
virsh undefine test                     # 删除配置文件(虚拟机)
virsh dumpxml test                      # 导出配置
virsh edit test                         # 编辑 配置文件
virsh console test  # 连接 虚拟机test

virsh net-list      # 查看kvm网络状态
virsh net-define /etc/libvirt/qemu/networks/mynet.xml    # 加载网络配置
virsh net-undefine mynet    # 删除网络配置
virsh net-start mynet       # 启用网络mynet
virsh net-destroy mynet     # 停用网络mynet
virsh net-dumpxml mynet     # 导出网络配置
virsh net-edit mynet        # 编辑网络配置

virsh domiflist test    # 查看 虚拟机test 网络状态
</code></pre>

### qemu-img

<pre><code class="language-bash line-numbers">qemu-img info /opt/img/centos.img   # 查看img信息
qemu-img create -f qcow2 /opt/img/new.img 20G   # 创建img
qemu-img convert -f raw -O qcow2 test1.img test1.qcow2 ## 转换磁盘格式;raw不支持快照等特性
</code></pre>

### 添加硬盘

<pre><code class="language-bash line-numbers">virsh shutdown test

qemu-img create -f qcow2 /opt/img/new.img 20G

virsh edit test

--- test.xml ---
    &lt;disk type=&#x27;file&#x27; device=&#x27;disk&#x27;&gt;
      &lt;driver name=&#x27;qemu&#x27; type=&#x27;qcow2&#x27;/&gt;
      &lt;source file=&#x27;/opt/img/centos.img&#x27;/&gt;
      &lt;target dev=&#x27;vda&#x27; bus=&#x27;virtio&#x27;/&gt;
      &lt;address type=&#x27;pci&#x27; domain=&#x27;0x0000&#x27; bus=&#x27;0x00&#x27; slot=&#x27;0x06&#x27; function=&#x27;0x0&#x27;/&gt;
    &lt;/disk&gt;
    &lt;disk type=&#x27;file&#x27; device=&#x27;disk&#x27;&gt;
      &lt;driver name=&#x27;qemu&#x27; type=&#x27;qcow2&#x27;/&gt;
      &lt;source file=&#x27;/opt/img/new.img&#x27;/&gt;
      &lt;target dev=&#x27;vdb&#x27; bus=&#x27;virtio&#x27;/&gt;
    &lt;/disk&gt;
--- test.xml ---

virsh define /etc/libvirt/qemu/test.xml

virsh start test
</code></pre>

### 添加网卡

<pre><code class="language-bash line-numbers">virsh shutdown test

virsh edit test

--- test.xml ---
    &lt;interface type=&#x27;bridge&#x27;&gt;
      &lt;mac address=&#x27;52:54:00:1c:b3:36&#x27;/&gt;
      &lt;source bridge=&#x27;br0&#x27;/&gt;
      &lt;model type=&#x27;virtio&#x27;/&gt;
      &lt;address type=&#x27;pci&#x27; domain=&#x27;0x0000&#x27; bus=&#x27;0x00&#x27; slot=&#x27;0x03&#x27; function=&#x27;0x0&#x27;/&gt;
    &lt;/interface&gt;
    
    &lt;interface type=&#x27;bridge&#x27;&gt;   # bridge
      &lt;source bridge=&#x27;br0&#x27;/&gt;
      &lt;model type=&#x27;virtio&#x27;/&gt;
    &lt;/interface&gt;
    
    &lt;interface type=&#x27;network&#x27;&gt;  # nat
      &lt;source network=&#x27;default&#x27;/&gt;
      &lt;model type=&#x27;virtio&#x27;/&gt;
    &lt;/interface&gt;
--- test.xml ---

virsh define /etc/libvirt/qemu/test.xml

virsh start test
</code></pre>

### 克隆

<pre><code class="language-bash line-numbers">## virt-clone 方式
virt-clone -o test -n test-clone -f /opt/img/test-clone.img

## 直接复制配置文件及镜像文件
cp -af test.xml test-clone.xml
cp -af test.img test-clone.img

--- test-clone.xml ---
删除uuid、网卡mac地址、修改vnc端口、修改disk源文件等
--- test-clone.xml ---

virsh define test-clone.xml
virsh start test-clone
</code></pre>

### 快照

<pre><code class="language-bash line-numbers">快照配置文件  /var/lib/libvirt/qemu/snapshot/虚拟机名/快照名.xml

virsh snapshot-list test
virsh snapshot-current test # 最近的快照
virsh snapshot-dumpxml test snap1
virsh snapshot-info test snap1
virsh snapshot-create test
virsh snapshot-create-as test snap1 # 自定义快照名
virsh snapshot-delete test snap1
virsh snapshot-edit test snap1
virsh snapshot-revert test snap1    # 恢复快照，须在关机状态下！
</code></pre>

### 迁移

<pre><code class="language-bash line-numbers">## nfs共享存储 - 动态迁移

node1:  192.168.255.101 (src)
node2:  192.168.255.102 (dst)
nfs_share:  192.168.255.103 # 共享/opt/img目录

node1:  mount -t nfs 192.168.255.103:/opt/img /opt/img
node2:  mount -t nfs 192.168.255.103:/opt/img /opt/img

node1:  virsh migrate --live --unsafe --verbose test qemu+ssh://root@192.168.255.102/system
node2:  virsh dumpxml test > /etc/libvirt/qemu/test.xml
        virsh define /etc/libvirt/qemu/test.xml
</code></pre>
