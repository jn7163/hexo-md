---
title: python3 正则表达式模块
date: 2016-12-09 11:55:00
categories:
- python
tags:
- python
keywords: python, python3, 正则表达式模块
---
> 
python3 正则表达式模块
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

>由于python字符串也用\转义。所以强烈建议re模式用r前缀！！！

### re.match
<pre><code class="language-python line-numbers"><script type="text/plain">re.match
尝试从字符串的起始位置匹配一个模式，如果不是起始位置匹配成功的话，match()就返回None

语法：
re.match(pattern, string, flags=0)

pattern   # re模式字符串
string    # 原字符串
flags     # 标志位，控制re的匹配方式
</script></code></pre>

### flags
<pre><code class="language-python line-numbers"><script type="text/plain"># 修饰符
re.I    # 使匹配对大小写不敏感
re.L    # 做本地化识别(locale-aware)匹配
re.M    # 多行匹配，影响 ^ 和 $
re.S    # 使 . 匹配包括 \n 在内的所有字符
re.U    # 根据Unicode字符集解析字符, 这个标志影响 \w \W \b \B
re.X    # 该标志通过给予你更灵活的格式以便你将正则表达式写得更易于理解

# 单行模式和多行模式
## 单行模式
更改点 . 的含义，使它与每一个字符匹配(而不是与除 \n 之外的每个字符匹配)

## 多行模式
更改 ^ 和 $ 的含义，使它们分别在任意一行的行首和行尾匹配，而不仅仅在整个字符串的开头和结尾匹配
</script></code></pre>

### group/groups
<pre><code class="language-python line-numbers"><script type="text/plain">## group(num=0)
匹配的整个表达式的字符串，group() 可以一次输入多个组号，在这种情况下它将返回一个包含那些组所对应值的tuple

## groups()
返回一个包含所有小组字符串的元组，从 1 到 所含的小组号

## 例如
In [22]: test = 'this is test string!'
In [23]: a=re.search(r'^(\w+)\s(\w+)\s(\w+)\s(\w+)!$', test)
In [24]: a
Out[24]: <_sre.SRE_Match object; span=(0, 20), match='this is test string!'>
In [25]: a.group()
Out[25]: 'this is test string!'
In [26]: a.groups()
Out[26]: ('this', 'is', 'test', 'string')
In [27]: a.group(0)
Out[27]: 'this is test string!'
In [28]: a.group(1)
Out[28]: 'this'
In [29]: a.group(2)
Out[29]: 'is'
In [30]: a.group(3)
Out[30]: 'test'
In [31]: a.group(4)
Out[31]: 'string'
In [33]: a.group(1,3)
Out[33]: ('this', 'test')
</script></code></pre>

### re.search
<pre><code class="language-python line-numbers"><script type="text/plain">## re.search
匹配整个字符串，直到找到一个匹配，与re.match的不同就是，re.match是从字符串的起始位置匹配

## re.match
尝试从字符串的起始位置匹配一个模式，如果不是起始位置匹配成功的话，match()就返回None
</script></code></pre>

### re.sub
<pre><code class="language-python line-numbers"><script type="text/plain">## re.sub
正则表达式替换，用过sed或者vim的同学应该比较熟悉

语法：
re.sub(pattern, repl, string, count=0, flags=0)

pattern     # 正则中的模式字符串
repl        # 替换的字符串，也可为一个函数
string      # 要被查找替换的原始字符串
count       # 模式匹配后替换的最大次数，默认 0 表示替换所有的匹配

## 例子
In [36]: src='<h1>test</h1>'
In [38]: dst=re.sub(r'^<.*?>(.*?)<.*?>$', r'<a>\1</a>', src)
In [39]: dst
Out[39]: '<a>test</a>'
</script></code></pre>

### re.compile
<pre><code class="language-python line-numbers"><script type="text/plain">## re.compile
编译re模式字符串，可以多次调用

## 语法
re.compile(pattern, flags=0)

