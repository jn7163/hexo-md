---
title: python3 web开发
date: 2016-12-09 12:55:00
categories:
- python
tags:
- python
keywords: python, python3, web开发
---
> 
python3 web开发
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

## wsgi
`wsgi: Web Server Gateway Interface`

### html.py
<pre><code class="language-python line-numbers">def html(environ,status): ## environ是一个包含http请求信息的dict对象，status是http响应
    status(&#x27;200 OK&#x27;,[(&#x27;Content-Type&#x27;,&#x27;text/html&#x27;)])
    body=&#x27;&lt;h1&gt;Hello, %s!&lt;/h1&gt;&#x27; % (environ[&#x27;PATH_INFO&#x27;][1:] or &#x27;Otokaze&#x27;)
    return [body.encode(&#x27;utf-8&#x27;)]
</code></pre>

### server.py
<pre><code class="language-python line-numbers">from wsgiref.simple_server import make_server
from test_web_html import html

httpd=make_server(&#x27;&#x27;,8000,html)
print(&#x27;Listen http request on 8000......&#x27;)
httpd.serve_forever()
</code></pre>

## web框架
### flask
<pre><code class="language-python line-numbers">pip install flask
</code></pre>

### flask_server.py
<pre><code class="language-python line-numbers">from flask import Flask,request,render_template

app=Flask(__name__)
rt=render_template

@app.route('/',methods=['GET'])
def index():
    return rt('index.html')

@app.route('/index.html',methods=['GET'])
def index_2():
    return rt('index.html')

@app.route('/login.html',methods=['GET'])
def login():
    return rt('login.html')

@app.route('/login.html',methods=['POST'])
def welcome():
    user=request.form['username']
    passwd=request.form['password']
    if user=='root' and passwd=='123456':
        return rt('welcome.html',username=user) ## username变量传递给了templates的html页面
    return rt('login.html',message='Bad username or password!',username=user) ## 同上！

if __name__=='__main__':
    app.run(host='0.0.0.0',port=80,debug=True)
</code></pre>

## jinja2模板
<pre><code class="language-python line-numbers">在Jinja2模板中，我们用{{ name }}表示一个需要替换的变量。
很多时候，还需要循环、条件判断等指令语句，在Jinja2中，用{% ... %}表示指令。

比如循环输出页码：

{% for i in page_list %}
    &lt;a href=&quot;/page/{{ i }}&quot;&gt;{{ i }}&lt;/a&gt;
{% endfor %}

如果page_list是一个list：[1, 2, 3, 4, 5]，上面的模板将输出5个超链接。
</code></pre>

## MVC
> 
`Model-View-Controller` - "模型-视图-控制器"
Python处理URL的函数就是C：Controller，Controller负责业务逻辑
比如检查用户名是否存在，取出用户信息等等；
> 
包含变量{{ name }}的模板就是V：View，View负责显示逻辑
通过简单地替换一些变量，View最终输出的就是用户看到的HTML。
> 
MVC中的Model在哪？Model是用来传给View的
这样View在替换变量的时候，就可以从Model中取出相应的数据

## cgi(请忽略)
<pre><code class="language-python line-numbers">-------- index.py --------
#!/usr/bin/python3
# -*- coding: utf-8 -*-

import cgi,os

print(&#x27;Content-type: text/html\n&#x27;)
print(&#x27;&lt;meta charset=&quot;UTF-8&quot;&gt;&#x27;)
print(&#x27;&lt;title&gt;dashboard - hexo&lt;/title&gt;&#x27;)

try:
    form=cgi.FieldStorage()
    fileitem=form[&#x27;filename&#x27;]
    filename=fileitem.filename
    with open(&#x27;/tmp/%s&#x27; % filename, &#x27;wb&#x27;) as f:
        data=fileitem.file.read()
        f.write(data)
    shell=os.system(&#x27;sudo mv /tmp/%s /root/www.zfl9.com/source/_posts/&#x27; % filename)
    if shell == 0:
        msg=&#x27;%s 上传成功!&#x27; % filename
    print(&#x27;&lt;/h4&gt;%s&lt;h4&gt;&#x27; % msg)
except BaseException:
    pass

print(&#x27;&#x27;&#x27;
&lt;form enctype=&quot;multipart/form-data&quot; action=&quot;/&quot; method=&quot;post&quot;&gt;
&lt;p&gt;&lt;input type=&quot;file&quot; name=&quot;filename&quot; /&gt;&lt;/p&gt;
&lt;p&gt;&lt;input type=&quot;submit&quot; value=&quot;上传文件&quot; /&gt;&lt;/p&gt;
&lt;/form&gt;
&lt;form action=&quot;/hexo.py&quot; method=&quot;post&quot;&gt;
&lt;select name=&quot;dropdown&quot;&gt;
&lt;option value=&quot;hexo&quot; selected&gt;渲染markdown&lt;/option&gt;
&lt;/select&gt;
&lt;input type=&quot;submit&quot; value=&quot;确定&quot;/&gt;
&lt;/form&gt;
&#x27;&#x27;&#x27;)
----------- index.py ---------------

---------- hexo.py -----------
#!/usr/bin/python3
# -*- coding: utf-8 -*-

import cgi,os

print(&#x27;Content-type: text/html\n&#x27;)
print(&#x27;&lt;meta charset=&quot;UTF-8&quot;&gt;&#x27;)
print(&#x27;&lt;title&gt;dashboard - hexo&lt;/title&gt;&#x27;)

try:
    form=cgi.FieldStorage()
    hexo=form.getvalue(&#x27;dropdown&#x27;)
except BaseException:
    pass

if hexo == &#x27;hexo&#x27;:
    shell=os.system(&#x27;sudo /sbin/myhexo&#x27;)
    if shell == 0:
        msg=&#x27;markdown 渲染完成!&#x27;
else:
    msg=&#x27;markdown 渲染失败!&#x27;
    print(&#x27;&lt;h2&gt;%s&lt;/h2&gt;&#x27; % msg)
---------- hexo.py -----------
</code></pre>
