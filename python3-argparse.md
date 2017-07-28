---
title: python3 argparse参数解析
date: 2017-05-09 14:37:00
categories:
- python
tags:
- python
- argparse参数解析
keywords: argparse参数解析
---

> 
自带的`sys.argv[]`解析一些简单的参数没什么问题，但是稍微复杂一些就工作量大了;
`argparse`是从python2.7引入的参数解析模块，功能非常强大，是一个全面的参数处理库

<!-- more -->

## sys.argv
`sys.argv`是一个包含所有命令行参数的`list`, 第一个元素是当前执行文件的文件名
<pre><code class="language-python line-numbers"><script type="text/plain">#/bin/env python3
# coding: utf-8
import sys
print(type(sys.argv))
print(sys.argv)
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain">$ chmod +x test.py
$ ./test.py
<class 'list'>
['./test.py']
$ ./test.py -a -b -c
<class 'list'>
['./test.py', '-a', '-b', '-c']
</script></code></pre>

## argparse
> 通过一个例子理解一下命令和命令参数

<pre><code class="language-python line-numbers"><script type="text/plain">[root@localhost ~]# ls
argparse_learn.py  jdlingyu
[root@localhost ~]# ls jdlingyu/
README.md  site.py  theme.py
[root@localhost ~]# ls -l
total 4
-rwxr-xr-x 1 root root 893 May  9 14:35 argparse_learn.py
drwxr-xr-x 3 root root  66 May  8 21:41 jdlingyu
[root@localhost ~]# ls --help
Usage: ls [OPTION]... [FILE]...
List information about the FILEs (the current directory by default).
Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.
......
</script></code></pre>

> `位置参数`，`可选参数`

<pre><code class="language-python line-numbers"><script type="text/plain">ls              # 不带参数，显示当前路径下的文件
ls jdlingyu/    # 位置参数，显示指定文件
ls -l           # 可选参数，显示文件的详细信息
ls --help       # 可选参数，显示帮助信息
</script></code></pre>

> 
先从最简单的开始，创建一个`argparse实例`

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.parse_args()
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [15:34:24]
$ ./test.py

# root @ localhost in ~ [15:34:26]
$ ./test.py -l
usage: test.py [-h]
test.py: error: unrecognized arguments: -l

# root @ localhost in ~ [15:34:39] C:2
$ ./test.py -h
usage: test.py [-h]

learn argparse module

optional arguments:
  -h, --help  show this help message and exit

# root @ localhost in ~ [15:34:43]
$ ./test.py --help
usage: test.py [-h]

learn argparse module

optional arguments:
  -h, --help  show this help message and exit

# root @ localhost in ~ [15:34:48]
$ ./test.py test
usage: test.py [-h]
test.py: error: unrecognized arguments: test
</script></code></pre>

> 
argparse自带了`-h/--help`参数，除此之外，test.py没有任何定义参数，也不能处理参数
我们现在定义一个`位置参数`，所谓的位置参数就是必须要有的参数，否则程序不能运行

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.add_argument("echo")
args=parser.parse_args()

print(args.echo)
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [15:41:02]
$ ./test.py
usage: test.py [-h] echo
test.py: error: the following arguments are required: echo

# root @ localhost in ~ [15:42:12] C:2
$ ./test.py test
test

# root @ localhost in ~ [15:42:19]
$ ./test.py -h
usage: test.py [-h] echo

learn argparse module

positional arguments:
  echo

optional arguments:
  -h, --help  show this help message and exit
</script></code></pre>

> 
注意`"echo"`可以为任意字符，这只是一个命名，但是别人并不知道你的`"echo"`参数是做什么的，我们再完善一下

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.add_argument("echo", help='echo the string you use here')
args=parser.parse_args()

print(args.echo)
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [15:45:50] C:2
$ ./test.py test
test

# root @ localhost in ~ [15:45:53]
$ ./test.py -h
usage: test.py [-h] echo

learn argparse module

positional arguments:
  echo        echo the string you use here

optional arguments:
  -h, --help  show this help message and exit
</script></code></pre>

> 
我们现在定义一个`位置参数`，输出它的平方数

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.add_argument("square", help='display a square of a given number')
args=parser.parse_args()

print(args.square ** 2)
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain">$ ./test.py
usage: test.py [-h] square
test.py: error: the following arguments are required: square

