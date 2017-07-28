---
title: svn 简明笔记
date: 2017-04-13 15:36:00
categories:
- 版本控制
- svn
tags:
- 版本控制
- svn
keywords: svn, 版本控制, subversion
---
> 其实不是很想用svn，git完全秒之好么，不过公司还是用的svn，只能默默的学习一下svn了

<!-- more -->

### 安装svn
`yum -y install subversion`

### 创建svn中央版本库
<pre><code class="language-bash line-numbers"># 指定svn调用的编辑器为vim
# 设置环境变量
SVN_EDITOR=vim

# 提交的文件存放路径
$svn/repo/db/revs/版本号

# 初始化版本库
mkdir -p /opt/svn/
svnadmin create /opt/svn/python3

# 配置版本库
cd /opt/svn/python3/conf/

--- svnserve.conf ---
[general]
anon-access = none
auth-access = write
password-db = passwd
authz-db = authz
realm = python3

###
anon-access: 控制非鉴权用户访问版本库的权限，取值范围为"write"、"read"和"none"。 即"write"为可读可写，"read"为只读，"none"表示无访问权限。 缺省值：read

auth-access: 控制鉴权用户访问版本库的权限。取值范围为"write"、"read"和"none"。 即"write"为可读可写，"read"为只读，"none"表示无访问权限。 缺省值：write

authz-db: 指定权限配置文件名，通过该文件可以实现以路径为基础的访问控制。 除非指定绝对路径，否则文件位置为相对conf目录的相对路径。 缺省值：authz

realm: 指定版本库的认证域，即在登录时提示的认证域名称。若两个版本库的 认证域相同，建议使用相同的用户名口令数据文件。 缺省值：一个UUID(Universal Unique IDentifier，全局唯一标示)。
--- svnserve.conf ---

# 密码文件
--- passwd ---
[users]
admin = 123456
test = 123456
--- passwd ---

# 权限配置
--- authz ---
[groups]
all = admin, test

[python3:/]
* =
@all = r
admin = rw
--- authz ---

# 启动svn
svnserve -d -r /opt/svn
</code></pre>

### apache+svn方式
<pre><code class="language-bash line-numbers"># 安装apache和dav_svn模块
yum -y install httpd httpd-tools mod_dav_svn subversion

# 配置apache
vim /etc/httpd/conf.d/svn.conf
&lt;Location /py3&gt;
    DAV svn
    SVNPath /opt/svn/py3
    AuthzSVNAccessFile /opt/svn/py3/conf/authz
    AuthType Basic
    AuthName &quot;Authorization SVN&quot;
    AuthUserFile /opt/svn/py3/conf/passwd
    Require valid-user
    Satisfy all
&lt;/Location&gt;

# 创建passwd文件
htpasswd -cb /opt/svn/py3/conf/passwd admin 123456
htpasswd -b /opt/svn/py3/conf/passwd test 123456

# 启动apache
systemctl start httpd

# 检出svn库，和原来的方式没什么不同，只不过用http协议传输了，并且passwd是经过md5加密的，安全一些
svn checkout http://192.168.255.101/py3/ --username admin --password 123456

浏览器访问 http://192.168.255.101/py3/ 输入用户名密码即可访问
</code></pre>

### svn基本命令
<pre><code class="language-bash line-numbers">cd ~
# checkout 检出版本库到工作区
svn checkout svn://127.0.0.1/python3 --username admin --password 123456
svn checkout svn://127.0.0.1/python3 --username admin --password 123456 my_works    # 指定本地路径
yes -> 缓存密码

# 提交commit
cd python3

--- readme.md ---
hello, subversion!
--- readme.md ---

svn status  # 查看待变更列表
svn status -u   # 显示更新信息
svn status -v   # 显示详细信息
svn add readme.md  # 添加文件(夹)到暂存区
svn commit -m 'add file readme.md'  # 提交暂存区至版本库

svn update  # 同步最新版本库至工作区
svn update -r 15    # 同步指定版本

# 版本回退
svn revert fire     # 回退工作区文件至暂存区版本
svn revert -R dir   # 目录
svn merge -r 22:20 file     # 回退文件至指定版本库版本(22->20)

svn log         # 查看版本信息
svn log -r 1:4  # 查看指定版本之间的信息
svn log -l 3 -v # -v 详细信息，-l N 指定输出条目

svn diff                    # 对比工作区文件和暂存区文件
svn diff -r 3 readme.md     # 对比工作区文件和版本库文件
svn diff -r 2:3 readme.md   # 对比版本库文件

svn cat -r 3 readme.md      # 查看指定版本文件

svn list svn://127.0.0.1/python3    # 查看版本库文件

svn export svn://127.0.0.1/python3 --username admin --password 123456 # 导出svn库(不包含.svn目录)
svn export svn://127.0.0.1/python3 --username admin --password 123456 exp   # 导出至指定目录

svn import ~/works/dest/ -m '...' svn://127.0.0.1/python3 --username admin  # 导入文件夹内容至版本库
svn import ~/.oh-my-zsh/ -m '...' svn://127.0.0.1/python3/oh-my-zsh --username admin    # 指定版本库中的文件夹名

# 分支branch
ls
branches tags master

svn copy master branches/dev

svn commit -m 'add new branch'

cd branches/dev/

do something...

svn add ..
svn status
svn commit -m 'add ...'

## 合并
cd master
svn update
svn merge ../branches/dev/
svn commit -m '...'

# 标签tag
svn copy master tags/v1.0_stable
svn commit -m '...'
</code></pre>

### svn文件冲突
<pre><code class="language-bash line-numbers"># 文件冲突原因:
当两个或者两个以上开发人员同时修改一个文件的同一个地方，就会出现冲突

# eg:
A和B两位工程师，版本库最新版本为10，A和B checkout，版本都是10，没问题

A 修改文件test的第一行为 A，然后提交到版本库，没问题，此时版本号已经是11了

B 修改文件test的第一行为 B, 然后提交到版本库，svn提示需要更新本地工作区副本至最新版本11
然后B执行 svn update , 又报错，提示文件冲突，需要手动解决，选择p

这时候工作区出现了4个文件
test    test.mine   test.r10    test.r11

test        冲突后的文件，主要就是手动修改此文件就行
test.mine   B修改后的文件test
test.r10    B提交前的版本，也就是没有导致冲突的版本
test.r11    A提交后的版本，就是A修改的文件test, 导致冲突

## 分两种情况
1. A和B讨论后，决定保留A的修改，放弃B的修改:
svn resolve --accept theirs-full test
svn ci -m '保留A的更改'

1. A和B讨论后，决定保留B的修改，放弃A的修改:
svn resolve --accept working test
svn ci -m '保留B的修改'

# svn resolve 用法
svn resolve --accept base FILE/DIR              # 保留test.r10
svn resolve --accept working FILE/DIR           # 保留test
svn resolve --accept mine-full FILE/DIR         # 保留test.mine
svn resolve --accept theirs-full FILE/DIR       # 保留test.r11
svn resolve --accept mine-conflict FILE/DIR     # 冲突部分以test.mine的为准
svn resolve --accept theirs-conflict FILE/DIR   # 冲突部分以test.r11的为准
</code></pre>

### svn树冲突
<pre><code class="language-bash line-numbers"># 树冲突:
当一名开发人员移动、重命名、删除一个文件或文件夹，而另一名开发人员也对它们进行了移动、重命名、删除或者仅仅是修改时就会发生树冲突。

# 解决: http://www.jianshu.com/p/e3cc83ca512d
</code></pre>
