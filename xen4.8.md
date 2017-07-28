---
title: CentOS7 编译安装 Xen4.8
date: 2017-04-06 09:42:00
categories:
- 虚拟化
- xen
tags:
- 虚拟化
- xen
keywords: 虚拟化, Xen, Xen4.8, Xen4, CentOS7
---
> 由于CentOS7没有现成的Xen rpm安装包，所以只能通过编译内核，编译安装Xen4

<!-- more -->

### 安装软件包
<pre><code class="language-bash line-numbers">yum -y groupinstall "Development Tools"

yum -y install texinfo python-devel acpica-tools libuuid-devel ncurses-devel glib2 glib2-devel libaio-devel openssl-devel yajl-devel glibc-devel glibc-devel.i686 pixman-devel bc xz-devel

rpm -ivh  http://mirror.centos.org/centos/6/os/x86_64/Packages/dev86-0.16.17-15.1.el6.x86_64.rpm
</code></pre>

### 编译安装Xen4.8
<pre><code class="language-bash line-numbers">git clone git://xenbits.xen.org/xen.git
cd xen/
./configure
make dist
make install
</code></pre>

### 编译内核3.18.18
<pre><code class="language-bash line-numbers">wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.18.18.tar.gz
tar xf linux-3.18.18.tar.gz
cd linux-3.18.18

make menuconfig

vim .config     # 修改下列参数
--------------------------------------
    CONFIG_X86_IO_APIC=y
    CONFIG_ACPI=y
    CONFIG_ACPI_PROCFS=y (optional)
    CONFIG_XEN_DOM0=y
    CONFIG_PCI_XEN=y
    CONFIG_XEN_DEV_EVTCHN=y
    CONFIG_XENFS=y
    CONFIG_XEN_COMPAT_XENFS=y
    CONFIG_XEN_SYS_HYPERVISOR=y
    CONFIG_XEN_GNTDEV=y
    CONFIG_XEN_BACKEND=y
    CONFIG_XEN_NETDEV_BACKEND=m
    CONFIG_XEN_BLKDEV_BACKEND=m
    CONFIG_XEN_PCIDEV_BACKEND=m
    CONFIG_XEN_BALLOON=y
    CONFIG_XEN_SCRUB_PAGES=y
---------------------------------------

make
make modules
make modules_install
make install
</code></pre>

### 配置grub
<pre><code class="language-bash line-numbers">grub2-set-default 'xxx'     # 设置默认启动项

grub2-mkconfig -o /etc/grub2.cfg    # 生成grub.cfg

cat /etc/grub2.cfg
--- grub2.cfg ---
...
        multiboot /xen.gz
        module /vmlinuz-3.18.18 root=/dev/mapper/centos-root ro rd.lvm.lv=centos/root rd.lvm.lv=centos/swap crashkernel=auto rhgb quiet
        module /initramfs-3.18.18.img
...
-----------------
## 如果是上面的输出则没问题

## 重启选择 Xen4.8+Kernel3.18.18 启动
</code></pre>

### 配置Xen
<pre><code class="language-bash line-numbers">### 链接xen相关库
cd /usr/local/lib/
ls | grep 'so' | xargs -i -n1 ln -s `pwd`/{} /usr/lib/{}
ldconfig

### 挂载xenfs内核模块
modprobe xenfs
mount -t xenfs xenfs /proc/xen
/etc/fstab --> none /proc/xen xenfs defaults 0 0

### 加入开机启动项
chkconfig --add xencommons
chkconfig xencommons on
service xencommons start

xl info
xl list
</code></pre>
