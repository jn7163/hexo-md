---
title: Linux文本处理工具
date: 2016-10-26 13:28:22
categories:
- linux
- 文本工具
tags:
- linux
- 文本工具
keywords: 文本处理, find, grep, xargs, sort, uniq, tr, cut, paste, wc
---
> 
本节介绍Linux下使用Shell处理文本时最常用的工具：find、grep、xargs、sort、uniq、tr、cut、paste、wc
提供的参数都是最常用的

<!-- more -->

## find
### 文件名`name`
<pre><code class="language-bash line-numbers">find . -name "*.txt"
</code></pre>

### 文件大小`size`
<pre><code class="language-bash line-numbers">find . -size 1k # 等于1k
find . -size +1k # 大于1k
find . -size -1k # 小于1k
find . -empty # 空文件(夹)
</code></pre>

### 权限`perm`
<pre><code class="language-bash line-numbers">find . -perm 777
</code></pre>

### 类型`type`
<pre><code class="language-bash line-numbers">find . -type d|f|l # (目录|普通文件|链接文件)
</code></pre>

### 所属用户(组)`user|group`
<pre><code class="language-bash line-numbers">find . -user root
find . -group root
find . -uid|gid 0 # uid|gid 为 0
find . -nouser|nogroup # (其他用户|其他用户组)
</code></pre>

### 正则`regex`
<pre><code class="language-bash line-numbers">find . -regex ".*\.txt" # 纯文本
# -iregex 忽略大小写
</code></pre>

### 时间`time`
<pre><code class="language-bash line-numbers">find . -atime 1 # 访问时间距现在1天(time单位1天;min单位1分钟-下同)的文件(夹)
find . -atime +1 # 1天前访问的文件(夹) (类似用法同size，下同)
find . -atime -1 # 1天之内访问的文件(夹)
find . -amin 1
find . -ctime 1 # 创建时间(文件属性修改时间)
find . -mtime 1 # 内容修改时间
</code></pre>

### 搜索深度`maxdepth`
<pre><code class="language-bash line-numbers">find . -maxdepth 1 -name "*.txt" # 仅当前文件夹下查找,不搜索子文件夹
</code></pre>

### 逻辑连接符
<pre><code class="language-bash line-numbers">find . -type f -size -10k # 普通文件"且"大小小于10k
find . -type f -o -size -10k # 普通文件"或"大小小于10k
find . ! -type f # 除普通文件外
</code></pre>

### 动作`exec`
<pre><code class="language-bash line-numbers">find . -name &quot;*.txt&quot; -exec tar -czvf txt.tar.gz {} \;
# {}代表find查找到的文件，此处为所有txt文件
# \; 为结束符
</code></pre>

## grep
<pre><code class="language-bash line-numbers">grep -ni --color "root" /etc/passwd # i为忽略大小写,n为打印所在行号,color为匹配结果高亮
grep -v "root" /etc/passwd # 取反
grep -c "root" /etc/passwd # 统计root匹配的次数
grep -o "root" /etc/passed # 只显示匹配文本，不打印整行
grep -R "root" . # 递归查找
grep -l|L "root" /etc/passwd /etc/fstab /etc/inittab # 不直接显示匹配结果所在行，显示包含(不包含)匹配项的文件
grep -B|A|Cx "root" /etc/passwd # 同时打印(前|后|前后)x行的内容
</code></pre>

## xargs
> xargs 能够将输入数据转化为特定命令的命令行参数；这样，可以配合很多命令来组合使用。

<pre><code class="language-bash line-numbers">## 常用参数
-d      指定定界符(默认为空格,多行的为\n换行符)
-t      显示执行的命令
-n      指定参数个数
-i      指定replace符,默认{}

xargs默认执行的命令为"echo", 参数个数为全部

echo '1;2;3;4;5' | xargs -d ';' -n 1
1
2
3
4
5

xargs --- 批量链接
[root@localhost ~]# ll
total 0
-rw-r--r-- 1 root root 0 Jan 6 14:35 1.so
-rw-r--r-- 1 root root 0 Jan 6 14:35 2.so
-rw-r--r-- 1 root root 0 Jan 6 14:35 3.so
-rw-r--r-- 1 root root 0 Jan 6 14:35 4.so
-rw-r--r-- 1 root root 0 Jan 6 14:35 5.so
-rw-r--r-- 1 root root 0 Jan 6 14:35 6.so
-rw-r--r-- 1 root root 0 Jan 6 14:35 7.so
-rw-r--r-- 1 root root 0 Jan 6 14:35 8.so
-rw-r--r-- 1 root root 0 Jan 6 14:35 9.so
drwxr-xr-x 2 root root 6 Jan 6 14:44 test
[root@localhost ~]# ll test/
total 0
[root@localhost ~]# ls | grep 'so' | xargs -i -n1 ln -s `pwd`/{} test/{}
[root@localhost ~]# ll test/
total 0
lrwxrwxrwx 1 root root 10 Jan 6 14:45 1.so -> /root/1.so
lrwxrwxrwx 1 root root 10 Jan 6 14:45 2.so -> /root/2.so
lrwxrwxrwx 1 root root 10 Jan 6 14:45 3.so -> /root/3.so
lrwxrwxrwx 1 root root 10 Jan 6 14:45 4.so -> /root/4.so
lrwxrwxrwx 1 root root 10 Jan 6 14:45 5.so -> /root/5.so
lrwxrwxrwx 1 root root 10 Jan 6 14:45 6.so -> /root/6.so
lrwxrwxrwx 1 root root 10 Jan 6 14:45 7.so -> /root/7.so
lrwxrwxrwx 1 root root 10 Jan 6 14:45 8.so -> /root/8.so
lrwxrwxrwx 1 root root 10 Jan 6 14:45 9.so -> /root/9.so
</code></pre>

