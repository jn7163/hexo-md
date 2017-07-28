---
title: python3 常用内建模块
date: 2016-12-09 12:00:00
categories:
- python
tags:
- python
keywords: python, python3, 常用内建模块
---
> 
python3 常用内建模块
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

## datetime
<pre><code class="language-python line-numbers">datetime是Python处理日期和时间的标准库。

## 获取当前日期和时间 now()

>>> from datetime import datetime
>>> now = datetime.now() # 获取当前datetime
>>> print(now)
2015-05-18 16:28:07.198690
>>> print(type(now))
<class 'datetime.datetime'>

## 获取指定日期和时间 datetime()

>>> from datetime import datetime
>>> dt = datetime(2015, 4, 19, 12, 20) # 用指定日期时间创建datetime
>>> print(dt)
2015-04-19 12:20:00

## datetime转换为timestamp

在计算机中，时间实际上是用数字表示的。我们把1970年1月1日 00:00:00 UTC+00:00时区的时刻称为epoch time，记为0（1970年以前的时间timestamp为负数）
当前时间就是相对于epoch time的秒数，称为timestamp

>>> from datetime import datetime
>>> dt = datetime(2015, 4, 19, 12, 20) # 用指定日期时间创建datetime
>>> dt.timestamp() # 把datetime转换为timestamp
1429417200.0
注意Python的timestamp是一个浮点数。如果有小数位，小数位表示毫秒数

## timestamp转换为datetime

要把timestamp转换为datetime，使用datetime提供的fromtimestamp()方法：

>>> from datetime import datetime
>>> t = 1429417200.0
>>> print(datetime.fromtimestamp(t))
2015-04-19 12:20:00

### timestamp也可以直接被转换到UTC标准时区的时间：

>>> from datetime import datetime
>>> t = 1429417200.0
>>> print(datetime.fromtimestamp(t)) # 本地时间
2015-04-19 12:20:00
>>> print(datetime.utcfromtimestamp(t)) # UTC时间
2015-04-19 04:20:00

## str转换为datetime

很多时候，用户输入的日期和时间是字符串，要处理日期和时间，首先必须把str转换为datetime。转换方法是通过datetime.strptime()实现，需要一个日期和时间的格式化字符串：

>>> from datetime import datetime
>>> cday = datetime.strptime('2015-6-1 18:19:59', '%Y-%m-%d %H:%M:%S')
>>> print(cday)
2015-06-01 18:19:59

字符串'%Y-%m-%d %H:%M:%S'规定了日期和时间部分的格式。详细的说明请参考Python文档。
注意转换后的datetime是没有时区信息的

## datetime转换为str

如果已经有了datetime对象，要把它格式化为字符串显示给用户，就需要转换为str，转换方法是通过strftime()实现的，同样需要一个日期和时间的格式化字符串：

>>> from datetime import datetime
>>> now = datetime.now()
>>> print(now.strftime('%a, %b %d %H:%M'))
Mon, May 05 16:28

## datetime加减

对日期和时间进行加减实际上就是把datetime往后或往前计算，得到新的datetime。加减可以直接用+和-运算符，不过需要导入timedelta这个类：

>>> from datetime import datetime, timedelta
>>> now = datetime.now()
>>> now
datetime.datetime(2015, 5, 18, 16, 57, 3, 540997)
>>> now + timedelta(hours=10)
datetime.datetime(2015, 5, 19, 2, 57, 3, 540997)
>>> now - timedelta(days=1)
datetime.datetime(2015, 5, 17, 16, 57, 3, 540997)
>>> now + timedelta(days=2, hours=12)
datetime.datetime(2015, 5, 21, 4, 57, 3, 540997)
</code></pre>

## collections

collections是Python内建的一个集合模块，提供了许多有用的集合类。

### namedtuple
<pre><code class="language-python line-numbers">我们知道tuple可以表示不变集合，例如，一个点的二维坐标就可以表示成：

>>> p = (1, 2)
但是，看到(1, 2)，很难看出这个tuple是用来表示一个坐标的。

定义一个class又小题大做了，这时，namedtuple就派上了用场：

>>> from collections import namedtuple
>>> Point = namedtuple('Point', ['x', 'y'])
>>> p = Point(1, 2)
>>> p.x
1
>>> p.y
2

