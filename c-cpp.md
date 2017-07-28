---
title: c语言 - 预处理指令
date: 2017-07-09 19:23:00
categories:
- c
tags:
- c
keywords: C语言 预处理指令
---

> 
c语言 - 预处理指令

<!-- more -->

## 文件包含
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include "userdef.h"
</script></code></pre>

主要用来包含头文件，include 指令的处理很简单，就是把头文件插入到源文件，之后就不需要头文件了，因为头文件的内容都包含进来了
一个 include 命令只能包含一个文件，可以有多个 include 指令，允许嵌套包含
`<>`形式一般用于标准库头文件，`""`一般用于自定义头文件
`<>`的头文件搜索路径不包含当前路径，而`""`则包含当前路径
所以，可以都用`""`形式来包含头文件，包括系统的和自定义的，但是这不符合规范，标准库的就用`<>`，其他的就用`""`

我们来查看一下预处理的过程
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    printf("hello, world!\n");
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [19:36:07]
$ gcc -E a.c -o a.i

# root @ localhost in ~ [19:36:17]
$ tail a.i

extern void funlockfile (FILE *__stream) __attribute__ ((__nothrow__ , __leaf__));
# 943 "/usr/include/stdio.h" 3 4

# 2 "a.c" 2

int main(){
    printf("hello, world!\n");
    return 0;
}
</script></code></pre>

## 宏定义
允许用一个标识符来表示一个字符串
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#define PI 3.14

