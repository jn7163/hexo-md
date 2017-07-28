---
title: python3 基础
date: 2016-12-09 11:10:00
categories:
- python
tags:
- python
keywords: python, python3, 基础
---
> 
python3 基础
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

> 
`缩进`
建议将vim的tab换成4个空格！
当语句以‘冒号’结尾时，缩进语句视为代码块
不能混用tab和空格！

## 数据类型
<pre><code class="language-python line-numbers">整数 int()
浮点数 float()
# 1.23x10^9 写成 1.23e9
# 整数和浮点数在计算机内部存储的方式是不同的，整数运算永远是精确的（除法难道也是精确的？是的！），而浮点数运算则可能会有四舍五入的误差
字符串 str()
# 字符串是以单引号'或双引号"括起来的任意文本，比如'abc'，"xyz"等等。请注意，''或""本身只是一种表示方式，不是字符串的一部分!
# 如果'本身也是一个字符，那就可以用""括起来
# 如果字符串内部既包含'又包含"怎么办？可以用转义字符\来标识
# 允许用r''表示''内部的字符串默认不转义
# 允许用'''...'''的格式表示多行内容
'''
otokaze
zflyun
zfl9
'''
# r'''...'''同样适用！
布尔值 True False(1,0)
# 一个布尔值只有True、False两种值，要么是True，要么是False
# 非零int 非空list,tuple,dict,set为True
# 布尔值可用 and or not 运算
空值 None
# 一个特殊的值，用None表示。None不能理解为0，因为0是有意义的，而None是一个特殊的空值
列表 list()
# [1,2,3]
# 列表的元素是可变的
# append pop insert等操作
# sort可排序 a.sort()
元组 tuple()
# 和list的区别就是元素不可变，其他基本可以理解为一样
字典 dict()
# key-value 对应
# dict.keys() dict.values() dict.items()
自定义数据类型

type()函数可检测数据类型
eg: type('str')
</code></pre>

## 变量
> 
变量名必须是大小写英文、数字和_的组合，且不能用数字开头
等号=是赋值语句，可以把任意数据类型赋值给变量，同一个变量可以反复赋值，而且可以是不同类型的变量
对于 a='ABC'
Python解释器干了两件事情：
`在内存中创建了一个'ABC'的字符串`
`在内存中创建了一个名为a的变量，并把它指向'ABC'`

## 常量
> 所谓常量就是不能变的变量，比如常用的数学常数π就是一个常量。在Python中，通常用全部大写的变量名表示常量

最后解释一下整数的除法为什么也是精确的。
在Python中，有两种除法，一种除法是 /

<pre><code class="language-python line-numbers">&gt;&gt;&gt; 10 / 3
3.3333333333333335
&gt;&gt;&gt; 9 / 3
3.0
</code></pre>

/ 除法计算结果是浮点数，即使是两个整数恰好整除，结果也是浮点数

还有一种除法是//，称为地板除，两个整数的除法仍然是整数

<pre><code class="language-python line-numbers">&gt;&gt;&gt; 10 // 3
3
</code></pre>

你没有看错，整数的地板除//永远是整数，即使除不尽。
要做精确的除法，使用/就可以。
因为//除法只取结果的整数部分，所以Python还提供一个余数运算，可以得到两个整数相除的余数：

<pre><code class="language-python line-numbers">&gt;&gt;&gt; 10 % 3
1
</code></pre>

无论整数做//除法还是取余数，结果永远是整数，所以，整数运算结果永远是精确的。

## 字符串和编码
<pre><code class="language-python line-numbers"># unicode编码
Python3 字符串是以Unicode编码的，也就是说，Python的字符串支持多语言

对于单个字符的编码，Python提供了ord()函数获取字符的整数表示，chr()函数把编码转换为对应的字符

Python对bytes类型的数据用带b前缀的单引号或双引号表示

要计算str包含多少个字符，可以用len()函数

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
声明python编码为utf-8
-*- coding: utf-8 -*-
或者
# coding=utf-8
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

# b'string'
硬盘、网络中使用的基本单位

# 'string'
内存中使用的单位

# 类型转换
## str -> bytes
a='Hello,world!' # str
a_bytes=a.encode('utf-8') # bytes

# bytes->str
a='Hello,world!' # str
a_bytes=a.encode('utf-8') # bytes
b=a_bytes.decode('utf-8') # str
</code></pre>

## list和tuple
### list列表
> 
Python内置的一种数据类型是列表：list。
list是一种有序的集合，可以随时添加和删除其中的元素

<pre><code class="language-python line-numbers">用索引来访问list中每一个位置的元素，记得索引是从0开始的
如果要取最后一个元素，除了计算索引位置外，还可以用-1做索引，直接获取最后一个元素

