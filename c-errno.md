---
title: c语言 - 错误处理
date: 2017-07-17 16:17:00
categories:
- c
tags:
- c
keywords: C语言 错误处理 errno
---

> 
c语言 - 错误处理，c语言不提供对错误处理的直接支持，但是作为一种系统编程语言，它以返回值的形式允许您访问底层数据，在发生错误时，大多数的 C 或 UNIX 函数调用返回 1 或 NULL，同时会设置一个错误代码 errno，该错误代码是全局变量，表示在函数调用期间发生了错误

<!-- more -->

## errno、perror()和strerror()
- `errno`：头文件`errno.h`定义的一个全局变量，记录了函数调用发生错误时的错误码，int类型，0表示没有错误
- `void perror(char *msg)`：头文件`stdio.h`中定义的函数，用于抛出最近的一次系统错误信息
- `char *strerror(int errnum)`：头文件`string.h`中定义的函数，用来依参数errnum的错误代码来查询其错误原因的描述字符串, 然后将该字符串指针返回

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <string.h>
#include <errno.h>

int main(void){
    FILE *fp = fopen("unexist.file", "rb");
    if(fp == NULL){
        fprintf(stderr, "error code: %d\n", errno);
        fprintf(stderr, "error code detail: %s\n", strerror(errno));
        perror("error detail");
    }else{
        fclose(fp);
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [16:33:44]
$ gcc a.c

# root @ localhost in ~ [16:33:45]
$ ./a.out
error code: 2
error code detail: No such file or directory
error detail: No such file or directory
</script></code></pre>

## 程序退出状态
通常情况下，程序成功执行完一个操作正常退出的时候会带有值 `EXIT_SUCCESS`，在这里，`EXIT_SUCCESS` 是宏，它被定义为 0
如果程序中存在一种错误情况，当您退出程序时，会带有状态值 `EXIT_FAILURE`，被定义为 -1
