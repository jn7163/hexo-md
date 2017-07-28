---
title: python3 高级特性
date: 2016-12-09 11:12:00
categories:
- python
tags:
- python
keywords: python, python3, 高级特性
---
> 
python3 高级特性
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

## 切片
<pre><code class="language-python line-numbers">L[0:3]表示，从索引0开始取，直到索引3为止，但不包括索引3。
即索引0，1，2，正好是3个元素。

如果第一个索引是0，还可以省略
同理 L[-3:]表示，从索引-3开始取，直到取完

L[:10:2] 表示取前十个 并且每隔两个得取
L[::2] 表示每个两个的取，全部取完
L[:] 表示整个列表

不仅仅于list ， tuple， str也可以进行切片操作

利用步长对序列进行倒序取值

在切片运算中，步长为正，表示从左至右，按照索引值与起始位置索引之差可以被步长整除的规律取值；
当步长为负，则表示从右至左，按照按照索引值与起始位置索引之差可以被步长整除的规律取值
</code></pre>

## 迭代 Iteration
<pre><code class="language-python line-numbers">如果给定一个list或tuple，我们可以通过for循环来遍历这个list或tuple，这种遍历我们称为迭代（Iteration）

# 在Python中，迭代是通过for ... in来完成的

如何判断一个对象是可迭代对象呢？方法是通过collections模块的Iterable类型判断