list是一个可变的有序表，所以，可以往list中追加元素到末尾：
&gt;&gt;&gt; classmates.append(&#x27;Adam&#x27;)
&gt;&gt;&gt; classmates
[&#x27;Michael&#x27;, &#x27;Bob&#x27;, &#x27;Tracy&#x27;, &#x27;Adam&#x27;]

也可以把元素插入到指定的位置，比如索引号为1的位置：
&gt;&gt;&gt; classmates.insert(1, &#x27;Jack&#x27;)
&gt;&gt;&gt; classmates
[&#x27;Michael&#x27;, &#x27;Jack&#x27;, &#x27;Bob&#x27;, &#x27;Tracy&#x27;, &#x27;Adam&#x27;]

要删除list末尾的元素，用pop()方法：
&gt;&gt;&gt; classmates.pop()
&#x27;Adam&#x27;
&gt;&gt;&gt; classmates
[&#x27;Michael&#x27;, &#x27;Jack&#x27;, &#x27;Bob&#x27;, &#x27;Tracy&#x27;]

要删除指定位置的元素，用pop(i)方法，其中i是索引位置：
&gt;&gt;&gt; classmates.pop(1)
&#x27;Jack&#x27;
&gt;&gt;&gt; classmates
[&#x27;Michael&#x27;, &#x27;Bob&#x27;, &#x27;Tracy&#x27;]

要把某个元素替换成别的元素，可以直接赋值给对应的索引位置：
&gt;&gt;&gt; classmates[1] = &#x27;Sarah&#x27;
&gt;&gt;&gt; classmates
[&#x27;Michael&#x27;, &#x27;Sarah&#x27;, &#x27;Tracy&#x27;]

list元素也可以是另一个list，比如：
&gt;&gt;&gt; s = [&#x27;python&#x27;, &#x27;java&#x27;, [&#x27;asp&#x27;, &#x27;php&#x27;], &#x27;scheme&#x27;]
&gt;&gt;&gt; len(s)
4

要注意s只有4个元素，其中s[2]又是一个list，如果拆开写就更容易理解了：
&gt;&gt;&gt; p = [&#x27;asp&#x27;, &#x27;php&#x27;]
&gt;&gt;&gt; s = [&#x27;python&#x27;, &#x27;java&#x27;, p, &#x27;scheme&#x27;]

要拿到'php'可以写p[1]或者s[2][1]，因此s可以看成是一个二维数组，类似的还有三维、四维……数组，不过很少用到
</code></pre>

### tuple元组
另一种有序列表叫元组：tuple。
tuple和list非常类似，但是tuple一旦初始化就不能修改

<pre><code class="language-python line-numbers">tuple()

但是，要定义一个只有1个元素的tuple，如果你这么定义：
&gt;&gt;&gt; t = (1)
&gt;&gt;&gt; t
1
定义的不是tuple，是1这个数！这是因为括号()既可以表示tuple，又可以表示数学公式中的小括号，这就产生了歧义，因此，Python规定，这种情况下，按小括号进行计算，计算结果自然是1。

所以，只有1个元素的tuple定义时必须加一个逗号,，来消除歧义：
&gt;&gt;&gt; t = (1,)
&gt;&gt;&gt; t
(1,)
</code></pre>

## 字典 dict
<pre><code class="language-python line-numbers">a = {&#x27;otokaze&#x27;:1, &#x27;zflyun&#x27;:2, &#x27;zfl9&#x27;:3}
b=dict(name=&#x27;Otokaze&#x27;,age=17)

# key-value对应 查找很快
#
# 要避免key不存在的错误，有两种办法，一是通过in判断key是否存在：
# 
&gt;&gt;&gt; &#x27;Thomas&#x27; in d
False
# 二是通过dict提供的get方法，如果key不存在，可以返回None，或者自己指定的value：
&gt;&gt;&gt; d.get(&#x27;Thomas&#x27;)
&gt;&gt;&gt; d.get(&#x27;Thomas&#x27;, -1)
-1
#
# 要删除一个key，用pop(key)方法，对应的value也会从dict中删除
#
# 正确使用dict非常重要，需要牢记的第一条就是dict的key必须是‘不可变对象’！

# 这个通过key计算位置的算法称为哈希算法（Hash）
</code></pre>

## 集合 set
<pre><code class="language-python line-numbers">set和dict类似，也是一组key的集合，但不存储value。
由于key不能重复，所以，在set中，没有重复的key

# 要创建一个set，需要提供一个list作为输入集合
# eg：
&gt;&gt;&gt; s = set([1, 2, 3])
&gt;&gt;&gt; s
{1, 2, 3}

# 重复元素在set中自动被过滤：
&gt;&gt;&gt; s = set([1, 1, 2, 2, 3, 3])
&gt;&gt;&gt; s
{1, 2, 3}

# 通过add(key)方法可以添加元素到set中，可以重复添加，但不会有效果：
&gt;&gt;&gt; s.add(4)
&gt;&gt;&gt; s
{1, 2, 3, 4}

# 通过remove(key)方法可以删除元素：
&gt;&gt;&gt; s.remove(4)
&gt;&gt;&gt; s
{1, 2, 3}

# set可以看成数学意义上的无序和无重复元素的集合，因此，两个set可以做数学意义上的交集、并集等操作：
&gt;&gt;&gt; s1 = set([1, 2, 3])
&gt;&gt;&gt; s2 = set([2, 3, 4])
&gt;&gt;&gt; s1 &amp; s2 # 交集
{2, 3}
&gt;&gt;&gt; s1 | s2 # 并集
{1, 2, 3, 4}

# set和dict的唯一区别仅在于没有存储对应的value，但是，set的原理和dict一样，所以，同样不可以放入可变对象，因为无法判断两个可变对象是否相等，也就无法保证set内部“不会有重复元素”。试试把list放入set，看看是否会报错。
</code></pre>

## 可变和不可变对象
<pre><code class="language-python line-numbers"># 对于可变对象，比如list，对list进行操作，list内部的内容是会变化的，比如：
&gt;&gt;&gt; a = [&#x27;c&#x27;, &#x27;b&#x27;, &#x27;a&#x27;]
&gt;&gt;&gt; a.sort()
&gt;&gt;&gt; a
[&#x27;a&#x27;, &#x27;b&#x27;, &#x27;c&#x27;]

# 而对于不可变对象，比如str，对str进行操作呢：
&gt;&gt;&gt; a = &#x27;abc&#x27;
&gt;&gt;&gt; a.replace(&#x27;a&#x27;, &#x27;A&#x27;)
&#x27;Abc&#x27;
&gt;&gt;&gt; a
&#x27;abc&#x27;

虽然字符串有个replace()方法，也确实变出了&#x27;Abc&#x27;，但变量a最后仍是&#x27;abc&#x27;，应该怎么理解呢？

我们先把代码改成下面这样：
&gt;&gt;&gt; a = &#x27;abc&#x27;
&gt;&gt;&gt; b = a.replace(&#x27;a&#x27;, &#x27;A&#x27;)
&gt;&gt;&gt; b
&#x27;Abc&#x27;
&gt;&gt;&gt; a
&#x27;abc&#x27;

# 要始终牢记的是，a是变量，而&#x27;abc&#x27;才是字符串对象！
# 有些时候，我们经常说，对象a的内容是&#x27;abc&#x27;，但其实是指，a本身是一个变量，它指向的对象的内容才是&#x27;abc&#x27;

# 当我们调用a.replace(&#x27;a&#x27;, &#x27;A&#x27;)时，实际上调用方法replace是作用在字符串对象&#x27;abc&#x27;上的，而这个方法虽然名字叫replace，但却没有改变字符串&#x27;abc&#x27;的内容。

# 相反，replace方法创建了一个新字符串&#x27;Abc&#x27;并返回，如果我们用变量b指向该新字符串，就容易理解了，变量a仍指向原有的字符串&#x27;abc&#x27;，但变量b却指向新字符串&#x27;Abc&#x27;了

# 所以，对于不变对象来说，调用对象自身的任意方法，也不会改变该对象自身的内容。相反，这些方法会创建新的对象并返回，这样，就保证了不可变对象本身永远是不可变的。

小结：

# 使用key-value存储结构的dict在Python中非常有用，选择不可变对象作为key很重要，最常用的key是字符串。
# tuple虽然是不变对象，但试试把(1, 2, 3)和(1, [2, 3])放入dict或set中，并解释结果。

a = {(1,2,3):1}
b = {(1,[2,3]):1}
print(a)
print(b)

# a正常显示
# b报错：list不能设置为key！
</code></pre>

## 条件判断
<pre><code class="language-python line-numbers">elif是else if的缩写，完全可以有多个elif，所以if语句的完整形式就是：
if &lt;条件判断1&gt;:
    &lt;执行1&gt;
elif &lt;条件判断2&gt;:
    &lt;执行2&gt;
elif &lt;条件判断3&gt;:
    &lt;执行3&gt;
else:
    &lt;执行4&gt;

if语句执行有个特点，它是从上往下判断，如果在某个判断上是True，把该判断对应的语句执行后，就忽略掉剩下的elif和else
</code></pre>

## 循环
<pre><code class="language-python line-numbers">循环就是把每个元素代入变量x，然后执行缩进块的语句
Python提供一个range()函数，可以生成一个整数序列，再通过list()函数可以转换为list。
比如range(5)生成的序列是从0开始小于5的整数
list(range(5))

## for 循环
for x in ...
for x, y in ...

## while 循环
第二种循环是while循环，只要条件满足，就不断循环，条件不满足时退出循环

## break 结束循环
如果要提前结束循环，可以用break语句

## continue 跳出当前循环，直接开始下轮循环
在循环过程中，也可以通过continue语句，跳过当前的这次循环，直接开始下一次循环

这两个语句通常都必须配合if语句使用
要特别注意，不要滥用break和continue语句
break和continue会造成代码执行逻辑分叉过多，容易出错
大多数循环并不需要用到break和continue语句，上面的两个例子，都可以通过改写循环条件或者修改循环逻辑，去掉break和continue语句
</code></pre>
