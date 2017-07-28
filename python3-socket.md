---
title: python3 tcp/udp编程
date: 2016-12-09 12:24:00
categories:
- python
tags:
- python
keywords: python, python3, tcp/udp编程
---
> 
python3 tcp/udp编程
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

> Socket是网络编程的一个抽象概念。通常我们用一个Socket表示“打开了一个网络链接”，而打开一个Socket需要知道目标计算机的IP地址和端口号，再指定协议类型即可

## socket连接
<pre><code class="language-python line-numbers">import socket

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(('b.zfl9.com',80))
s.send(b'GET / HTTP/1.1\r\nHost: b.zfl9.com\r\nConnection: close\r\n\r\n')

L=[]
while True:
    d=s.recv(1024)
    if d:
        L.append(d)
    else:
        break

data=b''.join(L)
print(data.decode('utf-8'))
s.close()

header,html=data.split(b'\r\n\r\n',1)
print('--------header--------')
print(header.decode('utf-8'))
print('--------header--------')
print('---------html---------')
print(html.decode('utf-8'))
print('---------html---------')
with open('b_index.html','wb') as f:
    f.write(html)
</code></pre>

## tcp

### 服务器
<pre><code class="language-python line-numbers">import socket
import time
import threading

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.bind(('127.0.0.1',9999))
s.listen(5)

def tcplink(sock,addr):
    print('Accept new connection from %s:%s'%addr)
    sock.send(b'Welcome!')
    while True:
        data=sock.recv(1024)
        time.sleep(1)
        if not data or data.decode('utf-8') == 'exit':
            break
        sock.send(('Hello, %s!'%data.decode('utf-8')).encode('utf-8'))
    sock.close()
    print('Connection %s:%s is close...'%addr)

while True:
    sock,addr=s.accept()
    t=threading.Thread(target=tcplink,args=(sock,addr))
    t.start()
</code></pre>

### 客户端
<pre><code class="language-python line-numbers"><script type="text/plain">import socket

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(('127.0.0.1',9999))

print(s.recv(1024).decode('utf-8'))
n=0
while n<20:
    n=n+1
    for data in [b'otokaze.top',b'zflyun.com',b'zfl9.com']:
        s.send(data)
        print(s.recv(1024).decode('utf-8'))
</script></code></pre>

## udp

### 服务器
<pre><code class="language-python line-numbers">import socket

s=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
s.bind(('127.0.0.1',9999))

print('-----Listen udp: 9999 ------')
while True:
    data,addr=s.recvfrom(1024)
    print('Welcome %s:%s' % addr)
    s.sendto(b'Hello %s' % data,addr)
</code></pre>

### 客户端
<pre><code class="language-python line-numbers">import socket

s=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)

for data in [b'otokaze.top',b'zflyun.com',b'zfl9.com']:
    s.sendto(data,('127.0.0.1',9999))
    print(s.recv(1024).decode('utf-8'))

s.close()
</code></pre>
