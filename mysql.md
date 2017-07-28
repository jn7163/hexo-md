---
title: mysql简明笔记
date: 2016-10-15 16:31:12
categories:
- 数据库
- mysql
tags:
- 数据库
- mysql
keywords: mysql笔记, mysql命令
---
> Mysql是最流行的关系型数据库管理系统，在WEB应用方面MySQL是最好的RDBMS(Relational Database Management System：关系数据库管理系统)应用软件之一。Linux运维需要熟悉常用的MySQL操作-增删改查

<!-- more -->

## 连接mysql
- 本地连接localhost
<pre><code class="language-sql line-numbers">mysql -uroot -p123456   # 用户名root，密码123456
mysql -p123456          # 当前shell登录用户为root时，可省略-u选项, 默认为root用户
</code></pre>

- 远程连接remotehost
<pre><code class="language-sql line-numbers">mysql -hmysql.org -uroot -p123456   # 目标主机mysql.org 用户名root，密码123456
</code></pre>

- 退出mysql交互界面
<pre><code class="language-sql line-numbers">exit
ctrl+c
ctrl+d
</code></pre>

## 修改密码
<pre><code class="language-sql line-numbers">mysqladmin -uroot -p123456 password abc123  # password后接新密码
</code></pre>

## 查看数据库
<pre><code class="language-sql line-numbers">show databases; # mysql语句以;结尾
</code></pre>

## 选择数据库
<pre><code class="language-sql line-numbers">use mysql;
</code></pre>

## 查看所有表
<pre><code class="language-sql line-numbers">use mysql;
show tables;
</code></pre>

## 查看表结构
<pre><code class="language-sql line-numbers">use mysql;
describe 表名;
</code></pre>

## 创建数据库
<pre><code class="language-sql line-numbers">create database 库名;
</code></pre>

## 创建表
<pre><code class="language-sql line-numbers">use 库名;
create table 表名 (字段设定列表);
</code></pre>

## 删除数据库
<pre><code class="language-sql line-numbers">drop database 库名;
</code></pre>

## 删除表
<pre><code class="language-sql line-numbers">use 库名;
drop table 表名;
</code></pre>

## 删除表记录
<pre><code class="language-sql line-numbers">use 库名;
delete from 表名 where ...;
</code></pre>

## 查看表记录
<pre><code class="language-sql line-numbers">use 库名;
select * from 表名;
</code></pre>

## 调用sql文件
直接在mysql交互界面有时候可能不方便，我们可以将mysql语句写入一个文本文件里面。然后用mysql调用
<pre><code class="language-sql line-numbers">mysql -uroot -p123456 < mysql_file.sql  # mysql_file.sql假定为我们保存的sql语句文件
</code></pre>

## 综合
<pre><code class="language-sql line-numbers">## 注释行
单行注释 # ...
多行注释 /* ... */

## 创建数据表
create table TABLE (
id int not null auto_increment,
name varchar(50) not null,
email varchar(50) not null,
date date,
primary key (id)
);

auto_increment  # 主键自增
now()           # 获取当前时间的mysql函数

## insert select update delete 语句
insert into TABLE (field1,field2,field3) values (value1,value2,value3);
select field_name,field_name from TABLE where name='otokaze';
# 可用and or逻辑符
# like %表示任意字符
# regexp 正则匹配
select * from TABLE where id=1;
update TABLE set name='zfl' where id=1;
delete from TABLE where id=1;

## mysql limit 列出指定行的记录
select * from test limit 5,10;  # 返回第6-15行
select * from test limit 5;     # 返回前5行
select * from test limit 10,-1; # 返回第11-last行

## order by 排序
select * from zfl.test order by name asc|desc limit 5;

## alter 修改
alter table test drop i;                    # 删除i字段
alter table test add i int;                 # 追加i字段
alter table test add i int first|after c;
alter table test modify c char(10);         # 修改字段类型
alter table test modify rename to NewName;  # 重命名表

## mysql grant|revoke 授权管理 revoke和grant用法几乎一致，只是将to改为from
grant all privileges on zfl.* to 'otokaze'@'%' identified by '123456';
grant all on zfl.* to 'zfl'@'192.168.255.%' identified by '123456' with grant option; # 可授权给其他人
revoke all on zfl.* from 'zfl'@'%';
delete from mysql.user where user='zfl';

flush privileges;

## 其他
show columns from mysql.user;           # 查看表属性等信息
show index from mysql.user;             # 查看索引信息
show table status from mysql;           # 查看mysql数据库的表信息
show table status from mysql like 'user'\G;     # 查看mysql.user 表的信息 \G表示结果按列打印
show create table mysql.user\G;         # 查看完整表信息
insert into zfl.test (...) select ... from 源_Table;    # 复制表
select version()        # mysql版本信息
select database()       # 当前数据库名
select user()           # 当前用户名
show status             # 服务器状态
show variables          # 服务器配置变量

## mysql数据类型
### 数值
tinyint 0-255           小整数
smallint 0-65535        大整数
mediumint 0-16777215    大整数
int 0-4294967295        大整数
bigint                  极大整数
float                   单精度浮点数
double                  双精度浮点数

### 日期
date YYYY-MM-DD                 日期
time HH:MM:SS                   时间
year YYYY                       年份
datetime YYYY-MM-DD HH:MM:SS
timestamp YYYYMMDD HHMMSS       时间戳

### 字符串
char 0-255 bytes        定长字符
varchar 0-65535         变长字符 innodb建议使用varchar
blob 0-65535            二进制形式长文本
text 0-65535            长文本
tinyblob 0-255
tinytext 0-255
medium~
long~

## 导入导出数据
mysqldump -uroot -p123456 -A > all.sql
mysqldump -uroot -p123456 zabbix > zabbix.sql
mysqldump -uroot -p123456 zfl test > zfl_test.sql

mysql -uroot -p123456 abc < test.sql
</code></pre>

## mysql编码`utf8`和`utf8mb4`
<pre><code class="language-sql line-numbers">## 查看mysql编码
show variables like '%char%';
utf8mb4兼容utf8，支持表情符号等特性

## my.cnf utf8mb4编码设置
[client]
default-character-set = utf8mb4
[mysql]
default-character-set = utf8mb4
[mysqld]
init-connect = 'SET NAMES utf8mb4'
collation_server = utf8mb4_unicode_ci
character_set_server = utf8mb4
skip-character-set-client-handshake

## my.cnf utf8编码设置
### mariadb：
--- my.cnf.d/server.cnf ---
[mysqld]
character_set_server=utf8


### mysql:
---- my.cnf ----
#在[client]段增加下面代码
default-character-set=utf8

#在[mysqld]段增加下面的代码
character-set-server=utf8
collation-server=utf8_general_ci
</code></pre>
