---
title: expect自动交互
date: 2016-12-07 18:51:00
categories:
- shell
tags:
- shell
- expect自动交互
keywords: expect, expect自动交互, shell
---
> 在实际工作中，我们运行命令、脚本或程序时，这些命令、脚本或程序都需要从终端输入某些继续运行的指令，而这些输入都需要人为的手工进行。而利用expect，则可以根据程序的提示，模拟标准输入提供给程序，从而实现自动化交互执行。这就是expect

<!-- more -->

## 安装expect
<pre><code class="language-bash line-numbers">yum -y install expect
</code></pre>

## 实例1
<pre><code class="language-bash line-numbers">#!/usr/bin/expect

set timeout 30 # 设置超时
set host "192.168.255.104" # 定义变量
set user "root" # 定义变量
set passwd "123456" # 定义变量

spawn ssh $user@$host # 启动需要运行的进程

expect {
    "*password*" {
        send "$passwd\r" # 匹配到password则输入passwd变量的值
    }
    "*connecting*" {
        send "yes\r" # 匹配到connecting则输入yes
        exp_continue # 继续匹配
        "*password*" {
            send "$passwd\r" # 匹配到password则输入passwd变量的值
        }
    }
}

interact # 保持交互状态
</code></pre>

## 实例2 - 传递参数
<pre><code class="language-bash line-numbers">#!/usr/bin/expect

if {$argc < 3} { # argc表示参数个数
    puts &quot;Usage:cmd &lt;host&gt; &lt;username&gt; &lt;password&gt;&quot;
    exit 1
}

set timeout -1 # 永不超时
set host [lindex $argv 0] # 第一个参数
set user [lindex $argv 1] # 第二个参数
set passwd [lindex $argv 2] # 第三个参数

spawn ssh $user@$host

expect {
    "*password*" {
        send "$passwd\r"
    }
    "*connecting*" {
        send "yes\r"
        exp_continue
        "*password*" {
            send "$passwd\r"
        }
    }
}

interact
</code></pre>

## 实例3 - eof
<pre><code class="language-bash line-numbers">#!/usr/bin/expect

if {$argc < 4} {
    puts &quot;Usage:cmd &lt;file&gt; &lt;host&gt; &lt;username&gt; &lt;password&gt;&quot;
    exit 1
}

set timeout -1
set file [lindex $argv 0]
set host [lindex $argv 1]
set user [lindex $argv 2] 
set passwd [lindex $argv 3]

spawn scp $file $user@$host:

expect {
    "*password*" {
        send "$passwd\r"
    }
    "*connecting*" {
        send "yes\r"
        exp_continue
        "*password*" {
            send "$passwd\r"
        }
    }
}

expect eof # 未加入此行会导致spawn启动的进程无法顺利执行！
</code></pre>

## awk浮点数计算
<pre><code class="language-bash line-numbers">awk 'BEGIN{print 5/4}'
</code></pre>

## shell for 循环用法二
<pre><code class="language-bash line-numbers"><script type="text/plain"># root @ www in ~ [16:45:02]
$ for ((i=0; i<10; i++)); do echo $i; done
0
1
2
3
4
5
6
7
8
9
</script></code></pre>

## shell 四则运算
<pre><code class="language-bash line-numbers">## 整数运算 ##

1. let 几乎支持所有运算符(变量前面无需加$)

let a+b
let a-b
let a*b
let a**b
let a/b
let a%b

let a+=b
let a-=b
let a*=b
let a/=b
let a%=b

let a++
let a--


2. (()) 用法基本同let

注: 赋值时需添加 $ 符号
eg: b=$((a++))


3. $[] 用法基本同let


4. expr

运算符和数之间必须要有空格！
操作符: |、&、<、<=、=、!=、>=、>、+、-、*、/、% 

## 浮点数运算 ##

bc

echo 'scale=3;4/3' | bc

scale 指定 保留的小数位
</code></pre>
