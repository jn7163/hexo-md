---
title: sed详解
date: 2016-11-02 17:29:29
categories:
- linux
- 文本工具
tags:
- linux
- 文本工具
- sed
keywords: sed, 文本处理
---
> 
sed：Stream EDitor，流编辑器，默认只处理模式空间，不处理原数据
如果你处理的数据是针对行进行处理的，可以使用sed, 配合强大的regexp，能做很多事

<!-- more -->

## sed命令

> sed和cut一样，都是以"行"为处理单位。
sed 用来把文档或字符串里面的文字经过一系列编辑命令转换为另一种格式输出。
sed 通常用来匹配一个或多个正则表达式的文本进行处理。
sed 是一种在线编辑器，它一次处理一行内容。
处理时，把当前处理的行存储在临时缓冲区中，称为"模式空间"(pattern space)，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。
接着处理下一行，这样不断重复，直到文件末尾。
文件内容并没有改变，除非你使用重定向存储输出。

## 命令格式
<pre><code class="language-bash line-numbers">sed 'AddressCommand' file
# Address-定位，Command-执行的命令
</code></pre>

## 常用参数
<pre><code class="language-bash line-numbers">-n # 静默模式，不再显示模式空间的内容
-i # 直接修改源文件
-e Command -e Command # 同时执行多个命令
-f file # 调用sed命令文件file
-r # 使用扩展正则表达式
</code></pre>

## Address
<pre><code class="language-bash line-numbers">StartLine,EndLine # 开始行,结束行
2,10 表示第2到10行
2,$ 表示第二行到最后一行

/RegExp/ # 正则表达式
/^#/ 匹配注释行

/pattern1/,/pattern2/ # 表示第一次被pattern1匹配行开始到第一次被pattern2匹配行之间的所有行

LineNumber # 指定行
10 表示第十行

StartLine,+n # 表示从StartLine开始向后的n行
10,+10 # 表示从第十行至第二十行之间的所有行
</code></pre>

## Command
<pre><code class="language-bash line-numbers">d # 删除匹配行
p # 打印匹配行,在不使用-n选项时被匹配到的行会显示两遍,因为sed处理时默认会把处理的信息输出
a \string # 在指定行下面追加新行，内容为"string",追加多行,在每行后加\n进行换行
i \string # 在指定行上面添加新行，内容为"string"
r file # 将指定文件的内容添加至符合条件的文件中
w file # 将地址指定范围内的行另存至指定的文件中
s/pattern/string/修饰符
# 查找并替换,默认只替换每行中第一次匹配到的字符串
# 修饰符:
# g # 全局替换
# i # 忽略大小写
# s///,s###,s@@@等都行,当所使用的分割符号与内容中显示的相同时,需使用转义字符转义
& # 引用模式匹配的整个字符串
</code></pre>

## 实例
<pre><code class="language-bash line-numbers"># 测试的文件
cat test
Beijing 2001
Beijing 2002
Beijing 2003
Beijing 2004
Beijing 2005
Beijing 2006
# 删除第1至3行
sed '1,3d' test
Beijing 2004
Beijing 2005
Beijing 2006
# 不加n打印行
sed '1p' test
Beijing 2001
Beijing 2001
Beijing 2002
Beijing 2003
Beijing 2004
Beijing 2005
Beijing 2006
# 加n打印行
sed -n '1p' test
Beijing 2001
# 追加文本
sed '/2001/a\123' test
Beijing 2001
123
Beijing 2002
Beijing 2003
Beijing 2004
Beijing 2005
Beijing 2006
# 追加多行文本
sed '/2001/a\123\n123' test
Beijing 2001
123
123
Beijing 2002
Beijing 2003
Beijing 2004
Beijing 2005
Beijing 2006
# 指定行另存为
sed '1,2w 1-2.txt' test
cat 1-2.txt
Beijing 2001
Beijing 2002
# 读取文件内容至指定行后
sed '2r 1-2.txt' test
Beijing 2001
Beijing 2002
Beijing 2001
Beijing 2002
Beijing 2003
Beijing 2004
Beijing 2005
Beijing 2006
# 正则替换
sed 's/Beijing/BEIJING/' test
BEIJING 2001
Beijing 2002
Beijing 2003
Beijing 2004
Beijing 2005
Beijing 2006
# 正则替换(全局)
sed 's/Beijing/BEIJING/g' test
BEIJING 2001
BEIJING 2002
BEIJING 2003
BEIJING 2004
BEIJING 2005
BEIJING 2006
# 扩展正则表达式
sed -r 's/Beijing/BEIJING/g;s/BEI(.*)/bei\1/g' test
或
sed -r -e 's/Beijing/BEIJING/g' -e 's/BEI(.*)/bei\1/g' test
beiJING 2001
beiJING 2002
beiJING 2003
beiJING 2004
beiJING 2005
beiJING 2006
# &引用
sed 's/Beijing/&666/g' test
Beijing666 2001
Beijing666 2002
Beijing666 2003
Beijing666 2004
Beijing666 2005
Beijing666 2006
# 小技巧-鉴别连续空格、tab
# a文件第一行间隔符为8个空格
# a文件第二行间隔符为1个tab
[root@zfl9 ~]# cat a
abc     123
abc	123
[root@zfl9 ~]# sed -n l a
abc     123$
abc\t123$

## sed 不支持正则非贪婪匹配
</code></pre>
