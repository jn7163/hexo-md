---
title: python3 电子邮件
date: 2016-12-09 12:33:00
categories:
- python
tags:
- python
keywords: python, python3, 电子邮件
---
> 
python3 电子邮件
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

## 不带附件的邮件
<pre><code class="language-python line-numbers">from email.mime.text import MIMEText
from email import encoders
from email.header import Header
from email.utils import parseaddr,formataddr
import smtplib

def msg_format(s):
    name,addr=parseaddr(s)
    return formataddr((Header(name,&#x27;utf-8&#x27;).encode(&#x27;utf-8&#x27;),addr))

user=input(&#x27;Please enter your email: &#x27;)
passwd=input(&#x27;Please enter the password: &#x27;)
smtp_server=input(&#x27;Please enter your SMTP_Server address: &#x27;)
to=input(&#x27;Send the mail to: &#x27;)
mail_title=input(&#x27;mail title: &#x27;)
#mail_body=input(&#x27;mail body: &#x27;)

#msg=MIMEText(mail_body,&#x27;plain&#x27;,&#x27;utf-8&#x27;)
msg=MIMEText(&#x27;&lt;html&gt;&lt;body&gt;&lt;h1&gt;哈哈&lt;/h1&gt;&lt;h1&gt;哈哈&lt;/h1&gt;&lt;h1&gt;哈哈&lt;/h1&gt;&lt;h1&gt;哈哈&lt;/h1&gt;&lt;/body&gt;&lt;/html&gt;&#x27;,&#x27;html&#x27;,&#x27;utf-8&#x27;)
msg[&#x27;From&#x27;]=msg_format(user)
msg[&#x27;To&#x27;]=msg_format(to)
msg[&#x27;Subject&#x27;]=Header(mail_title,&#x27;utf-8&#x27;).encode(&#x27;utf-8&#x27;)

server=smtplib.SMTP_SSL(smtp_server,465) # SSL表示加密的连接。不加SSL则默认25端口
server.set_debuglevel(1)
server.login(user,passwd)
server.sendmail(user,[to],msg.as_string())
server.quit()
</code></pre>

## 带附件(图片)
<pre><code class="language-python line-numbers">from email.mime.text import MIMEText
from email import encoders
from email.header import Header
from email.utils import parseaddr,formataddr
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
import smtplib

def msg_format(s):
    name,addr=parseaddr(s)
    return formataddr((Header(name,&#x27;utf-8&#x27;).encode(&#x27;utf-8&#x27;),addr))

user=input(&#x27;Please enter your email: &#x27;)
passwd=input(&#x27;Please enter the password: &#x27;)
smtp_server=input(&#x27;Please enter your SMTP_Server address: &#x27;)
to=input(&#x27;Send the mail to: &#x27;)
mail_title=input(&#x27;mail title: &#x27;)
#mail_body=input(&#x27;mail body: &#x27;)

#msg=MIMEText(mail_body,&#x27;plain&#x27;,&#x27;utf-8&#x27;)
#msg=MIMEMultipart() # 只显示html版本
msg=MIMEMultipart(&#x27;alternative&#x27;) # 兼容不能渲染html的邮件客户端
msg.attach(MIMEText(&#x27;&lt;html&gt;&lt;body&gt;&lt;h1&gt;哈哈&lt;/h1&gt;&lt;h1&gt;哈哈&lt;/h1&gt;&lt;h1&gt;哈哈&lt;/h1&gt;&lt;h1&gt;哈哈&lt;/h1&gt;&lt;p&gt;&lt;img src =&quot;cid:0&quot;&gt;&lt;/p&gt;&lt;/body&gt;&lt;/html&gt;&#x27;,&#x27;html&#x27;,&#x27;utf-8&#x27;)) # cid:0表示将图片附件放在正文
with open(&#x27;./ipt.png&#x27;,&#x27;rb&#x27;) as f:
    mime=MIMEBase(&#x27;image&#x27;,&#x27;png&#x27;,filename=&#x27;ipt.png&#x27;) # 创建附件 类型为image png格式。附件名为ipt.png
    mime.add_header(&#x27;Content-Disposition&#x27;, &#x27;attachment&#x27;, filename=&#x27;ipt.png&#x27;)
    mime.add_header(&#x27;Content-ID&#x27;, &#x27;&lt;0&gt;&#x27;)
    mime.add_header(&#x27;X-Attachment-Id&#x27;, &#x27;0&#x27;)
    mime.set_payload(f.read()) # 读取本地附件
    encoders.encode_base64(mime) # base64编码
    msg.attach(mime) # 加入邮件中
msg[&#x27;From&#x27;]=msg_format(user)
msg[&#x27;To&#x27;]=msg_format(to)
msg[&#x27;Subject&#x27;]=Header(mail_title,&#x27;utf-8&#x27;).encode(&#x27;utf-8&#x27;)

server=smtplib.SMTP_SSL(smtp_server,465)
server.set_debuglevel(1)
server.login(user,passwd)
server.sendmail(user,[to],msg.as_string())
server.quit()
</code></pre>

## 邮件内容结构
<pre><code class="language-python line-numbers">Message
+- MIMEBase
   +- MIMEMultipart
   +- MIMENonMultipart
      +- MIMEMessage
      +- MIMEText
      +- MIMEImage
</code></pre>
