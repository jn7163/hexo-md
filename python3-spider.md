---
title: python3 爬虫
date: 2016-12-09 13:14:00
categories:
- python
tags:
- python
keywords: python, python3, 爬虫
---
> 
python3 `urllib/requests/bs4`模块
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

## urllib
<pre><code class="language-python line-numbers">from urllib import request

url='https://www.zfl9.com'
ua={'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.87 Safari/537.36'}
req=request.Request(url,headers=ua)
r=request.urlopen(req)

## status
status_code=r.code
status_msg=r.msg
# or
status_reason=r.reason
print(status_code,status_msg)

## headers
headers=r.headers
print(headers)

## url
url=r.url

## html
html=r.read().decode('utf-8')
print(html)
</code></pre>

## requests
`pip install requests`  安装requests模块

<pre><code class="language-python line-numbers">import requests
url='https://www.zfl9.com'
ua={'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.87 Safari/537.36'}
r=requests.get(url,headers=ua)

## status
status_code=r.status_code
status_reason=r.reason
print(status_code,status_reason)

## headers
headers=r.headers

## url
url=r.url

## method
method=r.request

## cookies
cookies=r.cookies

## encoding
r.encoding
r.encoding='utf-8'

## html
r.text # str数据
r.content # bytes数据
r.raw # 原始(bytes)数据
</code></pre>

## beautifulsoup
`pip install beautifulsoup4`    安装bs4模块
<pre><code class="language-python line-numbers">from bs4 import BeautifulSoup as BSP
import requests

url='http://www.baidu.com'
ua={'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.87 Safari/537.36'}
r=requests.get(url,headers=ua)
html=r.text

## 创建bsp实例
bsp=BSP(html)
## 打印html(自动缩进)
print(bsp.prettify())
## title、p、a等标签
bsp.title
bsp.title.name
bsp.title.string
bsp.p
bsp.a
## find_all
bsp.find_all('a')
for link in bsp.find_all('a'):
    print(link.get('href'))
## link
bsp.a.get('href')
## text
bsp.get_text()


### 运行beautifulsoup 是发现抛出一个异常

/usr/local/python3.6.0/lib/python3.6/site-packages/bs4/__init__.py:181: UserWarning: No parser was explicitly specified, so I'm using the best available HTML parser for this system ("html.parser"). This usually isn't a problem, but if you run this code on another system, or in a different virtual environment, it may use a different parser and behave differently.

The code that caused this warning is on line 32 of the file ./test_urllib.py. To get rid of this warning, change code that looks like this:
BeautifulSoup([your markup])
to this:
BeautifulSoup([your markup], "html.parser")
markup_type=markup_type))

按照提示 BeautifulSoup(html,'html.parser') 修改后就OK了
</code></pre>
