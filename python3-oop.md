---
title: python3 面向对象编程 OOP
date: 2016-12-09 11:30:00
categories:
- python
tags:
- python
keywords: python, python3, 面向对象编程OOP
---
> 
python3 面向对象编程OOP
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

> 
OOP把对象作为程序的基本单元，一个对象包含了数据和操作数据的函数
在Python中，所有数据类型都可以视为对象，当然也可以自定义对象。
自定义的对象数据类型就是面向对象中的类（Class）的概念


<pre><code class="language-python line-numbers">如：我们需要打印学生的成绩

如果采用面向对象的程序设计思想，我们首选思考的不是程序的执行流程

而是Student这种数据类型应该被视为一个对象
这个对象拥有name和score这两个属性（Property）。
如果要打印一个学生的成绩，首先必须创建出这个学生对应的对象，
然后，给对象发一个print_score消息，
让对象自己把自己的数据打印出来

class Student(object):
    def __init__(self,name,score):
        self.name=name
        self.score=score
    def print_score(self):
        print('%s score: %s' % (self.name,self.socre))

a=Student('Otokaze',79)
a.print_score()
</code></pre>

## 属性
<pre><code class="language-python line-numbers">实例的属性 (实际就是变量)
</code></pre>

## 方法
<pre><code class="language-python line-numbers">给对象发消息实际上就是调用对象对应的关联函数
我们称之为对象的方法(Method)
</code></pre>

## 类（Class）和实例（Instance）
<pre><code class="language-python line-numbers">Class是一种抽象概念
比如我们定义的Class——Student，是指学生这个概念

而实例（Instance）则是一个个具体的Student
比如，Bart Simpson和Lisa Simpson是两个具体的Student。

所以，面向对象的设计思想是抽象出Class，根据Class创建Instance
</code></pre>

## 类和实例
<pre><code class="language-python line-numbers">class后面紧接着是类名，即Student
类名通常是大写开头的单词，紧接着是(object)
表示该类是从哪个类继承下来的

通常，如果没有合适的继承类，就使用object类，这是所有类最终都会继承的类
</code></pre>

## 创建类
<pre><code class="language-python line-numbers">class Student(object):
    pass
</code></pre>

## 创建实例
<pre><code class="language-python line-numbers">zfl=Student()
abc=Student()
</code></pre>

## 给实例绑定属性
<pre><code class="language-python line-numbers">zfl.name='Otokaze'
zfl.score=99
zfl.name
zfl.score
</code></pre>

## 限制属性
<pre><code class="language-python line-numbers">class Student(object):
    def __init__(self,name,score):
        self.name=name
        self.score=score
 
注意类的方法的第一个参数永远是self 表示创建的实例本身。

在调用方法时不需要传入改参数！

除了第一个self外 其他函数参数和之前讲的普通函数一样 共有5种函数参数类型

# 限制变量的访问

只需在内部变量前面加上 "__" 即可
python解释器把双下划线开头的变量解释为 _Student__score
</code></pre>

## 判断实例的类
<pre><code class="language-python line-numbers">type('str')==type('abc')
True

isinstance([1, 2, 3], (list, tuple))
True

isinstance((1, 2, 3), (list, tuple))
True

isinstance(1,int)
True
</code></pre>

## dir()
获取对象的所有属性和方法
<pre><code class="language-python line-numbers">如果要获得一个对象的所有属性和方法，可以使用dir()函数
它返回一个包含字符串的list
</code></pre>

## getattr, setattr, hasattr
仅仅把属性和方法列出来是不够的
配合 getattr()、setattr() 以及 hasattr()
我们可以直接操作一个对象的状态：
<pre><code class="language-python line-numbers">&gt;&gt;&gt; class MyObject(object):
...     def __init__(self):
...         self.x = 9
...     def power(self):
...         return self.x * self.x
...
&gt;&gt;&gt; obj = MyObject()

紧接着，可以测试该对象的属性：

