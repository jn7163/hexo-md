---
title: python3 函数式编程
date: 2016-12-09 11:14:00
categories:
- python
tags:
- python
keywords: python, python3, 函数式编程
---
> 
python3 函数式编程
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

## 概念
> 
先搞清楚几样东西：
函数名、函数体、返回值，函数的内存地址、函数名加括号、函数名被当作参数、函数名加括号被当作参数、返回函数名、返回函数名加括号

<pre><code class="language-python line-numbers">def foo():
    print("让我们干点啥！")
    return "ok"

foo()

函数名:          foo (函数名其实就是指向函数的变量)

函数体:          第1-3行

返回值:          字符串“ok”(如果不显式给出return的对象，那么默认返回None)

函数的内存地址:   当函数体被读进内存后的保存位置，它由标识符即函数名foo引用，也就是说foo指向的是函数体在内存内的保存位置

函数名加括号:     例如foo()，函数的调用方法，只有见到这个括号，程序会根据函数名从内存中找到函数体，然后执行它
</code></pre>

## 高阶函数
既然变量可以指向函数，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数

### map
<pre><code class="language-python line-numbers">map()函数接收两个参数，一个是函数，一个是Iterable，
map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回

list(map(lambda x:x**3,[1,2,3]))
 由于map返回的是一个Iterable可迭代对象，所以需要使用list()转换为list类型
</code></pre>

### reduce
<pre><code class="language-python line-numbers">reduce()把一个函数作用在一个Iterable上，
这个函数必须接收两个参数，
reduce把结果继续和序列的下一个元素做累积计算，其效果就是

reduce(func,[1,2,3]) <==> func(func(1,2),3)

from functools import reduce
reduce(lambda x,y:10*x+y,[1,2,3])
123
</code></pre>

### filter
<pre><code class="language-python line-numbers">和map()类似，filter()也接收一个函数和一个序列。
和map()不同的是，filter()把传入的函数依次作用于每个元素，
然后根据返回值是True还是False决定保留还是丢弃该元素

取一个列表的所有偶数

filter(lambda x:x%2==0,[1,2,3,4,5])
</code></pre>

### str 的 strip 方法
<pre><code class="language-python line-numbers">s='abccc 123 acccc'
s.strip('abc') # 边界
删除边界的'a','b','c'字符，直到没有匹配的字符为止
s.lstrip('abc') # 左边界
s.rstrip('abc') # 右边界
s.strip() # 不加参数删除边界的空值，空字符串
</code></pre>

### sorted
<pre><code class="language-python line-numbers">sorted()函数也是一个高阶函数，
它还可以接收一个key函数来实现自定义的排序

key指定的函数将作用于list的每一个元素上，
并根据key函数返回的结果进行排序
要进行反向排序，不必改动key函数，可以传入第三个参数reverse=True

默认情况下，对字符串排序，是按照ASCII的大小比较的，

由于'Z' < 'a'，结果，大写字母Z会排在小写字母a的前面
给sorted传递一个key参数就可以实现我们想要的字母表排序

sorted(['a','B'],key=str.lower)
</code></pre>

## 返回函数
> 
函数不仅可以作为参数传入，也可以作为返回值返回
返回闭包时牢记的一点就是：
返回函数不要引用任何循环变量，或者后续会发生变化的变量

## lambda 匿名函数
<pre><code class="language-python line-numbers">匿名函数有个限制，就是只能有一个表达式，不用写return，返回值就是该表达式的结果

lambda x:x*x

没有函数名，函数参数为x，返回值为x*x
</code></pre>

## 装饰器 decorator
<pre><code class="language-python line-numbers">现在，假设我们要增强now()函数的功能，
比如，在函数调用前后自动打印日志，但又不希望修改now()函数的定义
这种在代码运行期间动态增加功能的方式，称之为 "装饰器" (decorator)

装饰器通常用于在不改变原有函数代码和功能的情况下，为其添加额外的功能。

比如在原函数执行前先执行点什么，在执行后执行点什么。

装饰器函数的规则是：
"被装饰的函数名会被当作参数传递给装饰函数。装饰函数执行它自己内部的代码后，会将它的返回值赋值给被装饰的函数。"

装饰器内部需要使用两层函数
不然在调用装饰器时就执行了装饰器语句

eg:

# 在执行f1之前进行认证，执行f1后进行日志记录 ----单个装饰器

def outer(func):
    def inner(*args,**kw):
        print('认证成功！')
        func(*args,**kw)
        print('日志添加成功！')
    return inner
    
@outer
def f1(name):
    print('Hello,%s!'%name)
    
f1('Otokaze')
认证成功！
Hello,Otokaze!
日志添加成功！

eg:

# 多个装饰器

def outer1(func):
    def inner(*args,**kw):
        print('Begin')
        func(*args,**kw)
        print('End')
    return inner
    
def outer2(func):
    def inner(*args,**kw):
        print('认证成功！')
        func(*args,**kw)
        print('日志添加成功！')
    return inner

@outer1
@outer2
def f1(name):
    print('Hello,%s!'%name)

f1('Otokaze')

Begin
认证成功！
Hello,Otokaze!
日志添加成功！
End

# 装饰器带参数

def INIT(test):
    def outer(func):
        def inner(*args,**kw):
            print('认证成功！----%s---'%test)
            func(*args,**kw)
            print('日志添加成功！----%s----'%test)
        return inner
    return outer
    
@INIT('ZFLYUN')
def f1(name):
    print('Hello,%s!'%name)
    
f1('Otokaze')

认证成功！----ZFLYUN---
Hello,Otokaze!
日志添加成功！----ZFLYUN----


##################################
装饰器部分 更标准的写法是：
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def outer(func):
    def inner(*args,**kw):
        print('auth successful!')
        result=func(*args,**kw)
        print('added logs!')
        return result
    return inner

@outer
def f1(name):
    print('my name is %s!' % name)

f1('Otokaze')
</code></pre>

### 函数的__name__属性：获取函数名称
> 
因为我们讲了函数也是对象，它有__name__等属性，
但你去看经过decorator装饰之后的函数，它们的__name__已经从原来的'f1'变成了'inner'

<pre><code class="language-python line-numbers">所以需要加入一行代码：

import functools
@functools.wraps(func)

# 加在inner前面 即可获取原始函数名

</code></pre>

## 偏函数 partial
<pre><code class="language-python line-numbers">import functools
max10=functools.partial(max,10) # 固定一个参数10
</code></pre>