namedtuple是一个函数，它用来创建一个自定义的tuple对象，并且规定了tuple元素的个数，并可以用属性而不是索引来引用tuple的某个元素。

这样一来，我们用namedtuple可以很方便地定义一种数据类型，它具备tuple的不变性，又可以根据属性来引用，使用十分方便
</code></pre>

### deque
<pre><code class="language-python line-numbers">使用list存储数据时，按索引访问元素很快，但是插入和删除元素就很慢了，因为list是线性存储，数据量大的时候，插入和删除效率很低。

deque是为了高效实现插入和删除操作的双向列表，适合用于队列和栈：

>>> from collections import deque
>>> q = deque(['a', 'b', 'c'])
>>> q.append('x')
>>> q.appendleft('y')
>>> q
deque(['y', 'a', 'b', 'c', 'x'])

deque除了实现list的append()和pop()外，还支持appendleft()和popleft()，这样就可以非常高效地往头部添加或删除元素
</code></pre>

### defaultdict
<pre><code class="language-python line-numbers">使用dict时，如果引用的Key不存在，就会抛出KeyError。如果希望key不存在时，返回一个默认值，就可以用defaultdict：

>>> from collections import defaultdict
>>> dd = defaultdict(lambda: 'N/A')
>>> dd['key1'] = 'abc'
>>> dd['key1'] # key1存在
'abc'
>>> dd['key2'] # key2不存在，返回默认值
'N/A'

注意默认值是调用函数返回的，而函数在创建defaultdict对象时传入。

除了在Key不存在时返回默认值，defaultdict的其他行为跟dict是完全一样的
</code></pre>

### OrderedDict
<pre><code class="language-python line-numbers">使用dict时，Key是无序的。在对dict做迭代时，我们无法确定Key的顺序。

如果要保持Key的顺序，可以用OrderedDict：

>>> from collections import OrderedDict
>>> d = dict([('a', 1), ('b', 2), ('c', 3)])
>>> d # dict的Key是无序的
{'a': 1, 'c': 3, 'b': 2}
>>> od = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
>>> od # OrderedDict的Key是有序的
OrderedDict([('a', 1), ('b', 2), ('c', 3)])

注意，OrderedDict的Key会按照插入的顺序排列，不是Key本身排序
</code></pre>

### Counter
<pre><code class="language-python line-numbers">Counter是一个简单的计数器，例如，统计字符出现的个数：

>>> from collections import Counter
>>> c = Counter()
>>> for ch in 'programming':
...     c[ch] = c[ch] + 1
...
>>> c
Counter({'g': 2, 'm': 2, 'r': 2, 'a': 1, 'i': 1, 'o': 1, 'n': 1, 'p': 1})

Counter实际上也是dict的一个子类，上面的结果可以看出，字符'g'、'm'、'r'各出现了两次，其他字符各出现了一次
</code></pre>

### hashlib
<pre><code class="language-python line-numbers">我们以常见的摘要算法MD5为例，计算出一个字符串的MD5值：

import hashlib
md5 = hashlib.md5()
md5.update('how to use md5 in python hashlib?'.encode('utf-8'))
print(md5.hexdigest())

计算结果如下：
d26a53750bc40b38b65a520292f69306

如果数据量很大，可以分块多次调用update()，最后计算的结果是一样的：

import hashlib
md5 = hashlib.md5()
md5.update('how to use md5 in '.encode('utf-8'))
md5.update('python hashlib?'.encode('utf-8'))
print(md5.hexdigest())

另一种常见的摘要算法是SHA1，调用SHA1和调用MD5完全类似：

import hashlib
sha1 = hashlib.sha1()
sha1.update('how to use sha1 in '.encode('utf-8'))
sha1.update('python hashlib?'.encode('utf-8'))
print(sha1.hexdigest())

## 摘要算法常用于验证用户密码

## 可以加盐 就是在用户密码中混入自定义字符串 如:用户名或其他。。
</code></pre>

### itertools
<pre><code class="language-python line-numbers">Python的内建模块itertools提供了非常有用的用于操作迭代对象的函数。

首先，我们看看itertools提供的几个“无限”迭代器：

