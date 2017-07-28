---
title: python3 模块
date: 2016-12-09 11:20:00
categories:
- python
tags:
- python
keywords: python, python3, 模块
---
> 
python3 模块
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

> 
使用模块有什么好处？
最大的好处是大大提高了代码的可维护性。
其次，编写代码不必从零开始。
当一个模块编写完毕，就可以被其他地方引用。
我们在编写程序的时候，也经常引用其他模块，
包括Python内置的模块和来自第三方的模块。
使用模块还可以避免函数名和变量名冲突。
相同名字的函数和变量完全可以分别存在不同的模块中，
因此，我们自己在编写模块时，不必考虑名字会与其他模块冲突。
但是也要注意，尽量不要与内置函数名字冲突
你也许还想到，如果不同的人编写的模块名相同怎么办？
为了避免模块名冲突，Python又引入了按目录来组织模块的方法，称为包(Package)

<pre><code class="language-python line-numbers">./my_test
    __init__.py
    hello.py
    new/
        __init__.py
        new.py

__init__.py 的模块名就是 my_test

import my_test 导入的就是 __init__.py

import my_test.hello 导入的就是 hello.py

#

import my_test # 导入__init__.py
my_test.test('Otokaze') # 使用__init__.py中的函数test

import my_test.new.new # 导入new文件夹的new.py
my_test.new.new.new('Otokaze') #使用new.py 的函数new
</code></pre>

## 导入模块的两种方式
<pre><code class="language-python line-numbers">import os
os.system('ls -l')

from os import system
system('ls -l')

from os import system as bash
bash('ls -l')

from os import * # 导入所有（除 _ 开头的私有变量）
</code></pre>

## 编写模块
<pre><code class="language-python line-numbers">#!/usr/bin/python
# -*- coding: utf-8 -*-

'This is Otokaze first module'

__author__='Otokaze'

from os import system as sh

if __name__=='__main__':
    sh('ls -l')

第一个字符串为文档注释

author为作者

最后的为最常见的运行测试语句
</code></pre>

## 私有变量、特殊变量
<pre><code class="language-python line-numbers">正常的函数和变量名是"公开的"（public），可以被直接引用
比如：abc，x123，PI等；

类似__xxx__这样的变量是"特殊变量"，可以被直接引用，但是有特殊用途

比如上面的__author__，__name__就是特殊变量，
hello模块定义的文档注释也可以用特殊变量__doc__访问，我们自己的变量一般不要用这种变量名；

类似_xxx和__xxx这样的函数或变量就是"非公开的"（private），不应该被直接引用，比如_abc，__abc等；

之所以我们说，private函数和变量“不应该”被直接引用，而不是“不能”被直接引用，是因为Python并没有一种方法可以完全限制访问private函数或变量

但是，从编程习惯上不应该引用private函数或变量。

单下划线开头的变量为私有变量 用 from test import * 将不会引用单下划线开头的变量

双下划线开头的变量为class中使用的 _Student__age --> __age 

双下划线开头和结尾的变量为特殊变量
</code></pre>

## 安装第三方模块
<pre><code class="language-python line-numbers">pip install Pillow # 著名的图片处理库
pip install mysql-connector # mysql/mariadb驱动
pip install requests # 比自带的urllib好用
pip install beautifulsoup4 # xml/html解析模块
</code></pre>

## 模块搜索路径
<pre><code class="language-python line-numbers">默认在当前路径、所有已安装的内置模块和第三方模块的路径

可以通过sys模块的path变量查看

import sys
sys.path

添加自定义搜索路径

1. sys.path.append('/root/my_mod') # 临时生效
2. PYTHONPATH 设置环境变量 只需填上要添加的路径即可(暂测没用)
</code></pre>
