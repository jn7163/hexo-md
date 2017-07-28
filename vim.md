---
title: vim解决粘贴缩进混乱,tab替换为4个空格
date: 2016-10-16 15:41:33
categories:
- linux
- 基础命令
- vim
tags:
- linux
- 基础命令
- vim
keywords: vim注释, vim, 自动注释
---
> 
我们有时候从网上复制一个比较长的代码，代码中夹带着注释符号时，vim就会自作聪明的在新行添加注释符号，代码量少的时候还好，当代码量有几十行或几百行时，就很麻烦了；顺便把tab更换为4个空格！

<!-- more -->

### 解决粘贴代码缩进混乱
<pre><code class="language-vim line-numbers"><script type="text/plain">vim /etc/vimrc
set pastetoggle=<F12>

# Paste模式，在该模式下，可将文本原本的粘贴到Vim中，以避免一些格式错误
# 设置F12快捷键切换Paste模式和正常缩进模式
# 也可以进入命令行模式 set paste 切换到粘贴模式
# set nopaste 切换到正常模式
</script></code></pre>

### 替换tab为4个空格
<pre><code class="language-vim line-numbers"><script type="text/plain">vim /etc/vimrc
set shiftwidth=4
set tabstop=4
set softtabstop=4
set expandtab
set autoindent
</script></code></pre>

### vim进阶配置
[CentOS + Windows Vim 配置](https://www.zfl9.com/vim+.html)
![vim配置附图](/images/gvim.png)
