---
title: awk详解
date: 2016-11-03 12:54:33
categories:
- linux
- 文本工具
tags:
- linux
- 文本工具
keywords: awk, 文本工具
---
> 这篇文章介绍的是awk常用参数和用法，关于awk的函数，条件判断，for循环等暂不涉及，因为我认为不必要掌握这么多。我们可以通过shell和python完成这些，没必要一棵树上吊死，我们可以在其他树上多死几次

<!-- more -->

## awk命令
> 
awk是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。
简单来说awk就是把文件逐行的读入，以空格(多个连续空格或tab也视为一个空格处理)为默认分隔符将每行切片，切开的部分再进行各种分析处理。
awk有3个不同版本: awk、nawk和gawk，未作特别说明，一般指gawk，gawk是GNU版本。

## 命令格式
<pre><code class="language-bash line-numbers"># 命令行方式
awk '{pattern action}' file(s)

# 命令行方式(自定义定界符，默认为空格)
awk -F : '{pattern action}' file(s)

# 调用awk命令文件
awk -f awk-command-file file(s)
# shell脚本方式
#!/bin/awk
...
</code></pre>

## 模式和动作
> 
任何awk语句都是由模式和动作组成，在一个awk脚本中可能有许多语句
模式部分决定动作语句何时触发及触发事件.
动作即对数据进行的操作.动作须使用{}括起来
模式可以是任何条件语句或复合语句或正则表达式，
模式包含两个特殊字段BEGIN和END
BEGIN语句使用在任何文本浏览动作之前，之后文本浏览动作依据输入文件开始执行;END语句用来在awk完成文本浏览完成后
常用的动作就是打印print，但是还有更长的代码如if和循环looping语句及循环退出等
如果不指明动作，awk默认打印出所有浏览出的记录

## 记录和域
> 
awk将输入的文件行定义为记录，行中的每个字符串定义为域，域之间用空格、TAB键或其他符号分隔，这些符号叫做分隔符
$1表示第一个域，$2表示第二个域，以此类推，$0表示所有域

## 实例
<pre><code class="language-bash line-numbers"># 准备测试文件
[root@zfl9 ~]# cp /etc/passwd .
[root@zfl9 ~]# cat passwd 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucp:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
gopher:x:13:30:gopher:/var/gopher:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
vcsa:x:69:69:virtual console memory owner:/dev:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
openvpn:x:499:499:OpenVPN:/etc/openvpn:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
nginx:x:498:498:Nginx web server:/var/lib/nginx:/sbin/nologin
#
# 打印第1个域，间隔符为:
[root@zfl9 ~]# awk -F : '{print $1}' passwd 
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
uucp
operator
games
gopher
ftp
nobody
vcsa
sshd
openvpn
tcpdump
nginx
#
# 打印第1和第3个域,用逗号隔开,打印时显示为空格
[root@zfl9 ~]# awk -F : '{print $1,$3}' passwd 
root 0
bin 1
daemon 2
adm 3
lp 4
sync 5
shutdown 6
halt 7
mail 8
uucp 10
operator 11
games 12
gopher 13
ftp 14
nobody 99
vcsa 69
sshd 74
openvpn 499
tcpdump 72
nginx 498
#
# 正则匹配,打印以root开头的记录
[root@zfl9 ~]# awk -F : '/^root/{print $0}' passwd 
root:x:0:0:root:/root:/bin/bash
#
# BEGIN和END
[root@zfl9 ~]# awk -F : 'BEGIN{print "Begin"}/^root/{print $0}END{print "End"}' passwd 
Begin
root:x:0:0:root:/root:/bin/bash
End
#
# 格式化打印user和uid
[root@zfl9 ~]# awk -F : '{print "user: "$1"\tuid: "$3}' passwd 
user: root	uid: 0
user: bin	uid: 1
user: daemon	uid: 2
user: adm	uid: 3
user: lp	uid: 4
user: sync	uid: 5
user: shutdown	uid: 6
user: halt	uid: 7
user: mail	uid: 8
user: uucp	uid: 10
user: operator	uid: 11
user: games	uid: 12
user: gopher	uid: 13
user: ftp	uid: 14
user: nobody	uid: 99
user: vcsa	uid: 69
user: sshd	uid: 74
user: openvpn	uid: 499
user: tcpdump	uid: 72
user: nginx	uid: 498
#
# BEGIN自定义变量
[root@zfl9 ~]# awk 'BEGIN{var=123456;print var}'
123456
</code></pre>