&gt;&gt;&gt; hasattr(obj, &#x27;x&#x27;) # 有属性&#x27;x&#x27;吗？
True
&gt;&gt;&gt; obj.x
9
&gt;&gt;&gt; hasattr(obj, &#x27;y&#x27;) # 有属性&#x27;y&#x27;吗？
False
&gt;&gt;&gt; setattr(obj, &#x27;y&#x27;, 19) # 设置一个属性&#x27;y&#x27;
&gt;&gt;&gt; hasattr(obj, &#x27;y&#x27;) # 有属性&#x27;y&#x27;吗？
True
&gt;&gt;&gt; getattr(obj, &#x27;y&#x27;) # 获取属性&#x27;y&#x27;
19
&gt;&gt;&gt; obj.y # 获取属性&#x27;y&#x27;
19

可以传入一个default参数，如果属性不存在，就返回默认值：
&gt;&gt;&gt; getattr(obj, &#x27;z&#x27;, 404)  
404
# 获取属性&#x27;z&#x27;，如果不存在，返回默认值404

del # 删除变量
a=&#x27;213&#x27;
del a
</code></pre>

## `__slots__` 限制属性
<pre><code class="language-python line-numbers">class Student(object):
    __slots__=('__name','__score')
    def __init__(self,name,score): #list或tuple
        self.__name=name
        self.__score=score
    @property
    def name(self):
        return self.__name
    @name.setter
    def name(self,name):
        self.__name=name
    @property
    def score(self):
        return self.__score
    @score.setter
    def score(self,score):
        self.__score=score
        
只允许设置__name __score属性

使用__slots__要注意，__slots__定义的属性仅对当前类实例起作用，对继承的子类是不起作用的

当子类也使用__slots__时 子类允许设置的属性为父类与子类的总和
</code></pre>

## 使用 @property

有没有既能检查参数，又可以用类似属性这样简单的方式来访问类的变量呢？
对于追求完美的Python程序员来说，这是必须要做到的！

还记得装饰器（decorator）可以给函数动态加上功能吗？
对于类的方法，装饰器一样起作用。
Python内置的@property装饰器就是负责把一个方法变成属性调用的

<pre><code class="language-python line-numbers">class Student(object):
    @property
    def score(self):
        return self._score
    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value

@property的实现比较复杂，我们先考察如何使用。

把一个getter方法变成属性，只需要加上@property就可以了

此时，@property本身又创建了另一个装饰器@score.setter
负责把一个setter方法变成属性赋值，于是，我们就拥有一个可控的属性操作

还可以定义只读属性，只定义getter方法，不定义setter方法就是一个只读属性
</code></pre>

## 多重继承
**MixIn**
> 
在设计类的继承关系时，通常，主线都是单一继承下来的
例如，Ostrich继承自Bird。
但是，如果需要“混入”额外的功能，通过多重继承就可以实现
比如，让Ostrich除了继承自Bird外，再同时继承Runnable。
这种设计通常称之为MixIn
为了更好地看出继承关系，我们把Runnable和Flyable改为RunnableMixIn和FlyableMixIn。
类似的，你还可以定义出肉食动物CarnivorousMixIn和植食动物HerbivoresMixIn，让某个动物同时拥有好几个MixIn

## 定制类 (特殊变量、函数)
### `__str__`
<pre><code class="language-python line-numbers">我们先定义一个Student类，打印一个实例：

&gt;&gt;&gt; class Student(object):
...     def __init__(self, name):
...         self.name = name
...
&gt;&gt;&gt; print(Student(&#x27;Michael&#x27;))
&lt;__main__.Student object at 0x109afb190&gt;

打印出一堆&lt;__main__.Student object at 0x109afb190&gt;，不好看。

怎么才能打印得好看呢？只需要定义好__str__()方法，返回一个好看的字符串就可以了：

&gt;&gt;&gt; class Student(object):
...     def __init__(self, name):
...         self.name = name
...     def __str__(self):
...         return &#x27;Student object (name: %s)&#x27; % self.name
...
&gt;&gt;&gt; print(Student(&#x27;Michael&#x27;))
Student object (name: Michael)

但是细心的朋友会发现直接敲变量不用print，打印出来的实例还是不好看：

&gt;&gt;&gt; s = Student(&#x27;Michael&#x27;)
&gt;&gt;&gt; s
&lt;__main__.Student object at 0x109afb310&gt;

这是因为直接显示变量调用的不是__str__()，而是__repr__()

两者的区别是__str__()返回用户看到的字符串
而__repr__()返回程序开发者看到的字符串
也就是说，__repr__()是为调试服务的。

解决办法是再定义一个__repr__()。
但是通常__str__()和__repr__()代码都是一样的，所以，有个偷懒的写法：

class Student(object):
    def __init__(self, name):
        self.name = name
    def __str__(self):
        return &#x27;Student object (name=%s)&#x27; % self.name
    __repr__ = __str__
</code></pre>

### `__iter__`
<pre><code class="language-python line-numbers">如果一个类想被用于for ... in循环，类似list或tuple那样

就必须实现一个__iter__()方法，该方法返回一个迭代对象，
然后，Python的for循环就会不断调用该迭代对象的__next__()方法拿到循环的下一个值，
直到遇到StopIteration错误时退出循环。

我们以斐波那契数列为例，写一个Fib类，可以作用于for循环：

class Fib(object):
    def __init__(self):
        self.a, self.b = 0, 1 # 初始化两个计数器a，b

    def __iter__(self):
        return self # 实例本身就是迭代对象，故返回自己

    def __next__(self):
        self.a, self.b = self.b, self.a + self.b # 计算下一个值
        if self.a &gt; 100000: # 退出循环的条件
            raise StopIteration();
        return self.a # 返回下一个值
        
现在，试试把Fib实例作用于for循环：

&gt;&gt;&gt; for n in Fib():
...     print(n)
...
1
1
2
3
5  
...
</code></pre>

### `__getitem__`
<pre><code class="language-python line-numbers">Fib实例虽然能作用于for循环，看起来和list有点像
但是，把它当成list来使用还是不行，比如，取第5个元素：

&gt;&gt;&gt; Fib()[5]
Traceback (most recent call last):
  File &quot;&lt;stdin&gt;&quot;, line 1, in &lt;module&gt;
TypeError: &#x27;Fib&#x27; object does not support indexing

要表现得像list那样按照下标取出元素，需要实现__getitem__()方法：

class Fib(object):
    def __getitem__(self, n):
        a, b = 1, 1
        for x in range(n):
            a, b = b, a + b
        return a

现在，就可以按下标访问数列的任意一项了：

&gt;&gt;&gt; f = Fib()
&gt;&gt;&gt; f[0]
1
&gt;&gt;&gt; f[1]
1
&gt;&gt;&gt; f[2]
2
&gt;&gt;&gt; f[3]
3
&gt;&gt;&gt; f[10]
89
&gt;&gt;&gt; f[100]
573147844013817084101

但是list有个神奇的切片方法：

&gt;&gt;&gt; list(range(100))[5:10]
[5, 6, 7, 8, 9]

对于Fib却报错。原因是__getitem__()传入的参数可能是一个int，也可能是一个切片对象slice，所以要做判断：

class Fib(object):
    def __getitem__(self, n):
        if isinstance(n, int): # n是索引
            a, b = 1, 1
            for x in range(n):
                a, b = b, a + b
            return a
        if isinstance(n, slice): # n是切片
            start = n.start
            stop = n.stop
            if start is None:
                start = 0
            a, b = 1, 1
            L = []
            for x in range(stop):
                if x &gt;= start:
                    L.append(a)
                a, b = b, a + b
            return L
</code></pre>

### `__getattr__`
<pre><code class="language-python line-numbers">Python还有另一个机制，那就是写一个__getattr__()方法
动态返回一个属性。修改如下：

class Student(object):
    def __init__(self):
        self.name = &#x27;Michael&#x27;
    def __getattr__(self, attr):
        if attr==&#x27;score&#x27;:
            return 99

当调用不存在的属性时
比如score，Python解释器会试图调用__getattr__(self, &#x27;score&#x27;)来尝试获得属性
这样，我们就有机会返回score的值：

&gt;&gt;&gt; s = Student()
&gt;&gt;&gt; s.name
&#x27;Michael&#x27;
&gt;&gt;&gt; s.score
99

注意，只有在没有找到属性的情况下，才调用__getattr__，已有的属性，比如name，不会在__getattr__中查找

此外，注意到任意调用如s.abc都会返回None，这是因为我们定义的__getattr__默认返回就是None。

要让class只响应特定的几个属性，我们就要按照约定，抛出AttributeError的错误：

class Student(object):
    def __getattr__(self, attr):
        if attr==&#x27;age&#x27;:
            return lambda: 25
        raise AttributeError(&#x27;\&#x27;Student\&#x27; object has no attribute \&#x27;%s\&#x27;&#x27; % attr)
        
这种完全动态调用的特性有什么实际作用呢？作用就是，可以针对完全动态的情况作调用。

举个例子：

现在很多网站都搞REST API，比如新浪微博、豆瓣啥的，调用API的URL类似：

http://api.server/user/friends
http://api.server/user/timeline/list

如果要写SDK，给每个URL对应的API都写一个方法，那得累死，而且，API一旦改动，SDK也要改。

利用完全动态的__getattr__，我们可以写出一个链式调用：

class Chain(object):
    def __init__(self, path=&#x27;&#x27;):
        self._path = path
    def __getattr__(self, path):
        return Chain(&#x27;%s/%s&#x27; % (self._path, path))
    def __str__(self):
        return self._path
    __repr__ = __str__
    
试试：

&gt;&gt;&gt; Chain().status.user.timeline.list
&#x27;/status/user/timeline/list&#x27;

这样，无论API怎么变，SDK都可以根据URL实现完全动态的调用，而且，不随API的增加而改变！
</code></pre>

### `__call__`
<pre><code class="language-python line-numbers">一个对象实例可以有自己的属性和方法，当我们调用实例方法时，我们用instance.method()来调用。
能不能直接在实例本身上调用呢？在Python中，答案是肯定的。

任何类，只需要定义一个__call__()方法，就可以直接对实例进行调用。请看示例：

class Student(object):
    def __init__(self, name):
        self.name = name
    def __call__(self):
        print(&#x27;My name is %s.&#x27; % self.name)

调用方式如下：

&gt;&gt;&gt; s = Student(&#x27;Michael&#x27;)
&gt;&gt;&gt; s() # self参数不要传入
My name is Michael.

我们需要判断一个对象是否能被调用，能被调用的对象就是一个Callable对象，比如函数和我们上面定义的带有__call__()的类实例：

&gt;&gt;&gt; callable(Student())
True
&gt;&gt;&gt; callable(max)
True
&gt;&gt;&gt; callable([1, 2, 3])
False
&gt;&gt;&gt; callable(None)
False
&gt;&gt;&gt; callable(&#x27;str&#x27;)
False

通过callable()函数，我们就可以判断一个对象是否是“可调用”对象。
</code></pre>

## 枚举类
更好的方法是为这样的枚举类型定义一个class类型
然后，每个常量都是class的一个唯一实例
Python提供了Enum类来实现这个功能：
<pre><code class="language-python line-numbers">from enum import Enum

Month = Enum(&#x27;Month&#x27;, (&#x27;Jan&#x27;, &#x27;Feb&#x27;, &#x27;Mar&#x27;, &#x27;Apr&#x27;, &#x27;May&#x27;, &#x27;Jun&#x27;, &#x27;Jul&#x27;, &#x27;Aug&#x27;, &#x27;Sep&#x27;, &#x27;Oct&#x27;, &#x27;Nov&#x27;, &#x27;Dec&#x27;))

这样我们就获得了Month类型的枚举类，可以直接使用Month.Jan来引用一个常量，或者枚举它的所有成员

&gt;&gt;&gt; for x,y in test.__members__.items():
...     print(&#x27;name: %s --&gt; members: %s --&gt; value: %s&#x27; % (x,y,y.value))
... 
name: a --&gt; members: test.a --&gt; value: 1
name: b --&gt; members: test.b --&gt; value: 2
name: c --&gt; members: test.c --&gt; value: 3

value属性则是自动赋给成员的int常量，默认从1开始计数

如果需要更精确地控制枚举类型，可以从Enum派生出自定义类：

from enum import Enum, unique

@unique
class Weekday(Enum):
    Sun = 0 # Sun的value被设定为0
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6
    
@unique装饰器可以帮助我们检查保证没有重复值

访问这些枚举类型可以有若干种方法：

&gt;&gt;&gt; day1 = Weekday.Mon
&gt;&gt;&gt; print(day1)
Weekday.Mon
&gt;&gt;&gt; print(Weekday.Tue)
Weekday.Tue
&gt;&gt;&gt; print(Weekday[&#x27;Tue&#x27;])
Weekday.Tue
&gt;&gt;&gt; print(Weekday.Tue.value)
2
&gt;&gt;&gt; print(day1 == Weekday.Mon)
True
&gt;&gt;&gt; print(day1 == Weekday.Tue)
False
&gt;&gt;&gt; print(Weekday(1))
Weekday.Mon
&gt;&gt;&gt; print(day1 == Weekday(1))
True
&gt;&gt;&gt; Weekday(7)
Traceback (most recent call last):
  ...
ValueError: 7 is not a valid Weekday
&gt;&gt;&gt; for name, member in Weekday.__members__.items():
...     print(name, &#x27;=&gt;&#x27;, member)
...
Sun =&gt; Weekday.Sun
Mon =&gt; Weekday.Mon
Tue =&gt; Weekday.Tue
Wed =&gt; Weekday.Wed
Thu =&gt; Weekday.Thu
Fri =&gt; Weekday.Fri
Sat =&gt; Weekday.Sat
</code></pre>

## super()
`super().__init__()`子类调用父类的构造函数
<pre><code class="language-python line-numbers"><script type="text/plain">#!/usr/env python3
#coding: utf-8

class Base(object):
    def __init__(self):
        print("Base class Create!")

class Son(Base):
    def __init__(self):
        super().__init__()
        print("Son class Create!")

Son()
</script></code></pre>

<pre><code class="language-python"><script type="text/plain"># root @ localhost in ~ [21:28:28]
$ chmod +x a.py

# root @ localhost in ~ [21:28:33]
$ ./a.py
Base class Create!
Son class Create!
</script></code></pre>
