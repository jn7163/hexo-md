---
title: vim 常用插件及进阶配置
date: 2017-04-09 14:35:00
categories:
- linux
- 基础命令
- vim
tags:
- linux
- 基础命令
- vim
- vim进阶
keywords: vim, vim进阶, vim插件, vim配置
---
> 在这个蔚蓝色的星球上，流传着两大神器的传说：据说Emacs是神的编辑器，而Vim是编辑器之神。一些人勇敢地拾起了Vim或Emacs，却发现学习曲线陡峭而漫长，还是有一些人留下来了，坚定地守护着这两大神器。一些说葡萄太酸的人想离开又不甘心，总是问：它们到底神在哪里啊?

<!-- more -->

### vim附图
![Win10 + gVim8](/images/gvim.png)

### CentOS - Vim
<pre><code class="language-vim line-numbers">### 环境: CentOS 7.3 1611 x86_64位  Vim 7.4
### molokai主题
# github: https://github.com/tomasr/molokai
# 把主题文件放到 /usr/share/vim/vim74/colors/molokai.vim
# /etc/vimrc
&quot; 配色主题 molokai
colo molokai

### vim插件管理器 - vundle
# 安装vundle
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
vim /etc/vimrc
&quot; vundle 插件管理器
set nocompatible
filetype off
set rtp+=~/.vim/bundle/Vundle.vim/
call vundle#begin()
Plugin &#x27;VundleVim/Vundle.vim&#x27;
call vundle#end()
filetype plugin indent on

# 打开vim
# 进入命令模式，安装vundle
PluginInstall   # 显示Done则表示安装完成

# 接下来就可以用vundle进行vim的插件管理了
### 目录树插件 - nerdtree
# 使用教程: http://yang3wei.github.io/blog/2013/01/29/nerdtree-kuai-jie-jian-ji-lu/
# 安装nerdtree
vim /etc/vimrc 只需添加Plugin语句进行
&quot; vundle 插件管理器
set nocompatible
filetype off
set rtp+=~/.vim/bundle/Vundle.vim/
call vundle#begin()
Plugin &#x27;VundleVim/Vundle.vim&#x27;
Plugin &#x27;scrooloose/nerdtree&#x27;
Plugin &#x27;jistr/vim-nerdtree-tabs&#x27;
call vundle#end()
filetype plugin indent on

# vim-nerdtree-tabs 插件是为了解决nerdtree不能在tab之间共享的问题
# 一样的步骤，打开vim,进入命令模式，输入PluginInstall

