---
title: c语言 - 内存管理
date: 2017-07-17 16:40:00
categories:
- c
tags:
- c
keywords: c语言 内存管理
---

> 
c语言 - 内存管理

<!-- more -->

## 动态分配内存
头文件：`stdlib.h`
- `void *malloc(int size);`：在堆区分配一块指定大小的内存空间，这块内存空间在函数执行完成后不会被初始化，它们的值是未知的
- `void *calloc(int len, int size);`：在堆区分配一块len*size大小的内存空间，并且每个字节都初始化为0
- `void *realloc(void *ptr, int newsize);`：重新分配内存，把内存扩展到newsize
- `void free(void *ptr);`：释放指针ptr所指向的内存块

对于`void *realloc(void *ptr, int newsize);`：
- 指针ptr必须是在动态内存空间分配成功的指针
- 如果ptr为NULL，效果同malloc()
- 如果newsize为0，效果同free()
- 成功分配内存后ptr将被系统回收，一定不可再对ptr指针做任何操作，包括free()
- 相反的，可以对realloc()函数的返回值进行正常操作
- 如果是扩大内存操作会把ptr指向的内存中的数据复制到新地址(新地址也可能会和原地址相同，但依旧不能对原指针进行任何操作)
- 如果是缩小内存操作，原始据会被复制并截取新长度
- 分配成功返回新的内存地址，可能与ptr相同，也可能不同；失败则返回NULL
- 如果分配失败，ptr指向的内存不会被释放，它的内容也不会改变，依然可以正常使用

在定义数组的时候，我们必须事先定好数组的长度，并且不能再改变长度
而使用动态内存分配，可以做到所谓的动态数组，在运行时确定数组长度
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]){
    int *arr = (int *)malloc(sizeof(int) * (argc-1));
    for(int i=1; i<argc; i++){
        int value = atoi(argv[i]);
        arr[i-1] = value;
    }
    for(int i=0; i<argc-1; i++){
        printf("arr[%d] = %d\n", i, arr[i]);
    }
    free(arr);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [17:15:52]
$ gcc a.c

# root @ localhost in ~ [17:15:55]
$ ./a.out 1 2 3 4 5 6 7 8 9 10
arr[0] = 1
arr[1] = 2
arr[2] = 3
arr[3] = 4
arr[4] = 5
arr[5] = 6
arr[6] = 7
arr[7] = 8
arr[8] = 9
arr[9] = 10
</script></code></pre>

## memset()、memcpy()、memmove()、strcpy()
头文件：string.h
- `void *memset(void *buffer, int c, int len);`
把buffer所指内存区域的前len个字节设置成字符c
经常用来初始化基本数据类型、结构体等，因为比用for循环快多了
- `void *memcpy(void *dest, void *src, int len);`
由src所指内存区域复制len个字节到dest所指内存区域
src和dest所指内存区域不能重叠，函数返回指向dest的指针，可以拿它拷贝任何数据类型的对象
- `void *memmove(void *dest, void *src, int len);`
它与memcpy的功能相似，都是将src所指的len个字节复制到dest所指的内存地址的起始位置
不同的是它处理了src和dest有重叠的情况，当目标区域与源区域没有重叠则和memcpy函数功能相同
- `char *strcpy(char *dest, char *src);`
把src所指由NULL结束的字符串复制到dest所指的数组中
src和dest所指内存区域不可以重叠且dest必须有足够的空间来容纳src的字符串，返回指向dest的指针

## 迷途指针、野指针
[什么是迷途指针、什么是野指针、它们都有什么危害？](https://www.zfl9.com/c-qa.html#野指针-迷途指针)

## 内存溢出、内存泄漏、内存越界、栈溢出
[内存溢出、内存泄漏、内存越界、栈溢出等等都有什么区别？该如何避免？](https://www.zfl9.com/c-qa.html#内存溢出-内存泄漏-内存越界-缓冲区溢出-栈溢出)

## 一个关于字符数组和字符指针的小问题
定义一个字符串可以用一个字符数组：如字符串"Google"可以表示为`char str[7] = {'G', 'o', 'o', 'g', 'l', 'e'};`
但是这样也太麻烦了，所以c语言允许这样简写：`char str[] = {"Google"};`或者`char str[] = "Google";`

但是只有在初始化的时候才能这么干，如果在赋值的时候，只能一个一个字符赋值！

因为在赋值的时候，对于`str = "Google";`，"Google"实际是一个`char *`类型的字符串常量，存储在代码区，而str则是`char [7]`类型的变量，所以编译报错

而对于一个字符指针，有这么几种赋值方式：
`char *str = "www.zfl9.com";`
对于"www.zfl9.com"，这是一个字符串常量，存储在代码区，因为是常量，所以不能通过指针修改它的值！

`char *str = (char *)malloc(sizeof(char) * 50);`
`strcpy(str, "www.zfl9.com");`
第一个语句，申请了50个字节长的堆空间，第二个语句，将存储在代码区的"www.zfl9.com"字符串常量拷贝到刚才申请到的堆内存中，也就是内存的复制

`char *str = (char *)malloc(sizeof(char) * 50);`
`str = "www.zfl9.com";`
第一个语句，申请了50个字节长的堆空间，第二个语句，改变这个指针的指向，指向存储在代码区的"www.zfl9.com"
这种方式，会直接把刚才申请的堆内存的地址给弄丢了！搞得不好就会内存溢出，因为那50个字节的内存找不回了！
