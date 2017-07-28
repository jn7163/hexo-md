---
title: git 简明笔记
date: 2017-04-06 12:21:00
categories:
- 版本控制
- git
tags:
- 版本控制
- git
keywords: git, 版本控制
---
> git和其他版本控制系统（如CVS）有不少的差别，git本身关心文件的整体性是否有改变，但多数的CVS或Subversion系统则在乎文件内容的差异。因此git更像一个文件系统，直接在本机上获取数据，不必连接到主机端获取数据

<!-- more -->

### git安装及配置
<pre><code class="language-bash line-numbers">yum -y install git

## 基本配置
git config --global core.editor vim
git config --global user.name 'USERNAME'
git config --global user.email 'Email_Address'
git config --global color.ui true
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.cl clone
git config --global alias.br branch
git config --global alias.di diff
git config --global alias.unstage 'reset HEAD'
git config --global alias.last 'log -1'

.gitignore  # 告诉git,忽略哪些文件(夹)

git config --list

## 配置https免密码认证
--- ~/.git-credentials ---
https://username:password@github.com
--- ~/.git-credentials ---

git config --global credential.helper store


### 附: gitconfig配置，直接复制到 ~/.gitconfig 即可
--- ~/.gitconfig ---
[push]
	default = matching
[core]
	trustctime = false
	editor = vim
	filemode = false
[color]
	ui = true
[credential]
	helper = cache --timeout=3600
[merge]
	tool = vimdiff
[mergetool]
	keeptemporaries = false
	keepbackups = false
	prompt = false
	trustexitcode = false
[alias]
	last = log -1 --stat
	cp = cherry-pick
	co = checkout
	cl = clone
	ci = commit
	st = status -sb
	br = branch
	unstage = reset HEAD --
	dc = diff --cached
	lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %Cblue<%an>%Creset' --abbrev-commit --date=relative --all
[user]
	email = example@gmail.com   # 修改为你的Email地址
	name = Bob                  # 修改为你的Name
</code></pre>

### git基本用法
<pre><code class="language-bash line-numbers"># 常用命令
git init                    # 在当前目录建立git版本库
git init test               # 在当前目录下建立名为test的git版本库
git init --bare             # 在当前目录建立git裸版本库(没有工作区,通常用于远程repo)
git init --bare test.git    # 在当前目录下建立名为test.git的裸版本库(没有工作区,通常用于远程repo)

git clone remote_addr   # 克隆remote版本库
git clone -o jQuery https://github.com/jquery/jquery.git    # 指定remote_name为jQuery,默认origin

git add files/dirs      # 添加文件(夹)至暂存区index
git add -A              # 添加所有文件(夹)至index
git commit -m 'init'    # 提交暂存区文件至版本库
g
git commit -a -m '...'  # commit所有修改了的文件, 没有加入暂存区的文件不会被commit

git status              # 查看状态

git diff
git diff file                   # 工作区 vs 暂存区，如果还没add到暂存区，则 工作区 vs 最新版本库
git diff dev
git diff dev file               # 工作区 vs dev分支
git diff --cached file          # 暂存区 vs 最新版本库
git diff --cached HEAD~2 file   # 暂存区 vs 最近的倒数第2次版本库
git diff --cached commit_id file# 暂存区 vs commit_id版本
git diff HEAD~2|commit_id file  # 工作区 vs 指定版本库
git diff commit_id commit_id    # 版本库 vs 版本库

git log                     # 查看log
git log -3                  # 查看最近3次commit的log
git log --oneline           # 每个commit放在一行
git log --pretty=oneline    # 完整显示commit_id 
git log --graph             # 查看分支图形
git log --oneline --graph
git log -p                  # 同时显示diff信息
git log dev                 # 查看dev分支log
git log origin/master       # 查看origin/master分支log
git log -p master FETCH_HEAD# 比较master分支与fetch最新分支

git reflog              # 查看操作记录,可查看所有commit_id

git ls-files    # 查看暂存区文件
git ls-files -s # 查看暂存区文件

git rm file             # 删除暂存区和工作区文件
git rm --cached file    # 删除暂存区文件
git commit -m '...'

git mv file_old file_new    # 重命名暂存区和工作区文件
git commit -m '...'

# 分支管理
git checkout -b dev # 创建dev分支并切换到dev分支
相当于
git branch dev
git checkout dev

git branch          # 查看当前分支
git branch -a|-r    # 查看所有|远程分支

git checkout master
git merge dev       # 合并dev分支到当前分支(master)
git branch -d dev   # 删除dev分支

git merge --no-ff -m '...' dev  # 保留dev分支信息(关闭fast forward模式),其实就是新建一个提交

git stash       # 保存暂存区文件至一个临时位置(常用于修复bug)
git stash list  # 查看
git stash apply name    # 恢复现场
git stash pop name      # 恢复现场(删除stash)

git branch -D dev   # 强制删除未合并的分支

git branch --set-upstream master origin/master  # 手动建立分支追踪关系,clone操作默认会建立分支追踪关系