>>> import itertools
>>> natuals = itertools.count(1)
>>> for n in natuals:
...     print(n)
...
1
2
3
...

因为count()会创建一个无限的迭代器，所以上述代码会打印出自然数序列，根本停不下来，只能按Ctrl+C退出。

cycle()会把传入的一个序列无限重复下去：

>>> import itertools
>>> cs = itertools.cycle('ABC') # 注意字符串也是序列的一种
>>> for c in cs:
...     print(c)
...
'A'
'B'
'C'
'A'
'B'
'C'
...

同样停不下来。

repeat()负责把一个元素无限重复下去，不过如果提供第二个参数就可以限定重复次数：

>>> ns = itertools.repeat('A', 3)
>>> for n in ns:
...     print(n)
...
A
A
A

无限序列只有在for迭代时才会无限地迭代下去，如果只是创建了一个迭代对象，它不会事先把无限个元素生成出来，事实上也不可能在内存中创建无限多个元素。

无限序列虽然可以无限迭代下去，但是通常我们会通过takewhile()等函数根据条件判断来截取出一个有限的序列：

>>> natuals = itertools.count(1)
>>> ns = itertools.takewhile(lambda x: x <= 10, natuals)
>>> list(ns)
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

itertools提供的几个迭代器操作函数更加有用：

chain()

chain()可以把一组迭代对象串联起来，形成一个更大的迭代器：

>>> for c in itertools.chain('ABC', 'XYZ'):
...     print(c)

# 迭代效果：'A' 'B' 'C' 'X' 'Y' 'Z'

groupby()

groupby()把迭代器中相邻的重复元素挑出来放在一起：

>>> for key, group in itertools.groupby('AAABBBCCAAA'):
...     print(key, list(group))
...
A ['A', 'A', 'A']
B ['B', 'B', 'B']
C ['C', 'C']
A ['A', 'A', 'A']

实际上挑选规则是通过函数完成的，只要作用于函数的两个元素返回的值相等，这两个元素就被认为是在一组的，而函数返回值作为组的key。如果我们要忽略大小写分组，就可以让元素'A'和'a'都返回相同的key：

>>> for key, group in itertools.groupby('AaaBBbcCAAa', lambda c: c.upper()):
...     print(key, list(group))
...
A ['A', 'a', 'a']
B ['B', 'B', 'b']
C ['c', 'C']
A ['A', 'A', 'a']
</code></pre>

## @contextmanager
<pre><code class="language-python line-numbers">Python的标准库contextlib提供了更简单的写法，上面的代码可以改写如下：

from contextlib import contextmanager

