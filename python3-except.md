﻿---
title: python3 异常处理
date: 2016-12-09 11:33:00
categories:
- python
tags:
- python
keywords: python, python3, 错误处理
---
> 
python3 异常处理
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

`try...except...except...else...finally...`
<pre><code class="language-python line-numbers">当我们认为某些代码可能会出错时，就可以用try来运行这段代码，如果执行出错，则后续代码不会继续执行，而是直接跳转至错误处理代码，即except语句块，执行完except后，如果有finally语句块，则执行finally语句块，至此，执行完毕

finally语句块无论有没有错误都会执行
可以有多个except语句块。
此外，如果没有错误发生，可以在except语句块后面加一个else，当没有错误发生时，会自动执行else语句

try:
    print('try...')
    r = 10 / 0
    print('result:', r)
except ZeroDivisionError as e:
    print('except:', e)
finally:
    print('finally...')
    
try:
    print('try...')
    r = 10 / int('a')
    print('result:', r)
except ValueError as e:
    print('ValueError:', e)
except ZeroDivisionError as e:
    print('ZeroDivisionError:', e)
finally:
    print('finally...')
    
try:
    print('try...')
    r = 10 / int('2')
    print('result:', r)
except ValueError as e:
    print('ValueError:', e)
except ZeroDivisionError as e:
    print('ZeroDivisionError:', e)
else:
    print('no error!')
finally:
    print('finally...')
    
Python的错误其实也是class，所有的错误类型都继承自BaseException

所以在使用except时需要注意的是，它不但捕获该类型的错误，还把其子类也“一网打尽”

使用try...except捕获错误还有一个巨大的好处，就是可以跨越多层调用
比如函数main()调用foo()，foo()调用bar()，结果bar()出错了，这时，只要main()捕获到了，就可以处理：

def foo(s):
    return 10 / int(s)

def bar(s):
    return foo(s) * 2

def main():
    try:
        bar('0')
    except Exception as e:
        print('Error:', e)
    finally:
        print('finally...')
        
也就是说，不需要在每个可能出错的地方去捕获错误，只要在合适的层次去捕获错误就可以了。这样一来，就大大减少了写try...except...finally的麻烦。
</code></pre>

## 调用堆栈

如果错误没有被捕获，它就会一直往上抛，最后被Python解释器捕获，打印一个错误信息，然后程序退出

## 记录错误
<pre><code class="language-python line-numbers">如果不捕获错误，自然可以让Python解释器来打印出错误堆栈，但程序也被结束了。
既然我们能捕获错误，就可以把错误堆栈打印出来，然后分析错误原因，同时，让程序继续执行下去

Python内置的logging模块可以非常容易地记录错误信息

import logging

def foo(s):
    return 10 / int(s)

def bar(s):
    return foo(s) * 2

def main():
    try:
        bar('0')
    except Exception as e:
        logging.exception(e)

main()
print('END')

同样是出错，但程序打印完错误信息后会继续执行，并正常退出：

$ python3 err_logging.py
ERROR:root:division by zero
Traceback (most recent call last):
  File "err_logging.py", line 13, in main
    bar('0')
  File "err_logging.py", line 9, in bar
    return foo(s) * 2
  File "err_logging.py", line 6, in foo
    return 10 / int(s)
ZeroDivisionError: division by zero
END


### ### ### ### ### ### ### ### ###

subprocess -- python3推荐使用的执行外部shell模块

1. 获取 exit_code
import subprocess
a=subprocess.call('ls -l',shell=True)


2. 获取 stdout
import subprocess
a=subprocess.getoutput('ls -l')
print(a)


## logging 模块 ##

简单使用
#!/usr/local/bin/python
# -*- coding:utf-8 -*-
import logging

logging.debug('debug message')
logging.info('info message')
logging.warn('warn message')
logging.error('error message')
logging.critical('critical message')

输出：

WARNING:root:warn message
ERROR:root:error message
CRITICAL:root:critical message

默认情况下，logging模块将日志打印到屏幕上(stdout)，日志级别为WARNING(即只有日志级别高于WARNING的日志信息才会输出)


简单配置

#!/usr/local/bin/python
# -*- coding:utf-8 -*-
import logging

# 通过下面的方式进行简单配置输出方式与日志级别
logging.basicConfig(filename='logger.log', level=logging.INFO)

logging.debug('debug message')
logging.info('info message')
logging.warn('warn message')
logging.error('error message')
logging.critical('critical message')


常用配置

#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import logging,os

logger=logging.getLogger(os.path.abspath(__file__))
logger.setLevel(logging.DEBUG)

handler=logging.FileHandler('log.log')
handler.setLevel(logging.DEBUG)

formatter=logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

logger.addHandler(handler)

logger.info('Hello,Otokaze!')

--- log.log ---
2017-01-10 19:21:11,723 - /root/test_logging.py - INFO - Hello,Otokaze!
--- log.log ---
</code></pre>

## 抛出错误
<pre><code class="language-python line-numbers">因为错误是class，捕获一个错误就是捕获到该class的一个实例。
因此，错误并不是凭空产生的，而是有意创建并抛出的。
Python的内置函数会抛出很多类型的错误，我们自己编写的函数也可以抛出错误。

如果要抛出错误，首先根据需要，可以定义一个错误的class，选择好继承关系，然后，用raise语句抛出一个错误的实例：

class FooError(ValueError):
    pass

def foo(s):
    n = int(s)
    if n==0:
        raise FooError('invalid value: %s' % s)
    return 10 / n

foo('0')

执行，可以最后跟踪到我们自己定义的错误：

$ python3 err_raise.py 
Traceback (most recent call last):
  File "err_throw.py", line 11, in <module>
    foo('0')
  File "err_throw.py", line 8, in foo
    raise FooError('invalid value: %s' % s)
__main__.FooError: invalid value: 0

只有在必要的时候才定义我们自己的错误类型。如果可以选择Python已有的内置的错误类型（比如ValueError，TypeError），尽量使用Python内置的错误类型。

最后，我们来看另一种错误处理的方式：

# err_reraise.py

def foo(s):
    n = int(s)
    if n==0:
        raise ValueError('invalid value: %s' % s)
    return 10 / n

def bar():
    try:
        foo('0')
    except ValueError as e:
        print('ValueError!')
        raise

bar()

在bar()函数中，我们明明已经捕获了错误，但是，打印一个ValueError!后，又把错误通过raise语句抛出去了，这不有病么？

其实这种错误处理方式不但没病，而且相当常见。捕获错误目的只是记录一下，便于后续追踪。但是，由于当前函数不知道应该怎么处理该错误，所以，最恰当的方式是继续往上抛，让顶层调用者去处理。好比一个员工处理不了一个问题时，就把问题抛给他的老板，如果他的老板也处理不了，就一直往上抛，最终会抛给CEO去处理。

raise语句如果不带参数，就会把当前错误原样抛出
</code></pre>