# root @ localhost in ~ [15:49:42] C:2
$ ./test.py 2
Traceback (most recent call last):
  File "./test.py", line 8, in <module>
    print(args.square ** 2)
TypeError: unsupported operand type(s) for ** or pow(): 'str' and 'int'

# root @ localhost in ~ [15:49:44] C:1
$ ./test.py -h
usage: test.py [-h] square

learn argparse module

positional arguments:
  square      display a square of a given number

optional arguments:
  -h, --help  show this help message and exit
</script></code></pre>

> 
what? 不能运行，TypeError，argparse默认将参数的type定为`str`，我们改一下

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.add_argument("square", help='display a square of a given number', type=int)
args=parser.parse_args()

print(args.square ** 2)
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [15:51:51]
$ ./test.py 5
25
</script></code></pre>

> 
很好，我们现在定义一个可选参数`--verbose`，用于显示详细信息

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.add_argument("square", help='display a square of a given number', type=int)
parser.add_argument('--verbose', help='display verbose info', action='store_true')
args=parser.parse_args()

dist=args.square ** 2

if args.verbose:
    print('{} * {} = {}'.format(args.square,args.square,dist))
else:
    print(dist)
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [15:57:36]
$ ./test.py 5
25

# root @ localhost in ~ [15:57:42]
$ ./test.py 5 --verbose
5 * 5 = 25
</script></code></pre>

> 
`action='store'`        保存参数值，默认为该动作
`action='store_true'`   表示将参数存储为布尔值`True`，就是说如果参数存在，则为`True`，否则为`False`
`action='store_false'`  与之相反
`action='append'`       将值保存到一个`list`中，若参数重复出现，则保存多个值
`action='count'`        记录参数的个数
`action='version'`      打印关于程序的版本信息，然后退出

熟悉命令行的可能知道，除了`--verbose`这种长选项外，更多的是`-v`这种短选项的格式
<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.add_argument("square", help='display a square of a given number', type=int)
parser.add_argument('-v', '--verbose', help='display verbose info', action='store_true')
args=parser.parse_args()

dist=args.square ** 2

if args.verbose:
    print('{} * {} = {}'.format(args.square,args.square,dist))
else:
    print(dist)
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [16:07:32]
$ ./test.py 5 -v
5 * 5 = 25

# root @ localhost in ~ [16:07:35]
$ ./test.py 5 --verbose
5 * 5 = 25
</script></code></pre>

> `version` 版本号

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.add_argument("square", help='display a square of a given number', type=int)
parser.add_argument('-v', '--verbose', help='display verbose info', action='store_true')
parser.add_argument('-V', '--version', help='display program version', action='version', version='%(prog)s v1.0')
args=parser.parse_args()

dist=args.square ** 2

if args.verbose:
    print('{} * {} = {}'.format(args.square,args.square,dist))
else:
    print(dist)
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [16:14:02]
$ ./test.py -V
test.py v1.0
</script></code></pre>

> 
`default` 设定默认值

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.add_argument("square", help='display a square of a given number', type=int)
parser.add_argument('-v', '--verbose', help='display verbose info', action='store_true', default=True)
parser.add_argument('-V', '--version', help='display program version', action='version', version='%(prog)s v1.0')
args=parser.parse_args()

dist=args.square ** 2

if args.verbose:
    print('{} * {} = {}'.format(args.square,args.square,dist))
else:
    print(dist)
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [16:19:58] C:127
$ ./test.py 4
4 * 4 = 16
</script></code></pre>

> 
`-v` `-vv` `-vvv` 参数个数计数

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.add_argument("square", help='display a square of a given number', type=int)
parser.add_argument('-v', '--verbose', help='display verbose info', action='count', default=0)
parser.add_argument('-V', '--version', help='display program version', action='version', version='%(prog)s v1.0')
args=parser.parse_args()

dist=args.square ** 2

if args.verbose >= 2:
    print('{} 的 平方数为 {}'.format(args.square, dist))
if args.verbose == 1:
    print('{} * {} = {}'.format(args.square,args.square,dist))
else:
    print(dist)
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [16:32:27] C:2
$ ./test.py 3
9

# root @ localhost in ~ [16:32:28]
$ ./test.py 3 -v
3 * 3 = 9