## wc
<pre><code class="language-bash line-numbers">wc -l file # 行数
wc -L file # 最长行的长度
wc -w file # 字数
wc -c file # 字节数
wc -m file # 字符数
wc file # 依次显示行数 字数 字节数 文件名称
</code></pre>

## sort
> sort将文件的每一行作为一个单位，相互比较，比较原则是从首字符向后，依次按ASCII码值进行比较，最后将他们按升序输出

### 常用参数
<pre><code class="language-bash line-numbers">sort file # 默认升序输出
sort -u file # 删除重复行(只是在输出结果中删除,不改变源文件)
sort -r file # 降序输出
sort file -o file # 默认升序输出，并写入源文件(直接用重定向源文件会被清空！)
sort -n file # 按数值升序输出(不加n选项会出现12比2小的情况,因为sort先比较第一位数字)
sort -f file # 忽略大小写
</code></pre>

### `-t`和`-k`选项
<pre><code class="language-bash line-numbers"># 有一个文件是这样的
cat test
banana:30:5.5
apple:10:2.5
pear:90:2.3
orange:20:3.4

我想以第二列升序排序，怎么处理
sort -n -t ':' -k 2 test # t指定定界符,k指定列
</code></pre>

### 扩展
<pre><code class="language-bash line-numbers">cat test
google 110 5000 # 依次为公司名、人数、工资
baidu 100 5000
guge 50 3000
sohu 100 4500

## 我想以公司人数升序排序
sort -n -t ' ' -k 2 test

这时发现人数相同的baidu排在sohu上面，因为相同的话，默认以第一个域升序排序，所以baidu排在sohu前面
那我们怎么自定义相同的多个元素如何排序呢？

## 以公司人数升序排序，人数相同的以工资降序排序
sort -n -t ' ' -k 2 -k 3r test

## 以公司名字第二个字母升序排序
sort -t ' ' -k 1.2 test

发现sohu排在google前面，原因就不说了，和上面的一样

## 仅以公司名字第二个字母升序排序，相同的以工资降序排序
sort -t ' ' -k 1.2,1.2 -k 3,3nr
</code></pre>

## uniq
<pre><code class="language-bash line-numbers">sort test | uniq # 消除重复行
sort test | uniq -c # 统计重复行出现次数
sort test | uniq -d # 找出重复行
</code></pre>

## tr
<pre><code class="language-bash line-numbers"># tr只处理stdin标准输入信息，后面如果要接文件,请加上重定向符
tr 'a-z' 'A-Z' < test # 大小写转换
tr ' ' '\t' < test # 空格转换为tab
tr -s ' ' ' ' < test # 将多个连续空格换成单个空格
tr -d ' ' < test # 删除所有空格
</code></pre>

## cut
> 
正如其名，cut的工作就是"剪"，具体的说就是在文件中负责剪切数据用的
cut是以每一行为一个处理对象的
cut一般以什么为依据呢? 也就是说，我怎么告诉cut我想定位到的剪切内容呢?
cut命令主要是接受三个定位方法：
`字节（bytes），用选项-b`
`字符（characters），用选项-c`
`域（fields），用选项-f`

### 字节
<pre><code class="language-bash line-numbers">echo 'abcdefg' | cut -b 2
echo 'abcdefg' | cut -b 2,4-6 # 第二个字节和第4-6个字节 (类似用法下同)
echo 'abcdefg' | cut -b -3 # 前三个字节
echo 'abcdefg' | cut -b 3- # 从第三个字节到结尾
</code></pre>

### 字符
<pre><code class="language-bash line-numbers">echo 'abcdefg' | cut -c 2
</code></pre>

### 域
<pre><code class="language-bash line-numbers">head -5 /etc/passwd | cut -d ':' -f 1
# -d指定定界符(默认为tab)，-f指定域
</code></pre>

## paste
> 这个很简单，和cut的原理几乎一样，就是将几个文件的相应行用制表符连接起来，并输出到标准输出

<pre><code class="language-bash line-numbers">cat a
1
2
3

cat b
a
b
c

paste a b
1	a
2	b
3	c

paste a b | sed -n l
1\ta$ # 默认以tab作为定界符
2\tb$
3\tc$

paste -d ':' a b # d指定定界符

paste -s -d ':' a b
1:2:3
a:b:c
# -s以文件为处理单位，说白了就是旋转了一下
</code></pre>

## diff
<pre><code class="language-bash line-numbers">diff 文本比较工具
[root@zfl ~]# ll
total 8
-rw-r--r-- 1 root root 10 Dec 16 15:41 a
-rw-r--r-- 1 root root 10 Dec 16 15:42 b
[root@zfl ~]# cat a
a
a
a
a
a
[root@zfl ~]# cat b
a
a
b
a
a
[root@zfl ~]# diff a b
3c3
< a
---
> b
[root@zfl ~]# diff -c a b
*** a	2016-12-16 15:41:53.554976421 +0800
--- b	2016-12-16 15:42:01.259576680 +0800
***************
*** 1,5 ****
a
a
! a
a
a
--- 1,5 ----
a
a
! b
a
a
[root@zfl ~]# diff -u a b
--- a	2016-12-16 15:41:53.554976421 +0800
+++ b	2016-12-16 15:42:01.259576680 +0800
@@ -1,5 +1,5 @@
a
a
-a
+b
a
a
</code></pre>
