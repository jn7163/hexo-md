---
title: nginx配置文件语法不能高亮问题
date: 2016-10-16 16:16:20
categories:
- linux
- 基础命令
- vim
tags:
- linux
- 基础命令
- vim
- nginx
- 语法高亮
keywords: nginx配置文件语法高亮
---
> 
今天又遇到了nginx配置文件用vim编辑语法无法高亮的问题,无论怎么重装vim和nginx都不管用，到现在才知道是vim没配置好!

<!-- more -->

## 重新下载nginx语法文件
<pre><code class="language-vim line-numbers">cd /usr/share/vim/vim74/syntax/
wget http://www.vim.org/scripts/download_script.php?src_id=14376 -O nginx.vim
echo 'au BufNewFile,BufRead /etc/nginx/*.conf setf nginx' >> ../filetype.vim
</code></pre>

## vim filetype
<pre><code class="language-vim line-numbers">vim检测文件的filetype一般有两种方式:
1. filetype.vim指定
比如上面的，只要匹配到"/etc/nginx/*.conf", vim就会加载nginx语法高亮文件

2. 手动指定
命令行模式 setf nginx
或者在文件中声明 "# vim: ft=nginx"

比如shell脚本
cat test
#!/bin/bash
# vim: ft=sh

注意: filetype声明行前面的注释符号没有限定，可以是# // "等等，符合语言规范即可
如:
vim     " vim: ft=vim
shell   # vim: ft=sh
c       // vim: ft=c
js      // vim: ft=javascript
python  # vim: ft=python
php     // vim: ft=php
</code></pre>
