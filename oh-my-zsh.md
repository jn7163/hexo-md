---
title: oh-my-zsh打造超级终端
date: 2017-04-13 11:09:00
categories:
- linux
- 基础命令
- oh-my-zsh
tags:
- linux
- 基础命令
- oh-my-zsh
- 超级终端
keywords: zsh, oh-my-zsh, 超级终端
---
> Oh My Zsh是一款社区驱动的命令行工具，正如它的主页上说的，Oh My Zsh 是一种生活方式。它基于zsh命令行，提供了主题配置，插件机制，已经内置的便捷操作。给我们一种全新的方式使用命令行

<!-- more -->

### 安装zsh
<pre><code class="language-bash line-numbers">yum -y install zsh

# 切换默认shell为zsh
chsh -s /bin/zsh
</code></pre>

### 安装oh-my-zsh
<pre><code class="language-bash line-numbers"># curl方式
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# wget方式
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

# ~/.oh-my-zsh 目录
lib         提供了核心功能的脚本库
tools       提供安装、升级等功能的快捷工具
plugins     自带插件的存在放位置
templates   自带模板的存在放位置
themes      自带主题文件的存在放位置
custom      个性化配置目录，自安装的插件和主题可放这里
</code></pre>

### 配置oh-my-zsh
<pre><code class="language-bash line-numbers"># 配置oh-my-zsh自动更新
--- ~/.zshrc ---
DISABLE_UPDATE_PROMPT=true

# 手动更新oh-my-zsh
upgrade_oh_my_zsh

# 命令历史记录添加时间yyyy-mm-dd
--- ~/.zshrc ---
HIST_STAMPS="yyyy-mm-dd"

# EDITOR=vim
--- ~/.zshrc ---
export EDITOR=vim

# 修复zsh/oh-my-zsh home/end/小键盘不能正常使用问题(能正常使用的请忽略)
--- ~/.zshrc ---
# key bindings
bindkey "\e[1~" beginning-of-line
bindkey "\e[4~" end-of-line
bindkey "\e[5~" beginning-of-history
bindkey "\e[6~" end-of-history

# for rxvt
bindkey "\e[8~" end-of-line
bindkey "\e[7~" beginning-of-line

# for non RH/Debian xterm, can't hurt for RH/DEbian xterm
bindkey "\eOH" beginning-of-line
bindkey "\eOF" end-of-line

# for freebsd console
bindkey "\e[H" beginning-of-line
bindkey "\e[F" end-of-line

# completion in the middle of a line
bindkey '^i' expand-or-complete-prefix

# Fix numeric keypad  
# 0 . Enter  
bindkey -s "^[Op" "0"
bindkey -s "^[On" "."
bindkey -s "^[OM" "^M"
# 1 2 3  
bindkey -s "^[Oq" "1"
bindkey -s "^[Or" "2"
bindkey -s "^[Os" "3"
# 4 5 6  
bindkey -s "^[Ot" "4"
bindkey -s "^[Ou" "5"
bindkey -s "^[Ov" "6"
# 7 8 9  
bindkey -s "^[Ow" "7"
bindkey -s "^[Ox" "8"
bindkey -s "^[Oy" "9"
# + - * /  
bindkey -s "^[Ol" "+"
bindkey -s "^[Om" "-"
bindkey -s "^[Oj" "*"
bindkey -s "^[Oo" "/"
</code></pre>

### oh-my-zsh主题
<pre><code class="language-bash line-numbers">--- ~/.zshrc ---
ZSH_THEME="ys"
</code></pre>

![oh-my-zsh ys主题](/images/oh-my-zsh.png)

### oh-my-zsh插件
<pre><code class="language-bash line-numbers"># 安装zsh语法高亮插件
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/plugins/zsh-syntax-highlighting

# 启用插件
--- ~/.zshrc ---
plugins=(git z wd extract colored-man-pages zsh-syntax-highlighting)
# 添加插件名，然后 source ~/.zshrc
# git       默认启用，提供了很多git命令别名
# z         利用最近访问的目录，模糊匹配，很方便
# wd        和z插件差不多，不过是指定一个tag给目录，相当于书签
# extract   解压用的，直接输入x 文件名，就能解压
# colored-man-pages 高亮显示man帮助页
# zsh-syntax-highlighting   语法高亮，语法检测
</code></pre>