git remote add origin https://github.com/zfl9/test.git  # 添加remote主机
git remote
git remote -v
git remote show origin
git remote rm origin                    # 删除remote主机
git remote rename origin github_test    # 重命名remote主机

# 需要同步多个远程仓库时，可以用这种方法，只要push一次，就把所有的远程仓库都同步了
git remote set-url --add origin git@git.coding.net:zfl9/vim.git  # 添加url
git remote set-url --del origin git@git.coding.net:zfl9/vim.git  # 删除url

### git fetch/pull
# git fetch是从remote拉取最新的commit,然后手动merge合并
# git pull是从remote拉取最新的commit并进行merge合并(不安全)

# fetch & merge
git fetch origin master     # 从origin拉取master分支
git fetch origin dev        # 从origin拉取dev分支
git fetch origin tag v1.0   # 从origin拉取tag

git log -p master origin/master # 查看log及diff

git merge origin/master     # 合并origin/master分支到当前分支
git merge FETCH_HEAD        # 同上
或者
git fetch origin master:temp    # 从origin拉取master分支,并在本地将其创建为temp分支
git log -p master temp
git merge temp
git branch -d temp

git branch -dr origin/master    # 删除fetch的远程分支origin/master

# pull
usage: git pull remote_name remote_branch:local_branch
git pull origin test:dev        # 拉取origin的test分支,并与本地dev分支合并
git pull origin master          # 拉取origin的master分支,并与当前分支合并
git pull origin                 # 如果当前分支存在追踪关系,则可以省略分支名
git pull                        # 如果当前分支只有一条追踪关系,则还可以省略remote主机名

# push
usage: git push remote_name local_branch:remote_branch
## branch
git push origin test:dev    # 推送test分支到remote的dev分支
git push origin master      # 推送master分支至与之存在追踪关系的remote分支
git push -u aliyun master   # 指定master分支默认remote主机为aliyun(当存在多条追踪关系时)
git push origin             # 如果当前分支存在追踪关系,则可以省略分支名
git push                    # 如果当前分支只有一条追踪关系,则还可以省略remote主机名
git push origin :master         # 删除远程分支
git push --delete origin master # 同上
git push --all origin       # 推送本地所有分支到origin主机

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
git v2.0 之前默认是推送全部分支到远程分支，"matching"方式
git v2.0 之后默认是推送当前分支到远程分支，"simple"方式

# 修改默认push方式
git config --global push.default matching
or
git config --global push.default simple
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

## tag
git push origin tag_name    # 推送tag
git push origin --tags      # 推送所有tag

git tag v1.0        # 打在最新的commit上
git tab v2.0 3f89   # 打在指定的commit上
git tag -a tag_name -m '...' # 含备注的tag

git tag             # 查看tag
git tag -l
git show tag_name

git tag -d v1.0                     # 删除本地tag
git push --delete origin tag v1.0   # 删除远程tag

git checkout -b dev v1.0    # 检出tag v1.0到新分支dev
git checkout v1.0           # 检出tag v1.0到工作区,但是不能修改,若要修改请用上面的方式
</code></pre>

### git恢复操作详解
在csdn发现的这个图，感觉非常清晰，就拿过来了
![工作区 暂存区 版本库](/images/git.png)

<pre><code class="language-bash line-numbers"># 暂存区 -> 工作区
# 恢复工作区的文件至暂存区状态(如果工作区文件没有add到暂存区,则恢复最新版本库的文件到工作区)
git checkout -- .       # 全部file
git checkout -- file    # 指定file

# 版本库 -> 暂存区
# 恢复暂存区的文件至版本库状态
git reset --soft            # 默认参数就是--soft,可以省略
git reset                   # 全部file
git reset file              # 指定file
git reset HEAD~2            # 指定commit
git reset Commit_ID         # 指定commit

# 版本库 -> 暂存区&工作区
# 恢复工作区和暂存区的文件至版本库状态
git checkout HEAD~2
git checkout HEAD~2 file
git checkout Commit_ID
git checkout Commit_ID file

git reset --hard
git reset --hard HEAD~2
git reset --hard HEAD~2 file
git reset --hard Commit_ID
git reset --hard Commit_ID file
</code></pre>

### git服务器搭建(ssh)
<pre><code class="language-bash line-numbers">### 安装git,添加ssh公钥
yum -y install git
useradd git -s /usr/bin/git-shell

mkdir -p /home/git/.ssh/
vim /home/git/.ssh/authorized_keys    # 添加所有成员的ssh公钥
chmod 600 /home/git/.ssh/* && chown -R git:git /home/git/.ssh/

### 创建bare_repo裸仓库(bare_repo没有工作区，用于repo共享，方便团队协作)
mkdir -p /repos/
cd /repos/
git init --bare python3.git
chown -R git:git /repos/

# 接下来就是clone了
git clone git@127.0.0.1:/repos/python3.git
</code></pre>

> 
推荐几个不错的git代码托管平台
[github](https://github.com)        不用说，都知道
[coding](https://coding.net)        国内的，很不错的git托管平台，速度比github快多了
[oschina](https://git.oschina.net)  开源中国的git代码托管，不过界面没有coding好
