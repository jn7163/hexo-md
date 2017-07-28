---
title: python3 进程与线程
date: 2016-12-09 23:59:00
categories:
- python
tags:
- python
keywords: python3多线程多进程
---

> 
python3 多进程/多线程
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

### 多进程
<pre><code class="language-python line-numbers">#!/bin/env python3
# -*- coding: utf-8 -*-

import multiprocessing,os,time,random

def task(name):
    print('start %s - PID=%d' % (name,os.getpid()))
    start=time.time()
    time.sleep(random.random() * 5)
    end=time.time()
    print('end %s - run_time=%0.2f' % (name,(end-start)))

if __name__ == '__main__':
    print('Parent process start %d....' % os.getpid())
    p=multiprocessing.Pool(4)
    for i in range(6):
        p.apply_async(task,args=(i,))
        print('wait for all sub_proc done...')
        p.close()
    p.join()
print('all sub_proc done...')
</code></pre>

### 多线程
<pre><code class="language-python line-numbers">#!/bin/env python3
# -*- coding: utf-8 -*-

import threading,time

def func(max_num):
    for i in range(max_num):
        print(i)
        time.sleep(1)

#for x in range(10000):
    #t=threading.Thread(target=func,args=(10,))
    #t.start()

threads=[]

for x in range(100):
    t=threading.Thread(target=func,args=(10,))
    threads.append(t)

for thr in threads:
    thr.start()

for thr in threads:
    if thr.isAlive():
        thr.join()
</code></pre>

> 
由于python gil锁的问题，多进程是可以利用多核的，多线程只能利用单核