# root @ localhost in ~ [16:32:31]
$ ./test.py 3 -vv
3 的 平方数为 9
9

# root @ localhost in ~ [16:32:32]
$ ./test.py 3 -vvv
3 的 平方数为 9
9
</script></code></pre>

> 
`required` 表示该可选参数默认被请求，相当于必选

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.add_argument("-s", '--square', help='display a square of a given number', type=int, required=True)
parser.add_argument('-v', '--verbose', help='display verbose info', action='count', default=0)
parser.add_argument('-V', '--version', help='display program version', action='version', version='%(prog)s v1.0')
args=parser.parse_args()

dist=args.square ** 2

if args.verbose >= 2:
    print('{} 的 平方数为 {}'.format(args.square, dist))
if args.verbose == 1:
    print('{} * {} = {}'.format(args.square,args.square,dist))
else:
    print(dist)
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [16:36:02]
$ ./test.py -s9
81

# root @ localhost in ~ [16:36:18]
$ ./test.py -s9 -v
9 * 9 = 81

# root @ localhost in ~ [16:36:20]
$ ./test.py -s9 -vv
9 的 平方数为 81
81

# root @ localhost in ~ [16:36:21]
$ ./test.py
usage: test.py [-h] -s SQUARE [-v] [-V]
test.py: error: the following arguments are required: -s/--square
</script></code></pre>

> 
`-q/--quiet` 简洁输出

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
parser.add_argument("-s", '--square', help='display a square of a given number', type=int, required=True)
parser.add_argument('-v', '--verbose', help='display verbose info', action='store_true')
parser.add_argument('-q', '--quiet', help='don\'t display verbose info', action='store_true')
parser.add_argument('-V', '--version', help='display program version', action='version', version='%(prog)s v1.0')
args=parser.parse_args()

dist=args.square ** 2

if args.quiet:
    print(dist)
elif args.verbose:
    print('{} 的 平方数为 {}'.format(args.square, dist))
else:
    print('{} * {} = {}'.format(args.square,args.square,dist))
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [16:43:37]
$ ./test.py -s8
8 * 8 = 64

# root @ localhost in ~ [16:43:56]
$ ./test.py -s8 -q
64

# root @ localhost in ~ [16:43:58]
$ ./test.py -s8 -v
8 的 平方数为 64

# root @ localhost in ~ [16:44:00]
$ ./test.py -s8 -vq
64

# root @ localhost in ~ [16:44:06]
$ ./test.py -s8 -qv
64
</script></code></pre>

> 
注意到一个问题，当`-q` `-v`同时出现时，会发生冲突，如何实现要么`-q`，要么`-v`

<pre><code class="language-python line-numbers"><script type="text/plain">import argparse
parser=argparse.ArgumentParser(description='learn argparse module')
group=parser.add_mutually_exclusive_group()
group.add_argument('-v', '--verbose', help='display verbose info', action='store_true')
group.add_argument('-q', '--quiet', help='don\'t display verbose info', action='store_true')
parser.add_argument("-s", '--square', help='display a square of a given number', type=int, required=True)
parser.add_argument('-V', '--version', help='display program version', action='version', version='%(prog)s v1.0')
args=parser.parse_args()

dist=args.square ** 2

if args.quiet:
    print(dist)
elif args.verbose:
    print('{} 的 平方数为 {}'.format(args.square, dist))
else:
    print('{} * {} = {}'.format(args.square,args.square,dist))
</script></code></pre>

<pre><code class="language-python line-numbers"><script type="text/plain"># root @ localhost in ~ [16:46:54]
$ ./test.py -s8
8 * 8 = 64

# root @ localhost in ~ [16:47:08]
$ ./test.py -s8 -v
8 的 平方数为 64

# root @ localhost in ~ [16:47:10]
$ ./test.py -s8 -q
64

# root @ localhost in ~ [16:47:11]
$ ./test.py -s8 -qv
usage: test.py [-h] [-v | -q] -s SQUARE [-V]
test.py: error: argument -v/--verbose: not allowed with argument -q/--quiet
</script></code></pre>

> 
argparse的功能比这里演示的多得多，有需要的请看[官方文档](https://docs.python.org/3/library/argparse.html)
还有一点，windows下命令参数可能用`/v` `/verbose`这种格式，argparse也是支持的，上面的全部例子在windows下都可以用这个格式
