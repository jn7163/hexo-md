---
title: c语言 - printf()及scanf()系列函数
date: 2017-07-18 06:47:00
categories:
- c
tags:
- c
keywords: C语言 printf() scanf() 系列函数
---

> 
c语言 - printf()及scanf()系列函数
printf()、fprintf()、sprintf()、snprintf()、vprintf()、vfprintf()、vsprintf()、vsnprintf();
scanf()、fscanf()、sscanf()、vscanf()、vfscanf()、vsscanf()

<!-- more -->

## printf()系列
头文件：stdio.h、stdarg.h
- `int printf(const char *fmt, ...);`：输出至stdout
- `int fprintf(FILE *stream, const char *fmt, ...);`：输出至stream
- `int sprintf(char *str, const char *fmt, ...);`：输出至str
- `int snprintf(char *str, size_t size, const char *fmt, ...);`：输出至str，写入至多size个字符(包括'\0'字符)
- `int vprintf(const char *fmt, va_list ap);`：fmt从变参列表ap获取数据，然后输出至stdout
- `int vfprintf(FILE *stream, const char *fmt, va_list ap);`：fmt从变参列表ap获取数据，然后输出至stream
- `int vsprintf(char *str, const char *fmt, va_list ap);`：fmt从变参列表ap获取数据，然后输出至str
- `int vsnprintf(char *str, size_t size, const char *fmt, va_list ap);`：fmt从变参列表ap获取数据，然后输出至str，写入至多size个字符(包括'\0'字符)

返回值：正常情况下，返回成功写入的字符数；若出错，则返回负值；

比如：我们可以利用vprintf()写一个自己的格式化输出函数，来模拟printf()
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdarg.h>

int my_printf(const char *fmt, ...){
    va_list ap;
    va_start(ap, fmt);
    int writec = vprintf(fmt, ap);
    va_end(ap);
    return writec;
}

int main(void){
    my_printf("name: %s, age: %d, score: %.1f\n", "Otokaze", 18, 115.5);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [7:55:44]
$ gcc a.c

# root @ localhost in ~ [7:55:46]
$ ./a.out
name: Otokaze, age: 18, score: 115.5
</script></code></pre>

## scanf()系列
头文件：stdio.h、stdarg.h
- `int scanf(const char *fmt, ...);`：从stdin获取源数据，然后根据fmt格式化，赋值给变参列表
- `int fscanf(FILE *stream, const char *fmt, ...);`：从stream获取源数据，然后根据fmt格式化，赋值给变参列表
- `int sscanf(const char *str, const char *fmt, ...);`：从str获取源数据，然后根据fmt格式化，赋值给变参列表
- `int vscanf(const char *fmt, va_list ap);`：从stdin获取源数据，然后根据fmt格式化，赋值给ap
- `int vfscanf(FILE *stream, const char *fmt, va_list ap);`：从stream获取源数据，然后根据fmt格式化，赋值给ap
- `int vsscanf(const char *str, const char *fmt, va_list ap);`：从str获取源数据，然后根据fmt格式化，赋值给ap

返回值：正常情况下，返回成功赋值的变量数目；若出错，则返回负值；

例如：利用vprintf()和vscanf()模拟printf()和scanf()函数：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdarg.h>

int my_printf(const char *fmt, ...){
    va_list ap;
    va_start(ap, fmt);
    int writec = vprintf(fmt, ap);
    va_end(ap);
    return writec;
}

int my_scanf(const char *fmt, ...){
    va_list ap;
    va_start(ap, fmt);
    int readc = vscanf(fmt, ap);
    va_end(ap);
    return readc;
}

int main(void){
    char name[20];
    int age;
    float score;
    my_printf("input student's info: ");
    my_scanf("%s %d %f", name, &age, &score);
    my_printf("name: %s, age: %d, score: %.1f\n", name, age, score);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [8:15:19] C:130
$ gcc a.c

# root @ localhost in ~ [8:15:21]
$ ./a.out
input student's info: Otokaze 18 114.5
name: Otokaze, age: 18, score: 114.5
</script></code></pre>