class Query(object):

    def __init__(self, name):
        self.name = name

    def query(self):
        print(&#x27;Query info about %s...&#x27; % self.name)

@contextmanager
def create_query(name):
    print(&#x27;Begin&#x27;)
    q = Query(name) # 创建实例
    yield q
    print(&#x27;End&#x27;)
    
@contextmanager这个decorator接受一个generator，用yield语句把with ... as var把变量输出出去，然后，with语句就可以正常地工作了：

with create_query(&#x27;Bob&#x27;) as q:
    q.query()
    
很多时候，我们希望在某段代码执行前后自动执行特定代码，也可以用@contextmanager实现。例如：
@contextmanager
def tag(name):
    print(&quot;&lt;%s&gt;&quot; % name)
    yield
    print(&quot;&lt;/%s&gt;&quot; % name)

with tag(&quot;h1&quot;):
    print(&quot;hello&quot;)
    print(&quot;world&quot;)
上述代码执行结果为：

&lt;h1&gt;
hello
world
&lt;/h1&gt;

代码的执行顺序是：

with语句首先执行yield之前的语句，因此打印出&lt;h1&gt;；
yield调用会执行with语句内部的所有语句，因此打印出hello和world；
最后执行yield之后的语句，打印出&lt;/h1&gt;。
因此，@contextmanager让我们通过编写generator来简化上下文管理

# contextlib

在Python中，读写文件这样的资源要特别注意，必须在使用完毕后正确关闭它们。正确关闭文件资源的一个方法是使用try...finally：

try:
    f = open(&#x27;/path/to/file&#x27;, &#x27;r&#x27;)
    f.read()
finally:
    if f:
        f.close()
写try...finally非常繁琐。Python的with语句允许我们非常方便地使用资源，而不必担心资源没有关闭，所以上面的代码可以简化为：

with open(&#x27;/path/to/file&#x27;, &#x27;r&#x27;) as f:
    f.read()
并不是只有open()函数返回的fp对象才能使用with语句。实际上，任何对象，只要正确实现了上下文管理，就可以用于with语句。

实现上下文管理是通过__enter__和__exit__这两个方法实现的。例如，下面的class实现了这两个方法：

class Query(object):

    def __init__(self, name):
        self.name = name

    def __enter__(self):
        print(&#x27;Begin&#x27;)
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type:
            print(&#x27;Error&#x27;)
        else:
            print(&#x27;End&#x27;)

    def query(self):
        print(&#x27;Query info about %s...&#x27; % self.name)
这样我们就可以把自己写的资源对象用于with语句：

with Query(&#x27;Bob&#x27;) as q:
    q.query()
@contextmanager

编写__enter__和__exit__仍然很繁琐，因此Python的标准库contextlib提供了更简单的写法，上面的代码可以改写如下：

from contextlib import contextmanager

class Query(object):

    def __init__(self, name):
        self.name = name

    def query(self):
        print(&#x27;Query info about %s...&#x27; % self.name)

@contextmanager
def create_query(name):
    print(&#x27;Begin&#x27;)
    q = Query(name)
    yield q
    print(&#x27;End&#x27;)
@contextmanager这个decorator接受一个generator，用yield语句把with ... as var把变量输出出去，然后，with语句就可以正常地工作了：

with create_query(&#x27;Bob&#x27;) as q:
    q.query()
很多时候，我们希望在某段代码执行前后自动执行特定代码，也可以用@contextmanager实现。例如：

@contextmanager
def tag(name):
    print(&quot;&lt;%s&gt;&quot; % name)
    yield
    print(&quot;&lt;/%s&gt;&quot; % name)

with tag(&quot;h1&quot;):
    print(&quot;hello&quot;)
    print(&quot;world&quot;)
上述代码执行结果为：

&lt;h1&gt;
hello
world
&lt;/h1&gt;
代码的执行顺序是：

with语句首先执行yield之前的语句，因此打印出&lt;h1&gt;；
yield调用会执行with语句内部的所有语句，因此打印出hello和world；
最后执行yield之后的语句，打印出&lt;/h1&gt;。
因此，@contextmanager让我们通过编写generator来简化上下文管理。

## @closing

如果一个对象没有实现上下文，我们就不能把它用于with语句。这个时候，可以用closing()来把该对象变为上下文对象。例如，用with语句使用urlopen()：

from contextlib import closing
from urllib.request import urlopen

with closing(urlopen(&#x27;https://www.python.org&#x27;)) as page:
    for line in page:
        print(line)
        
closing也是一个经过@contextmanager装饰的generator，这个generator编写起来其实非常简单：

@contextmanager
def closing(thing):
    try:
        yield thing
    finally:
        thing.close()
它的作用就是把任意对象变为上下文对象，并支持with语句
</code></pre>

## HTMLParser
<pre><code class="language-python line-numbers">如果我们要编写一个搜索引擎，第一步是用爬虫把目标网站的页面抓下来，第二步就是解析该HTML页面，看看里面的内容到底是新闻、图片还是视频。

假设第一步已经完成了，第二步应该如何解析HTML呢？

HTML本质上是XML的子集，但是HTML的语法没有XML那么严格，所以不能用标准的DOM或SAX来解析HTML。

好在Python提供了HTMLParser来非常方便地解析HTML，只需简单几行代码：

from html.parser import HTMLParser
from html.entities import name2codepoint

class MyHTMLParser(HTMLParser):

    def handle_starttag(self, tag, attrs):
        print(&#x27;&lt;%s&gt;&#x27; % tag)

    def handle_endtag(self, tag):
        print(&#x27;&lt;/%s&gt;&#x27; % tag)

    def handle_startendtag(self, tag, attrs):
        print(&#x27;&lt;%s/&gt;&#x27; % tag)

    def handle_data(self, data):
        print(data)

    def handle_comment(self, data):
        print(&#x27;&lt;!--&#x27;, data, &#x27;--&gt;&#x27;)

    def handle_entityref(self, name):
        print(&#x27;&amp;%s;&#x27; % name)

    def handle_charref(self, name):
        print(&#x27;&amp;#%s;&#x27; % name)

parser = MyHTMLParser()
parser.feed(&#x27;&#x27;&#x27;&lt;html&gt;
&lt;head&gt;&lt;/head&gt;
&lt;body&gt;
&lt;!-- test html parser --&gt;
    &lt;p&gt;Some &lt;a href=\&quot;#\&quot;&gt;html&lt;/a&gt; HTML&amp;nbsp;tutorial...&lt;br&gt;END&lt;/p&gt;
&lt;/body&gt;&lt;/html&gt;&#x27;&#x27;&#x27;)

feed()方法可以多次调用，也就是不一定一次把整个HTML字符串都塞进去，可以一部分一部分塞进去。

特殊字符有两种，一种是英文表示的&amp;nbsp;，一种是数字表示的&amp;#1234;，这两种字符都可以通过Parser解析出来
</code></pre>

## urllib
<pre><code class="language-python line-numbers">## Get

urllib的request模块可以非常方便地抓取URL内容，也就是发送一个GET请求到指定的页面，然后返回HTTP的响应：

例如，对豆瓣的一个URLhttps://api.douban.com/v2/book/2129650进行抓取，并返回响应：

from urllib import request

with request.urlopen(&#x27;https://api.douban.com/v2/book/2129650&#x27;) as f:
    data = f.read()
    print(&#x27;Status:&#x27;, f.status, f.reason)
    for k, v in f.getheaders():
        print(&#x27;%s: %s&#x27; % (k, v))
    print(&#x27;Data:&#x27;, data.decode(&#x27;utf-8&#x27;))
可以看到HTTP响应的头和JSON数据：

Status: 200 OK
Server: nginx
Date: Tue, 26 May 2015 10:02:27 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 2049
Connection: close
Expires: Sun, 1 Jan 2006 01:00:00 GMT
Pragma: no-cache
Cache-Control: must-revalidate, no-cache, private
X-DAE-Node: pidl1
Data: {&quot;rating&quot;:{&quot;max&quot;:10,&quot;numRaters&quot;:16,&quot;average&quot;:&quot;7.4&quot;,&quot;min&quot;:0},&quot;subtitle&quot;:&quot;&quot;,&quot;author&quot;:[&quot;廖雪峰编著&quot;],&quot;pubdate&quot;:&quot;2007-6&quot;,&quot;tags&quot;:[{&quot;count&quot;:20,&quot;name&quot;:&quot;spring&quot;,&quot;title&quot;:&quot;spring&quot;}...}
如果我们要想模拟浏览器发送GET请求，就需要使用Request对象，通过往Request对象添加HTTP头，我们就可以把请求伪装成浏览器。例如，模拟iPhone 6去请求豆瓣首页：

from urllib import request

req = request.Request(&#x27;http://www.douban.com/&#x27;)
req.add_header(&#x27;User-Agent&#x27;, &#x27;Mozilla/6.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/8.0 Mobile/10A5376e Safari/8536.25&#x27;)
with request.urlopen(req) as f:
    print(&#x27;Status:&#x27;, f.status, f.reason)
    for k, v in f.getheaders():
        print(&#x27;%s: %s&#x27; % (k, v))
    print(&#x27;Data:&#x27;, f.read().decode(&#x27;utf-8&#x27;))
这样豆瓣会返回适合iPhone的移动版网页：

...
    &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, user-scalable=no, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0&quot;&gt;
    &lt;meta name=&quot;format-detection&quot; content=&quot;telephone=no&quot;&gt;
    &lt;link rel=&quot;apple-touch-icon&quot; sizes=&quot;57x57&quot; href=&quot;http://img4.douban.com/pics/cardkit/launcher/57.png&quot; /&gt;
...

## Post

如果要以POST发送一个请求，只需要把参数data以bytes形式传入。

我们模拟一个微博登录，先读取登录的邮箱和口令，然后按照weibo.cn的登录页的格式以username=xxx&amp;password=xxx的编码传入：

from urllib import request, parse

print(&#x27;Login to weibo.cn...&#x27;)
email = input(&#x27;Email: &#x27;)
passwd = input(&#x27;Password: &#x27;)
login_data = parse.urlencode([
    (&#x27;username&#x27;, email),
    (&#x27;password&#x27;, passwd),
    (&#x27;entry&#x27;, &#x27;mweibo&#x27;),
    (&#x27;client_id&#x27;, &#x27;&#x27;),
    (&#x27;savestate&#x27;, &#x27;1&#x27;),
    (&#x27;ec&#x27;, &#x27;&#x27;),
    (&#x27;pagerefer&#x27;, &#x27;https://passport.weibo.cn/signin/welcome?entry=mweibo&amp;r=http%3A%2F%2Fm.weibo.cn%2F&#x27;)
])

req = request.Request(&#x27;https://passport.weibo.cn/sso/login&#x27;)
req.add_header(&#x27;Origin&#x27;, &#x27;https://passport.weibo.cn&#x27;)
req.add_header(&#x27;User-Agent&#x27;, &#x27;Mozilla/6.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/8.0 Mobile/10A5376e Safari/8536.25&#x27;)
req.add_header(&#x27;Referer&#x27;, &#x27;https://passport.weibo.cn/signin/login?entry=mweibo&amp;res=wel&amp;wm=3349&amp;r=http%3A%2F%2Fm.weibo.cn%2F&#x27;)

with request.urlopen(req, data=login_data.encode(&#x27;utf-8&#x27;)) as f:
    print(&#x27;Status:&#x27;, f.status, f.reason)
    for k, v in f.getheaders():
        print(&#x27;%s: %s&#x27; % (k, v))
    print(&#x27;Data:&#x27;, f.read().decode(&#x27;utf-8&#x27;))
如果登录成功，我们获得的响应如下：

Status: 200 OK
Server: nginx/1.2.0
...
Set-Cookie: SSOLoginState=1432620126; path=/; domain=weibo.cn
...
Data: {&quot;retcode&quot;:20000000,&quot;msg&quot;:&quot;&quot;,&quot;data&quot;:{...,&quot;uid&quot;:&quot;1658384301&quot;}}
如果登录失败，我们获得的响应如下：

...
Data: {&quot;retcode&quot;:50011015,&quot;msg&quot;:&quot;\u7528\u6237\u540d\u6216\u5bc6\u7801\u9519\u8bef&quot;,&quot;data&quot;:{&quot;username&quot;:&quot;example@python.org&quot;,&quot;errline&quot;:536}}

## Handler

如果还需要更复杂的控制，比如通过一个Proxy去访问网站，我们需要利用ProxyHandler来处理，示例代码如下：

proxy_handler = urllib.request.ProxyHandler({&#x27;http&#x27;: &#x27;http://www.example.com:3128/&#x27;})
proxy_auth_handler = urllib.request.ProxyBasicAuthHandler()
proxy_auth_handler.add_password(&#x27;realm&#x27;, &#x27;host&#x27;, &#x27;username&#x27;, &#x27;password&#x27;)
opener = urllib.request.build_opener(proxy_handler, proxy_auth_handler)
with opener.open(&#x27;http://www.example.com/login.html&#x27;) as f:
    pass
    

#### 其他(自定义http头部)
#!/bin/env python3
# -*- coding: utf-8 -*-

from urllib import request

url=&#x27;http://yaohuo.me/myfile.aspx&#x27;

headers={
&#x27;User-Agent&#x27;:&#x27;Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36&#x27;,
&#x27;Referer&#x27;:&#x27;http://yaohuo.me/wapindex.aspx?sid=-2&#x27;,
&#x27;Cookie&#x27;:&#x27;yunsuo_session_verify=beaab26e33f102e5a45fee5328d11a3a; ASP.NET_SessionId=wjs1b0it4y54w3neo2jq4f55; GUID=86c26f1114141531; _gat=1; sidyaohuo=14CE437ACB4C8CA70_6_17_23509_700100-2; _ga=GA1.2.19527068.1484115252&#x27;
}

req=request.Request(url,headers=headers)

r=request.urlopen(req)

html=b&#x27;&#x27;

while True:
data=r.read(1024)
if data:
html+=data
else:
break

print(html.decode())
</code></pre>
