---
title: python3 mysql数据库操作
date: 2016-12-09 12:44:00
categories:
- python
tags:
- python
keywords: python, python3, mysql数据库操作
---
> 
python3 mysql数据库操作
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

## 安装 mysql/mariadb 驱动
<pre><code class="language-bash line-numbers">pip install mysql-connector
</code></pre>

## 设置mariadb编码为utf-8
<pre><code class="language-python line-numbers">mysql -uroot -p123456
use mysql;
show variables like '%char%';
</code></pre>

## 操作test数据库的user表
<pre><code class="language-python line-numbers">>>> import mysql.connector
### 连接数据库
>>> conn = mysql.connector.connect(user='root', password='123456', database='test')
### 常见user数据表
>>> cursor = conn.cursor()
>>> cursor.execute('create table user (id varchar(20) primary key, name varchar(20))')
>>> cursor.execute('insert into user (id, name) values (%s, %s)', ['1', 'Otokaze'])
>>> cursor.rowcount
>>> conn.commit()
>>> cursor.close()
## 查询数据表
>>> cursor = conn.cursor()
>>> cursor.execute('select * from user where id = %s', ('1',))
>>> values = cursor.fetchall()
>>> values
### 关闭Cursor和Connection:
>>> cursor.close()
>>> conn.close()
</code></pre>