&gt;&gt;&gt; from collections import Iterable
&gt;&gt;&gt; isinstance(&#x27;abc&#x27;, Iterable) # str是否可迭代
True
&gt;&gt;&gt; isinstance([1,2,3], Iterable) # list是否可迭代
True
&gt;&gt;&gt; isinstance(123, Iterable) # 整数是否可迭代
False

Python内置的enumerate函数可以把一个list变成索引-元素对，这样就可以在for循环中同时迭代索引和元素本身

&gt;&gt;&gt; for i, value in enumerate([&#x27;A&#x27;, &#x27;B&#x27;, &#x27;C&#x27;]):
     print(i, value)
0 A
1 B
2 C
</code></pre>

## 列表生成式 List Comprehensions
<pre><code class="language-python line-numbers">循环太繁琐，而列表生成式则可以用一行语句代替循环生成上面的list：

&gt;&gt;&gt; [x * x for x in range(1, 11)]
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

写列表生成式时，把要生成的元素x * x放到前面，后面跟for循环，就可以把list创建出来

for循环后面还可以加上if判断，这样我们就可以筛选出仅偶数的平方：

&gt;&gt;&gt; [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]

还可以使用两层循环，可以生成全排列：

&gt;&gt;&gt; [m + n for m in &#x27;ABC&#x27; for n in &#x27;XYZ&#x27;]
[&#x27;AX&#x27;, &#x27;AY&#x27;, &#x27;AZ&#x27;, &#x27;BX&#x27;, &#x27;BY&#x27;, &#x27;BZ&#x27;, &#x27;CX&#x27;, &#x27;CY&#x27;, &#x27;CZ&#x27;]

三层和三层以上的循环就很少用到了

使用内建的isinstance函数可以判断一个变量是不是字符串：

&gt;&gt;&gt; x = &#x27;abc&#x27;
&gt;&gt;&gt; y = 123
&gt;&gt;&gt; isinstance(x, str)
True
&gt;&gt;&gt; isinstance(y, str)
False
</code></pre>

## 生成器 generator
> 
generator 保存的是算法，所以很省内存空间
要创建一个generator，有很多种方法。

### 第一种方法很简单，只要把一个列表生成式的 [] 改成 () ，就创建了一个generator：
<pre><code class="language-python line-numbers">&gt;&gt;&gt; L = [x * x for x in range(10)]
&gt;&gt;&gt; L
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
&gt;&gt;&gt; g = (x * x for x in range(10))
&gt;&gt;&gt; g
&lt;generator object &lt;genexpr&gt; at 0x1022ef630&gt;
</code></pre>

如果要一个一个打印出来，可以通过next()函数获得generator的下一个返回值：
<pre><code class="language-python line-numbers">&gt;&gt;&gt;&gt; next(g)
0
&gt;&gt;&gt; next(g)
1
&gt;&gt;&gt; next(g)
4
&gt;&gt;&gt; next(g)
9
&gt;&gt;&gt; next(g)
16
&gt;&gt;&gt; next(g)
25
&gt;&gt;&gt; next(g)
36
&gt;&gt;&gt; next(g)
49
&gt;&gt;&gt; next(g)
64
&gt;&gt;&gt; next(g)
81
&gt;&gt;&gt; next(g)
Traceback (most recent call last):
  File &quot;&lt;stdin&gt;&quot;, line 1, in &lt;module&gt;
StopIteration
</code></pre>

当然，上面这种不断调用next(g)实在是太变态了
正确的方法是使用for循环，因为generator也是可迭代对象

### 要把fib函数变成generator，只需要把print(b)改为yield b就可以了：
<pre><code class="language-python line-numbers">def fib(max):
    n, a, b = 0, 0, 1
    while n &lt; max:
        yield (b)
        a, b = b, a + b
        n = n + 1
    return &#x27;done&#x27;

这就是定义generator的另一种方法。
如果一个函数定义中包含yield关键字，那么这个函数就不再是一个普通函数，而是一个generator

这里，最难理解的就是generator和函数的执行流程不一样。

函数是顺序执行，遇到return语句或者最后一行函数语句就返回。

而变成generator的函数，在每次调用next()的时候执行，遇到yield语句返回，再次执行时从上次返回的yield语句处继续执行

举个简单的例子，定义一个generator，依次返回数字1，3，5：

def odd():
    print(&#x27;step 1&#x27;)
    yield 1
    print(&#x27;step 2&#x27;)
    yield(3)
    print(&#x27;step 3&#x27;)
    yield(5)
    
调用该generator时，首先要生成一个generator对象，然后用next()函数不断获得下一个返回值：

&gt;&gt;&gt; o = odd()
&gt;&gt;&gt; next(o)
step 1
1
&gt;&gt;&gt; next(o)
step 2
3
&gt;&gt;&gt; next(o)
step 3
5
&gt;&gt;&gt; next(o)
Traceback (most recent call last):
  File &quot;&lt;stdin&gt;&quot;, line 1, in &lt;module&gt;
StopIteration

可以看到，odd不是普通函数，而是generator
在执行过程中，遇到yield就中断，下次又继续执行。
执行3次yield后，已经没有yield可以执行了，所以，第4次调用next(o)就报错。

回到fib的例子，我们在循环过程中不断调用yield，就会不断中断。
当然要给循环设置一个条件来退出循环，不然就会产生一个无限数列出来。

同样的，把函数改成generator后，我们基本上从来不会用next()来获取下一个返回值，而是直接使用for循环来迭代：

但是用for循环调用generator时，发现拿不到generator的return语句的返回值。如果想要拿到返回值，必须捕获StopIteration错误，返回值包含在StopIteration的value中：

&gt;&gt;&gt; g = fib(6)
&gt;&gt;&gt; while True:
     try:
         x = next(g)
         print(&#x27;g:&#x27;, x)
     except StopIteration as e:
         print(&#x27;Generator return value:&#x27;, e.value)
         break

g: 1
g: 1
g: 2
g: 3
g: 5
g: 8
Generator return value: done
</code></pre>

## 迭代器
<pre><code class="language-python line-numbers">我们已经知道，可以直接作用于for循环的数据类型有以下几种：

一类是 集合数据类型，如list、tuple、dict、set、str等；

一类是 generator，包括生成器和带yield的generator function。

这些可以直接作用于for循环的对象统称为可迭代对象：Iterable。

可以使用isinstance()判断一个对象是否是Iterable对象：

&gt;&gt;&gt; from collections import Iterable
&gt;&gt;&gt; isinstance([], Iterable)
True
&gt;&gt;&gt; isinstance({}, Iterable)
True
&gt;&gt;&gt; isinstance(&#x27;abc&#x27;, Iterable)
True
&gt;&gt;&gt; isinstance((x for x in range(10)), Iterable)
True
&gt;&gt;&gt; isinstance(100, Iterable)
False

而生成器不但可以作用于for循环，还可以被next()函数不断调用并返回下一个值，直到最后抛出StopIteration错误表示无法继续返回下一个值了。

可以被next()函数调用并不断返回下一个值的对象称为迭代器：Iterator。

可以使用isinstance()判断一个对象是否是Iterator对象：

&gt;&gt;&gt; from collections import Iterator
&gt;&gt;&gt; isinstance((x for x in range(10)), Iterator)
True
&gt;&gt;&gt; isinstance([], Iterator)
False
&gt;&gt;&gt; isinstance({}, Iterator)
False
&gt;&gt;&gt; isinstance(&#x27;abc&#x27;, Iterator)
False

生成器都是Iterator对象，但list、dict、str虽然是Iterable，却不是Iterator。

把list、dict、str等Iterable变成Iterator可以使用iter()函数：

&gt;&gt;&gt; isinstance(iter([]), Iterator)
True
&gt;&gt;&gt; isinstance(iter(&#x27;abc&#x27;), Iterator)
True


小结

凡是可作用于for循环的对象都是Iterable类型；

凡是可作用于next()函数的对象都是Iterator类型，它们表示一个惰性计算的序列；

集合数据类型如list、dict、str等是Iterable但不是Iterator，不过可以通过iter()函数获得一个Iterator对象。

Python的for循环本质上就是通过不断调用next()函数实现的
</code></pre>