# 插件配置
&quot; NERDTree 打开/关闭快捷键
map &lt;F2&gt; :NERDTreeToggle&lt;CR&gt;
&quot; NERDTree 默认路径配置
&quot; au VIMEnter * NERDTree D:\vim
let NERDTreeMinimalUI = 1   &quot; 关闭书签标签(&#x27;Press ? for help&#x27;)
&quot; 显示书签列表
let NERDTreeShowBookmarks=1
&quot; 显示行号
let NERDTreeShowLineNumbers=1
let NERDTreeAutoCenter=1
&quot; 显示隐藏文件
let NERDTreeShowHidden=1
&quot; 设置宽度
let NERDTreeWinSize=21
&quot; 在终端启动vim时，共享NERDTree
let g:nerdtree_tabs_open_on_console_startup=1
&quot; 忽略以下文件的显示
&quot; let NERDTreeIgnore=[&#x27;\.pyc&#x27;,&#x27;\~$&#x27;,&#x27;\.swp&#x27;]

### 搜索插件CtrlP
# 使用教程：http://www.boiajs.com/2014/12/17/vim-ctrlp
# 安装
Plugin &#x27;kien/ctrlp.vim&#x27;
# PluginInstall进行下载安装

### xterm和vim开启256色
# 查看终端类型
env|grep -i term
如果是xterm,则继续
vim /etc/profile.d/xterm.sh
if [ $TERM == &#x27;xterm&#x27; ]; then
    export TERM=xterm-256color
fi

. /etc/profile

vim /etc/vimrc
set t_Co=256

### syntax-markdown语法高亮文件
mkdir -p ~/.vim/syntax/
cd ~/.vim/syntax/
wget https://raw.github.com/plasticboy/vim-markdown/master/syntax/markdown.vim

vim ~/.vim/filetype.vim
au BufNewFile,BufRead *.md,*.markdown setf markdown
</code></pre>

### vimrc_centos
<pre><code class="language-vim line-numbers"><script type="text/plain">" xterm-256color
set t_Co=256

" vundle 插件管理器
set nocompatible
filetype off
set rtp+=~/.vim/bundle/Vundle.vim/
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
Plugin 'scrooloose/nerdtree'
Plugin 'jistr/vim-nerdtree-tabs'
Plugin 'kien/ctrlp.vim'
call vundle#end()
filetype plugin indent on

" 配色主题 molokai
colo molokai
" 字体 Consolas 12号大小
" set guifont=Consolas:h12

" tab替换为4个空格
set shiftwidth=4
set tabstop=4
set softtabstop=4
set expandtab
set autoindent

" 设置粘贴模式 F12
set pastetoggle=<F12>

" 使用语法高亮定义代码折叠
set foldmethod=syntax
" 打开文件时默认不折叠代码
set foldlevelstart=99

" 设置文件的代码形式 utf8
set encoding=utf-8
set termencoding=utf-8
set fileencoding=utf-8
set fileencodings=ucs-bom,utf-8,chinese,cp936
" gvim提示信息防止乱码
" language messages zh_CN.utf-8
" language messages en_US.utf-8
" set helplang=cn " 设置中文帮助
set history=1000    " 保留历史记录
set nu! " 设置显示行号
set wrap    " 设置自动换行
set linebreak   " 整词换行，与自动换行搭配使用
set list    " 显示制表符和行尾符$
set autochdir   " 自动设置当前目录为正在编辑的目录
set hidden  " 自动隐藏没有保存的缓冲区，切换buffer时不给出保存当前buffer的提示
set scrolloff=5 " 在光标接近底端或顶端时，自动下滚或上滚
set showtabline=2   " 设置显示标签栏
set autoread    " 设置当文件在外部被修改，自动更新该文件
set nobackup    " 不生成备份文件
set noundofile  " 不生成un~文件

set hlsearch    " 高亮显示查找结果
set incsearch   " 增量查找

" 状态栏的设置
set statusline=[%F]%y%r%m%*%=[Line:%l/%L,Column:%c][%p%%]   " 显示文件名：总行数，总的字符数
set ruler   " 在编辑过程中，在右下角显示光标位置的状态行

" 代码设置
syntax enable   " 打开语法高亮
syntax on   " 打开语法高亮
set showmatch   " 设置匹配模式,相当于括号匹配
set smartindent " 智能对齐
" set cursorcolumn  " 启用光标列
set cursorline  " 启用光标行
set guicursor+=a:blinkon0   " 设置光标不闪烁
set showcmd " 状态栏显示当前执行的命令
set fdm=indent

" ========= Plugin 配置 =========
" NERDTree 打开/关闭快捷键
map <F2> :NERDTreeToggle<CR>
" NERDTree 默认路径配置
" au VIMEnter * NERDTree D:\vim
let NERDTreeMinimalUI = 1   " 关闭书签标签('Press ? for help')
" 显示书签列表
let NERDTreeShowBookmarks=1
" 显示行号
let NERDTreeShowLineNumbers=1
let NERDTreeAutoCenter=1
" 显示隐藏文件
let NERDTreeShowHidden=1
" 设置宽度
let NERDTreeWinSize=21
" 在终端启动vim时，共享NERDTree
let g:nerdtree_tabs_open_on_console_startup=1
" 忽略以下文件的显示
" let NERDTreeIgnore=['\.pyc','\~$','\.swp']

" map快捷键映射
" map <A-o> :tabnew<CR>
" map <A-w> :q!<CR>
" map <A-W> :qa!<CR>
" map <A-s> :mks! $VIM/_Session.vim<CR>
" map <A-l> :so $VIM/_Session.vim<CR>
</script></code></pre>

### Windows - gVim
<pre><code class="language-vim line-numbers">### 环境：Windows 10 x86_64 LTSB 2016  gVim 8.0 x86
# windows下也是能够使用vim的，官方的gvim就是windows下的gui增强版vim

### 安装gvim
下载：https://ftp.nluug.nl/pub/vim/pc/gvim80-069.exe
# 直接安装就行，安装目录建议不要放在系统盘C盘，会有权限问题

### 安装msysgit
下载：https://github.com/git-for-windows/git/releases/download/v2.12.2.windows.2/Git-2.12.2.2-64-bit.exe
# 默认安装就行，会自动添加到Path环境变量
# 进入cmd, 输入 git --version 如果能显示版本号，表示安装成功

### 安装curl
下载：https://dl.uxnr.de/build/curl/curl_winssl_msys2_mingw64_stc/curl-7.53.1/curl-7.53.1.zip
# 解压到一个目录
# 添加环境变量，Path -&gt; curl解压目录/bin
# Path环境变量不会设置具体百度 or Google
# 进入cmd, 输入 curl --version 如果显示版本号，表示安装成功

### 修改vim的主题和字体
# 下载主题: https://github.com/tomasr/molokai
# 把主题文件放到 $vim/vim80/colors/molokai.vim
# $vim/_vimrc
&quot; 配色主题 molokai
colo molokai
&quot; 字体 Consolas 12号大小
set guifont=Consolas:h12

### 安装vundle插件
# 打开cmd, 切换到vim的安装目录
# 运行 git clone https://github.com/VundleVim/Vundle.vim.git
# 资源管理器 -&gt; vim安装目录 -&gt; vimfiles -&gt; 新建文件夹 bundle -&gt; 将Vundle.vim移动到bundle下命名为vundle.vim
# 添加一个环境变量 vim 值为vim的安装路径
# 编辑 $vim/_vimrc

&quot; vundle 插件管理器
set nocompatible
filetype off
set rtp+=$vim/vimfiles/bundle/Vundle.vim/
call vundle#begin(&#x27;$vim/vimfiles/bundle/&#x27;)
Plugin &#x27;VundleVim/Vundle.vim&#x27;
Plugin &#x27;scrooloose/nerdtree&#x27;
Plugin &#x27;jistr/vim-nerdtree-tabs&#x27;
Plugin &#x27;kien/ctrlp.vim&#x27;
call vundle#end()
filetype plugin indent on

# 打开gvim -&gt; :PluginInstall 安装插件
### 插件设置
&quot; NERDTree 打开/关闭快捷键
map &lt;F2&gt; :NERDTreeToggle&lt;CR&gt;
&quot; NERDTree 默认路径配置
&quot; au VIMEnter * NERDTree D:\vim
autocmd VimEnter * wincmd w
let NERDTreeMinimalUI = 1   &quot; 关闭书签标签(&#x27;Press ? for help&#x27;)
&quot; 显示书签列表
let NERDTreeShowBookmarks=1
&quot; 显示行号
let NERDTreeShowLineNumbers=1
let NERDTreeAutoCenter=1
&quot; 显示隐藏文件
let NERDTreeShowHidden=1
&quot; 设置宽度
let NERDTreeWinSize=21
&quot; 在终端启动vim时，共享NERDTree
let g:nerdtree_tabs_open_on_console_startup=1
&quot; 忽略以下文件的显示
&quot; let NERDTreeIgnore=[&#x27;\.pyc&#x27;,&#x27;\~$&#x27;,&#x27;\.swp&#x27;]
</code></pre>

### vimrc_windows
<pre><code class="language-vim line-numbers"><script type="text/plain">" vundle 插件管理器
set nocompatible
filetype off
set rtp+=$vim/vimfiles/bundle/Vundle.vim/
call vundle#begin('$vim/vimfiles/bundle/')
Plugin 'VundleVim/Vundle.vim'
Plugin 'scrooloose/nerdtree'
Plugin 'jistr/vim-nerdtree-tabs'
Plugin 'kien/ctrlp.vim'
call vundle#end()
filetype plugin indent on

" 配色主题 molokai
colo molokai
" 字体 Consolas 12号大小
set guifont=Consolas:h12

" Windows下窗口自动全屏
autocmd GUIEnter * simalt ~x

" tab替换为4个空格
set shiftwidth=4
set tabstop=4
set softtabstop=4
set expandtab
set autoindent

" 设置粘贴模式 F12
set pastetoggle=<F12>

" 使用语法高亮定义代码折叠
set foldmethod=syntax
" 打开文件时默认不折叠代码
set foldlevelstart=99

" 设置文件的代码形式 utf8
set encoding=utf-8
set termencoding=utf-8
set fileencoding=utf-8
set fileencodings=ucs-bom,utf-8,chinese,cp936
" gvim菜单防止乱码
source $VIMRUNTIME/delmenu.vim
source $VIMRUNTIME/menu.vim
" gvim提示信息防止乱码
language messages zh_CN.utf-8
set helplang=cn " 设置中文帮助
set history=1000    " 保留历史记录
set nu! " 设置显示行号
set wrap    " 设置自动换行
set linebreak   " 整词换行，与自动换行搭配使用
set list    " 显示制表符和行尾符$
set autochdir   " 自动设置当前目录为正在编辑的目录
set hidden  " 自动隐藏没有保存的缓冲区，切换buffer时不给出保存当前buffer的提示
set scrolloff=5 " 在光标接近底端或顶端时，自动下滚或上滚
set showtabline=2   " 设置显示标签栏
set autoread    " 设置当文件在外部被修改，自动更新该文件
set mouse=a " 设置在任何模式下鼠标都可用
set guioptions-=T   " 隐藏gvim工具栏
set guioptions-=m   " 隐藏gvim菜单栏
set nobackup    " 不生成备份文件
set noundofile  " 不生成un~文件

set hlsearch    " 高亮显示查找结果
set incsearch   " 增量查找

" 状态栏的设置
set statusline=[%F]%y%r%m%*%=[Line:%l/%L,Column:%c][%p%%]   " 显示文件名：总行数，总的字符数
set ruler   " 在编辑过程中，在右下角显示光标位置的状态行

" 代码设置
syntax enable   " 打开语法高亮
syntax on   " 打开语法高亮
set showmatch   " 设置匹配模式,相当于括号匹配
set smartindent " 智能对齐
" set cursorcolumn  " 启用光标列
set cursorline  " 启用光标行
set guicursor+=a:blinkon0   " 设置光标不闪烁
set showcmd " 状态栏显示当前执行的命令
set fdm=indent

" ========= Plugin 配置 =========
" NERDTree 打开/关闭快捷键
map <F2> :NERDTreeToggle<CR>
" NERDTree 默认路径配置
au VIMEnter * NERDTree D:\vim
autocmd VimEnter * wincmd w
let NERDTreeMinimalUI = 1   " 关闭书签标签('Press ? for help')
" 显示书签列表
let NERDTreeShowBookmarks=1
" 显示行号
let NERDTreeShowLineNumbers=1
let NERDTreeAutoCenter=1
" 显示隐藏文件
let NERDTreeShowHidden=1
" 设置宽度
let NERDTreeWinSize=21
" 在终端启动vim时，共享NERDTree
let g:nerdtree_tabs_open_on_console_startup=1
" 忽略以下文件的显示
" let NERDTreeIgnore=['\.pyc','\~$','\.swp']

" map快捷键映射
map <A-o> :tabnew<CR>
map <A-c> :tabc<CR>
map <A-C> :qa!<CR>
map <A-n> :tabn<CR>
map <A-p> :tabp<CR>
map <A-s> :mks! $vim/_Session.vim<CR>
map <A-l> :so $vim/_Session.vim<CR>
nmap <S-v> <S-v>
vmap <S-v> <S-v>
nmap <A-v> v
vmap <A-v> v
nmap <C-v> <C-v>
vmap <C-v> <C-v>
</script></code></pre>