int main(){
    printf("%.2f\n", PI);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [19:38:33]
$ gcc a.c

# root @ localhost in ~ [19:38:36]
$ ./a.out
3.14
</script></code></pre>

`#define PI 3.14`是一个宏定义，`PI`是一个宏名(宏名是标识符的一种，所以要遵循标识符的命名规则)，`3.14`是宏的内容
在预处理时，所有遇到`PI`的地方，都会被替换为`3.14`，除了引号内的内容不会被替换，这个过程叫做`宏展开`

宏定义的作用域：从宏定义开始到源程序结束，除非用`#undef`来终止
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#define PI 3.14

int main(){
#undef PI
    printf("%.2f\n", PI);
    return 0;
}
</script></code></pre>

这样，在源文件第5行之后，这个宏定义就消失了

宏定义允许嵌套
<pre><code class="language-c line-numbers"><script type="text/plain">#define PI 3.14
#define S PI*10*10
</script></code></pre>

S展开后为`3.14*10*10`

宏名一般用大写，不过也可以用小写
可以用宏表示数据类型，书写方便
`#define UINT unsigned int`
`UINT a, b;`

注意和`typedef`的区别：
宏定义只是简单的字符串代换，是在预处理完成的，而typedef是在编译时处理的，它不是作简单的代换，而是对类型说明符重新命名，被命名的标识符具有类型定义说明的功能
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

#define PTR1 int *
typedef int * PTR2;

int main(){
    int a = 10, b = 20;
    PTR1 p1, p2;
    p1 = &a, p2 = &b;
    PTR2 p3, p4;
    p3 = &a, p4 = &b;
    printf("%d, %d, %d, %d\n", *p1, *p2, *p3, *p4);
    return 0;
}
</script></code></pre>

这段代码编译会报错，因为，只有 p1、p3、p4 是指针变量，而 p2 只是一个 int 类型的变量

## 带参数的宏定义
定义：`#define 宏名(形参列表) 字符串`
使用：`宏名(实参列表)`

例如：
<pre><code class="language-c line-numbers"><script type="text/plain">#define POW(x) x*x
int a = POW(10);
</script></code></pre>

展开后会替换为`int a = 10*10;`

对于带参宏定义不仅要在参数两侧加括号，还应该在整个字符串外加括号，防止发生错误

如果一个宏一行写不下，可以用 "\" 连接，如：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#define STR \
    "https://www.zfl9.com/iptables.html?userid=d89sd9f2j398ds2&auth=true"

int main(){
    printf("%s\n", STR);
    return 0;
}
</script></code></pre>

在宏定义中，有时还会用到`#`、`##`两个符号，它们能对宏参数进行操作
- `#`的用法
用来将宏参数转换为字符串，也就是在宏参数前后添加双引号
`#define STRING(str) #str`
`STRING(www.zfl9.com)`会替换为`"www.zfl9.com"`
`STRING("www.zfl9.com")`会替换为`"\"www.zfl9.com\""`
- `##`的用法
`##`称为连接符，将宏参数和其他的串拼接起来
`#define CAT(a, b) a##b`
`CAT(12, 34)`会替换为`1234`

## 预定义的宏
- `__DATE__`：当前日期，一个以 "MMM DD YYYY" 格式表示的字符常量
- `__TIME__`：当前日期，一个以 "MMM DD YYYY" 格式表示的字符常量
- `__FILE__`：当前文件名，一个字符串常量
- `__LINE__`：当前行号，一个十进制常量
- `__STDC__`：当编译器以 ANSI 标准编译时，则定义为 1
- `__cplusplus`：当编写 c++ 程序时，该标识符被定义

## 条件编译
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    #if _WIN32
        printf("Windows\n");
    #elif __linux__
        printf("Linux\n");
    #endif
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [21:17:38]
$ gcc a.c

# root @ localhost in ~ [21:17:39]
$ ./a.out
Linux
</script></code></pre>

这段代码的意思是，如果宏`_WIN32`的值为真，则保留第 5 行的代码，如果宏`__linux__`的值为真，则保留第 7 行的代码
注意，判断条件为`整型常量表达式`，也就是说表达式中不能包含变量，而且结果必须是整数

**`#if`**
<pre><code class="language-c line-numbers"><script type="text/plain">#if 整型常量表达式1
    程序段1
#elif 整型常量表达式2
    程序段2
#elif 整型常量表达式3
    程序段3
#else
    程序段4
#endif
</script></code></pre>

**`#ifdef`**，表示如果当前宏名已经定义，则保留程序段1，否则保留程序段2
<pre><code class="language-c line-numbers"><script type="text/plain">#ifdef  宏名
    程序段1
#else
    程序段2
#endif
</script></code></pre>

**`#ifndef`**，和`#ifdef`正好相反
<pre><code class="language-c line-numbers"><script type="text/plain">#ifndef 宏名
    程序段1 
#else 
    程序段2 
#endif
</script></code></pre>

例子：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#define N1 10
#define N2 20

int main(){
    #if N1 == 10 || N2 == 10
        printf("N1: %d\n", N1);
    #else
        printf("error!\n");
    #endif
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [21:29:57]
$ gcc a.c

# root @ localhost in ~ [21:29:58]
$ ./a.out
N1: 10
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#define N1
#define N2

int main(){
    #if defined N1 && defined N2
        printf("True!\n");
    #else
        printf("False!\n");
    #endif
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [21:31:11]
$ gcc a.c

# root @ localhost in ~ [21:31:13]
$ ./a.out
True!
</script></code></pre>

## `#error`指令，阻止编译
`#error error_msg`
如，我们的程序只支持 Windows，不兼容 Linux
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    #ifndef _WIN32
        #error 抱歉，请使用 Windows 系统，该程序暂时不支持其他平台
    #endif
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [21:37:44]
$ gcc a.c
a.c: In function ‘main’:
a.c:5:10: error: #error 抱歉，请使用 Windows 系统，该程序暂时不支持其他平台
         #error 抱歉，请使用 Windows 系统，该程序暂时不支持其他平台
          ^
</script></code></pre>

## 预处理指令小结
预处理指令是以`#`号开头的代码行，`#`号必须是该行除了任何空白字符外的第一个字符
`#`后是指令关键字，在关键字和`#`号之间允许存在任意个数的空白字符，整行语句构成了一条预处理指令
该指令将在编译器进行编译之前对源代码做某些转换
