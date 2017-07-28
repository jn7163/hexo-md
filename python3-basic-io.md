---
title: python3 输入与输出
date: 2016-12-09 11:09:00
categories:
- python
tags:
- python
keywords: python, python3, 输入与输出
---
> 
python3 输入与输出
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

## 输出 `print()`
<pre><code class="language-python line-numbers">print('Hello,world!')
# 多个参数用'逗号'隔开,'逗号'打印显示为'空格'，‘加号’用于拼接多个字符串或数学运算
#
# 使用格式化字符串
print('Hi,%s,you have %d monly!' % ('Otokaze',100))
%d  整数
%f  浮点数
%s  字符串
%x  十六进制数
# 注意一一对其
# 当不知道数据类型时，可全部使用%s(全部转换为字符串数据类型)
# 其中，格式化整数和浮点数还可以指定是否补0和整数与小数的位数
>>> '%2d-%02d' % (3, 1)
'3-01'
>>> '%.2f' % 3.1415926
'3.14'
# 有些时候，字符串里面的%是一个普通字符怎么办？这个时候就需要转义，用%%来表示一个%
</code></pre>

## 输入 `input()`
<pre><code class="language-python line-numbers">name=input('Please enter your name:')
# input()获取的数据类型为'字符串'
# 如果需要转换请使用int()等函数转换为'整数'或'其他数据类型'！
</code></pre>
