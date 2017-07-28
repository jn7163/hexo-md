---
title: python3 函数
date: 2016-12-09 11:11:00
categories:
- python
tags:
- python
keywords: python, python3, 函数
---
> 
python3 函数
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

## 调用函数
<pre><code class="language-python line-numbers">常用：

abs() # 绝对值
max() # max值
min() # min值
upper() # 转换为大写
lower() # 转换为小写

8e3 &lt;-&gt; 8*10^3 # 10的n次幂
8**3 &lt;-&gt; 8*8*8 # 乘方
hex(22) # 函数把一个整数转换成十六进制表示的字符串

int(&#x27;123&#x27;) # 转换为整数类型
float(&#x27;123&#x27;) # 转换为浮点数
str(123) # 转换为字符串
bool(1) # 转换为布尔值
bool(0) # 转换为布尔值 (除了0和非空list tuple 等之外都是True)
</code></pre>

## 函数别名
函数名其实就是指向一个函数对象的引用，完全可以把函数名赋给一个变量，相当于给这个函数起了一个“别名”
<pre><code class="language-python line-numbers">&gt;&gt;&gt; a = abs # 变量a指向abs函数
&gt;&gt;&gt; a(-1) # 所以也可以通过a调用abs函数
1
</code></pre>

## 定义函数
> 
在Python中，定义一个函数要使用def语句，依次写出函数名、括号、括号中的参数和冒号:，然后，在缩进块中编写函数体，函数的返回值用return语句返回
请注意，函数体内部的语句在执行时，一旦执行到return时，函数就执行完毕，并将结果返回。因此，函数内部通过条件判断和循环可以实现非常复杂的逻辑
如果没有return语句，函数执行完毕后也会返回结果，只是结果为None
return None可以简写为return

## 空函数
<pre><code class="language-python line-numbers">def nop():
    pass

pass语句什么都不做，那有什么用？
实际上pass可以用来作为占位符，比如现在还没想好怎么写函数的代码，就可以先放一个pass，让代码能运行起来。


pass的其他用途
pass还可以用在其他语句里，比如：
if age >= 18:
    pass

缺少了pass，代码运行就会有语法错误。
</code></pre>

## 参数检查
> 
当传入了不恰当的参数时，内置函数abs会检查出参数错误，而我们定义的my_abs没有参数检查，会导致if语句出错，出错信息和abs不一样。所以，这个函数定义不够完善。
让我们修改一下my_abs的定义，对参数类型做检查，只允许整数和浮点数类型的参数。数据类型检查可以用内置函数isinstance()实现：

<pre><code class="language-python line-numbers">def my_abs(x):
    if not isinstance(x, (int, float)):
        raise TypeError(&#x27;bad operand type&#x27;)
    if x &gt;= 0:
        return x
    else:
        return -x
import math语句表示导入math包，并允许后续代码引用math包里的sin、cos等函数

原来返回值是一个tuple！
但是，在语法上，返回一个tuple可以省略括号，而多个变量可以同时接收一个tuple，按位置赋给对应的值
所以，Python的函数返回多值其实就是返回一个tuple，但写起来更方便。
</code></pre>

## 小结
> 
定义函数时，需要确定函数名和参数个数；
如果有必要，可以先对参数的数据类型做检查；
函数体内部可以用return随时返回函数结果；
函数执行完毕也没有return语句时，自动return None。
函数可以同时返回多个值，但其实就是一个tuple。

## 函数参数
### 位置参数
`...`

### 默认参数
<pre><code class="language-python line-numbers">设置默认参数时，有几点要注意：

# 1. 当函数有多个参数时，把变化大的参数放前面，变化小的参数放后面。变化小的参数就可以作为默认参数

# 2. 有多个默认参数时，调用的时候，既可以按顺序提供默认参数.

比如调用enroll('Bob', 'M', 7)，意思是，除了name，gender这两个参数外，最后1个参数应用在参数age上，city参数由于没有提供，仍然使用默认值。

# 3. 也可以不按顺序提供部分默认参数。
# 当不按顺序提供部分默认参数时，需要把参数名写上。

比如调用enroll('Adam', 'M', city='Tianjin')，意思是，city参数用传进去的值，其他默认参数继续使用默认值。
#
# 4. 定义默认参数要牢记一点：默认参数必须指向不变对象！
</code></pre>

### 可变参数 *args
<pre><code class="language-python line-numbers">在Python函数中，还可以定义可变参数。
顾名思义，可变参数就是传入的参数个数是可变的，可以是1个、2个到任意个，还可以是0个
#
# Python允许你在list或tuple前面加一个*号，
# 把list或tuple的元素变成可变参数传进去
#
# 关键字参数同理 加两个*号，可以把dict以key-value方式传进去
</code></pre>

### 命名关键字参数
<pre><code class="language-python line-numbers">如果要限制关键字参数的名字，就可以用命名关键字参数

例如，只接收city和job作为关键字参数。
这种方式定义的函数如下：
def person(name, age, *, city, job):
    print(name, age, city, job)
    
和关键字参数**kw不同，命名关键字参数需要一个特殊分隔符*，
* 后面的参数被视为命名关键字参数。

有可变参数的情况下可以省略*
def person(name, age, *args, city, job)

# 命名关键字参数必须填写，而且需放置在位置参数及默认参数之后(若存在默认参数)
# 为了简化函数调用，一般会给命名关键字函数设置默认参数
</code></pre>

### 关键字参数 **kw
<pre><code class="language-python line-numbers">组装成dict 传进去
加两个*号，可以把dict以key-value方式传进去
</code></pre>

### 参数顺序
<pre><code class="language-python line-numbers">位置参数 -> 默认参数 -> 可变参数 -> 命名关键字参数 -> 关键字参数

当可变参数为 list 或 tuple时 可以放在命名关键字参数后面 但是不能放在关键字参数后面！

对于任意函数，都可以通过类似func(*args, **kw)的形式调用它，无论它的参数是如何定义的
</code></pre>

## 递归函数
> 
在函数内部，可以调用其他函数。
如果一个函数在内部调用自身本身，这个函数就是递归函数。
递归函数的优点是定义简单，逻辑清晰。
理论上，所有的递归函数都可以写成循环的方式，但循环的逻辑不如递归清晰。
使用递归函数需要注意防止栈溢出。
在计算机中，函数调用是通过栈（stack）这种数据结构实现的，每当进入一个函数调用，栈就会加一层栈帧，每当函数返回，栈就会减一层栈帧。
由于栈的大小不是无限的，所以，递归调用的次数过多，会导致栈溢出。
遗憾的是，大多数编程语言没有针对尾递归做优化，Python解释器也没有做优化
所以，即使把上面的fact(n)函数改成尾递归方式，也会导致栈溢出。