## 例子
In [50]: link="<a href='https://www.zfl9.com/images/git.png'>git</a>"
In [52]: re_url=re.compile(r'^<a\ href=\'(.*?)\'>git<\/a>$')
In [53]: url=re.search(re_url, link)
In [54]: url
Out[54]: <_sre.SRE_Match object; span=(0, 53), match="<a href='https://www.zfl9.com/images/git.png'>git>
In [56]: url.group()
Out[56]: "<a href='https://www.zfl9.com/images/git.png'>git</a>"
In [57]: url.group(1)
Out[57]: 'https://www.zfl9.com/images/git.png'
</script></code></pre>

### re.split
<pre><code class="language-python line-numbers"><script type="text/plain">## re.split
基于re模式字符串分割原字符串

## 语法
re.split(pattern, string, maxsplit=0)

## 分离次数
maxsplit    # 分离的次数，默认为0，不限制次数

## 例子
In [61]: test='this is a test string'
In [63]: a=re.split(r'\s+', test)
In [64]: a
Out[64]: ['this', 'is', 'a', 'test', 'string']
In [65]: a=re.split(r'(\s+)', test)
In [66]: a
Out[66]: ['this', ' ', 'is', ' ', 'a', ' ', 'test', ' ', 'string']

## 可以看到，如果pattern是一个子串的话，也就是用圆括号扩起来，也会在list中返回
</script></code></pre>

### re.findall
<pre><code class="language-python line-numbers"><script type="text/plain">## re.findall
匹配所有子串，并作为一个list返回

## 语法
re.findall(pattern, string, flags=0)

## 例子
In [67]: test='this is a test string'
In [68]: a=re.findall(r'^(\w+)\s(\w+)\s(\w+)\s(\w+)\s(\w+)$', test)
In [69]: a
Out[69]: [('this', 'is', 'a', 'test', 'string')]
In [70]: a=re.findall(r'^((\w+)\s(\w+)\s(\w+)\s(\w+)\s(\w+))$', test)
In [71]: a
Out[71]: [('this is a test string', 'this', 'is', 'a', 'test', 'string')]
In [72]: a=re.findall(r'^((\w+)\s(\w+))\s(\w+)\s(\w+)\s(\w+)$', test)
In [73]: a
Out[73]: [('this is', 'this', 'is', 'a', 'test', 'string')]
In [74]: a=re.findall(r'^(((\w+)\s(\w+))\s(\w+)\s(\w+)\s(\w+))$', test)
In [75]: a
Out[75]: [('this is a test string', 'this is', 'this', 'is', 'a', 'test', 'string')]

## 可以发现，如果有多层括号，是从左往右的顺序依次匹配的
</script></code></pre>

### re.finditer
<pre><code class="language-python line-numbers"><script type="text/plain">## re.finditer
re.finditer和re.findall很像
re.findall是返回一个list，list中的元素是匹配到的字符串
re.finditer是返回一个迭代器，迭代器中的元素是_sre.SRE_Match类的实例

## 用法
re.finditer(pattern, string, flags=0)

我们还是用例子说：
In [26]: test='this is a test string'
In [27]: re_test=re.compile(r'^(\w+)\s(\w+)\s(\w+)\s(\w+)\s(\w+)$')
In [28]: findall=re.findall(re_test, test)
In [29]: findall
Out[29]: [('this', 'is', 'a', 'test', 'string')]
In [30]: finditer=re.finditer(re_test, test)
In [31]: finditer
Out[31]: <callable_iterator at 0x7f3a346ab390>
In [32]: a=list(finditer)
In [33]: a
Out[33]: [<_sre.SRE_Match object; span=(0, 21), match='this is a test string'>]
In [34]: a[0]
Out[34]: <_sre.SRE_Match object; span=(0, 21), match='this is a test string'>
In [35]: a[0].group()
Out[35]: 'this is a test string'
In [36]: a[0].groups()
Out[36]: ('this', 'is', 'a', 'test', 'string')
In [37]: a[0].group(0)
Out[37]: 'this is a test string'
In [38]: a[0].group(1)
Out[38]: 'this'
</script></code></pre>
