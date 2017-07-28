---
title: Linux ulimit详解
date: 2017-07-12 05:03:00
categories:
- linux
- 基础命令
- ulimit
tags:
- linux
- 基础命令
- ulimit
keywords: Linux ulimit 详解
---

> 
`ulimit` 用于限制 shell 启动进程所占用的资源，支持以下各种类型的限制：所创建的内核文件的大小、进程数据块的大小、Shell 进程创建文件的大小、内存锁住的大小、常驻内存集的大小、打开文件描述符的数量、分配堆栈的最大大小、CPU 时间、单个用户的最大线程数、Shell 进程所能使用的最大虚拟内存，同时，它支持硬资源和软资源的限制

<!-- more -->

## ulimit命令
<pre><code class="language-bash line-numbers"><script type="text/plain"># root @ localhost in ~ [5:10:20]
$ ulimit -a
-t: cpu time (seconds)              unlimited
-f: file size (blocks)              unlimited
-d: data seg size (kbytes)          unlimited
-s: stack size (kbytes)             8192
-c: core file size (blocks)         0
-m: resident set size (kbytes)      unlimited
-u: processes                       7854
-n: file descriptors                1024
-l: locked-in-memory size (kbytes)  64
-v: address space (kbytes)          unlimited
-x: file locks                      unlimited
-i: pending signals                 7854
-q: bytes in POSIX msg queues       819200
-e: max nice                        0
-r: max rt priority                 0
-N 15:                              unlimited
</script></code></pre>

ulimit 用于限制当前 shell 启动进程所占用的资源
常用参数及用法：
`ulimit -a`：查看当前shell的所有资源限制，默认显示软限制
`ulimit -Sa`：查看当前shell的所有资源限制，-S表示显示软限制(即警告值)
`ulimit -Ha`：查看当前shell的所有资源限制，-H表示显示硬限制
`ulimit -n`：显示当前可打开的文件描述符数量，软限制
`ulimit -Hn`：显示当前可打开的文件描述符数量，硬限制
`ulimit -n 10240`：修改当前shell可打开的文件描述符数为 10240，默认都是软限制，除非指明参数-H，下同
`ulimit -Hn 51200`：硬限制
`ulimit -s 102400`：修改堆栈大小的限制为 102400 kbytes 即 100MB
`ulimit -Hs unlimited`：不限制
`ulimit -Ss unlimited`：不限制

注意：除了root用户，普通用户如果用 ulimit 将硬限制调小了，就不能再调回来了！

## limits.conf配置文件
主配置文件：`/etc/security/limits.conf`及分段配置文件：`/etc/security/limits.d/*.conf`
该配置文件有默认值，它限制每个用户及用户组的资源限制

当登录用户时：
- 先应用`/etc/security/limits.conf`中的限制值，包括软限制和硬限制
- 然后在应用`/etc/profile`、`/etc/bashrc`、`~/.bash_profile`、`~/.bashrc`中的限制，没有则保持配置文件limits.conf中的值

而用 ulimit 修改的硬限制不能超过该用户在`/etc/security/limits.conf`中的值，只能在它限定的范围内

当创建用户时，也是先应用`/etc/security/limits.conf`中的限制值

注意，修改了`/etc/security/limits.conf`文件后，需要重新登录用户才能生效，而`ulimit`是立即生效

## 文件描述符的限制
一般其他的值在配置文件中都可以设置，但是对于文件描述符的限制，除了配置文件，还有内核的限制
内核的限制决定了当前内核可以打开的最大文件句柄数，是所有进程能够打开的文件句柄数之和的最大值

一般我们不需要去修改这个值，除非因为某些原因需要修改，比如装 oracle 数据库就必须设置

如何修改：
- `/etc/sysctl.conf`中设置：`fs.file-max = 50000000`，然后执行`sysctl -p`生效
- 也可以临时修改`echo 50000000 > /proc/sys/fs/file-max`，都需要root权限

查看当前系统的文件句柄使用情况：`cat /proc/sys/fs/file-nr`：三个值分别表示：已分配的句柄数、已分配未使用的句柄数、file-max值
