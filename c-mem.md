---
title: c语言 - 程序内存分布
date: 2017-07-01 18:08:00
categories:
- c
tags:
- c
keywords: c语言程序内存分布
---

> 
c语言 - 程序内存分布，了解一个c程序在运行过程中，各个部分的内存是怎么分布的，都有什么特点

<!-- more -->

### 内存分布
![C语言程序内存分布图](/images/c_mem.jpg)

**代码区(code area)**
- 存放：函数体的`二进制代码`、字面量(`整数常量`、`浮点常量`、`字符常量`、`字符串字面值`、`枚举常量`)
- 进程中的所有线程共享该区域

**数据区(data area)**
- 又称`静态数据区`、`全局数据区`; 存放：`static存储类的变量`(全局变量和静态变量)
- 细分为`初始化`、`未初始化或初始化为0`的两块区域
- `程序结束后由操作系统释放`
- 进程中的所有线程共享该区域

**堆区(heap area)**
- `malloc/new`申请的内存存放区域
- `由程序员分配和释放`，若程序员不释放，`程序运行结束时由操作系统回收`
- 进程中的所有线程共享该区域

**栈区(stack area)**
- 存放`函数的参数`、`函数局部变量`等
- `由系统自动分配释放`
- 进程中的各个线程拥有各自的栈区

**命令行参数区**
- 存放`命令行参数`、`Shell环境变量`等
- 如：向main()函数传参

### 实例说明
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int i1; // bss区域
int i2 = 0; // bss区域
char ch = 'A';  // data区域
char *str1 = "www.zfl9.com";    // "www.zfl9.com\0"在text区域; str1在data区域
char *str2 = "www.zfl9.com";    // "www.zfl9.com\0"与str1指向的字符串是同一份拷贝; str2在data区域

int main(void){
    static int st_i;    // bss区域
    static char st_c = 'X'; // data区域
    int au_i;   // stack区域
    char *str3 = (char *)malloc(sizeof(char) * 20); // malloc申请的内存在heap区域; str3在stack区域
    char *str4 = "www.google.com";  // "www.google.com\0"在text区域; str4在stack区域
    strcpy(str3, "www.google.com"); // "www.google.com\0"和str4指向的字符串是同一份拷贝
    free(str3);
    return 0;
}
</script></code></pre>
