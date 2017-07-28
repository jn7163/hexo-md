---
title: c语言 - 断言
date: 2017-07-18 11:30:00
categories:
- c
tags:
- c
keywords: C语言 断言 assert
---

> 
c语言 - 断言

<!-- more -->

## assert断言
断言是对某种假设条件进行检查(可理解为若条件成立则无动作，否则应报告)
它可以快速发现并定位软件问题，同时对系统错误进行自动报警
断言可以对在系统中隐藏很深，用其它手段极难发现的问题进行定位，从而缩短软件问题定位时间，提高系统的可测性

使用断言需要包含头文件: `assert.h`

断言的宏原型为：`void assert(expression);`

expression 是一个表达式，表达式为真是程序员期待的正确情况；
表达式为假时，表明此时程序所处的状态违背了我们期望的状态，这时assert会调用系统提供的结束函数(abort或terminate)结束程序
在结束程序之前，assert会向stderr输出相关的出错信息，供程序分析

断言的一个简单例子：在函数入口处检查参数的合法性
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <assert.h>

long fsize(FILE *fp){
    assert(fp != NULL);
    long size;
    fpos_t fpos;
    fgetpos(fp, &fpos);
    fseek(fp, 0, 2);
    size = ftell(fp);
    fsetpos(fp, &fpos);
    return size;
}

int main(void){
    FILE *fp = fopen("unexist.file", "rb");
    long size = fsize(fp);
    printf("file size: %ld bytes\n", size);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [12:00:28]
$ gcc a.c

# root @ localhost in ~ [12:00:44]
$ ./a.out
a.out: a.c:5: fsize: Assertion `fp != ((void *)0)' failed.
[1]    12955 abort      ./a.out
</script></code></pre>

assert在运行时会检测传入的参数fp是否是空指针，如果是空指针则结束程序，并打印相关信息

注意：断言的引入，会占用代码的空间，同时需要占用cpu的时间来对断言进行处理，频繁的断言会对产品代码运行的效率产生影响
所以，断言只是在开发阶段使用，方便发现问题，及时纠正，而在产品代码中应该关闭断言
关闭断言很简单，只需在`#include <assert.h>`之前定义宏`#define NDEBUG`就可以了
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#ifndef NDEBUG
#define NDEBUG
#endif
#include <assert.h>

long fsize(FILE *fp){
    assert(fp != NULL);
    long size;
    fpos_t fpos;
    fgetpos(fp, &fpos);
    fseek(fp, 0, 2);
    size = ftell(fp);
    fsetpos(fp, &fpos);
    return size;
}

int main(void){
    FILE *fp = fopen("unexist.file", "rb");
    long size = fsize(fp);
    printf("file size: %ld bytes\n", size);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [12:07:18]
$ gcc a.c

# root @ localhost in ~ [12:07:19]
$ ./a.out
[1]    13019 segmentation fault  ./a.out
</script></code></pre>

注意，这个段错误不是断言引发的，而是传入了一个空指针导致的

## 断言的用途及注意事项
- 在函数开始处检验传入的参数的合法性
- 断言可以用于判断函数执行中的中间结果
- 每个assert应只检验一个条件，因为同时检验多个条件时，如果断言失败，无法直观判断是哪个条件失败
- 不能使用改变环境的语句，因为assert只在DEBUG中生效，如果这么做，会使用程序在真正运行时遇到问题
- 断言和后面的语句之间应该空一行，以形成视觉和逻辑上一致感
- 不能滥用断言和if条件判断
- 断言用于判断代码设计的缺陷(bug)
- 断言只能用于开发版本中，在产品代码中应该关闭断言宏
- 在实际项目中，一旦断言失败，应该立即查找错误原因，并解决问题，而不是把断言语句删掉
