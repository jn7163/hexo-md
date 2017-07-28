---
title: c语言 - 缓冲区详解
date: 2017-07-07 18:29:00
categories:
- c
tags:
- c
keywords: c语言缓冲区详解 刷新缓冲区
---

> 
c语言 - 缓冲区详解，`缓冲区`(Buffer)又称为`缓存`(Cache)，是内存空间的一部分
也就是说，在内存中预留了一定的存储空间，用来暂时保存输入或输出的数据，这部分预留的空间就叫做缓冲区

<!-- more -->

## 缓冲区
缓冲区根据其对应的是`输入设备`还是`输出设备`，分为`输入缓冲区`和`输出缓冲区`

## 为什么要引入缓冲区
比如从磁盘里取信息，我们先把读出的数据放在缓冲区，计算机再直接从缓冲区中取数据，等缓冲区的数据取完后再去磁盘中读取，这样就可以减少磁盘的读写次数，再加上计算机对缓冲区的操作大大快于对磁盘的操作，故应用缓冲区可大大提高计算机的运行速度。

又比如，我们使用打印机打印文档，由于打印机的打印速度相对较慢，我们先把文档输出到打印机相应的缓冲区，打印机再自行逐步打印，这时我们的CPU可以处理别的事情。

现在你基本明白了吧，缓冲区就是一块内存区，它用在输入输出设备和CPU之间，用来缓存数据。它使得低速的输入输出设备和高速的CPU能够协调工作，避免低速的输入输出设备占用CPU，解放出CPU，使其能够高效率工作。

## 缓冲区的类型
缓冲区分为三种类型：全缓冲、行缓冲和不带缓冲
- `全缓冲`
在这种情况下，当填满缓冲区后才进行实际I/O操作。
全缓冲的典型代表是对磁盘文件的读写。
- `行缓冲`
在这种情况下，当在输入和输出中遇到换行符时，执行真正的I/O操作。
这时，我们输入的字符先存放在缓冲区，等按下回车键换行时才进行实际的I/O操作。
典型代表是标准输入(stdin)和标准输出(stdout)。
- `不带缓冲`
也就是不进行缓冲，标准错误文件 stderr 是典型代表，这使得出错信息可以直接尽快地显示出来

## 缓冲区的大小
如果我们没有自己设置缓冲区的话，系统会默认为标准输入输出设置一个缓冲区，这个缓冲区的大小通常是`512个字节`的大小

缓冲区大小由 stdio.h 头文件中的宏 BUFSIZ 定义，如果希望查看它的大小，包含头文件，直接输出它的值即可
缓冲区的大小是可以改变的，也可以将文件关联到自定义的缓冲区，详情可以查看 `setvbuf()` 和 `setbuf()` 函数

## 缓冲区的刷新
下列情况会引发缓冲区的刷新：
- 缓冲区满时
- 行缓冲区遇到回车时
- 关闭文件
- 使用特定函数刷新缓冲区

## scanf()
scanf()是带有缓冲区的，scanf()先从缓冲区获取数据，如果缓冲区没有数据，则等待用户输入
scanf()匹配到想要的数据之后，就会将匹配到的数据从缓冲区中删除，没有匹配到的数据不被影响
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a, b;
    char str[50];
    printf("b: ");
    scanf("%d", &b);
    scanf("%d", &a);
    scanf("%d", &b);
    scanf("%s", str);
    printf("a: %d, b: %d, str: %s\n", a, b, str);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [18:51:11]
$ gcc a.c

# root @ localhost in ~ [18:51:12]
$ ./a.out
b: 99
11 22 string
a: 11, b: 22, str: string

# root @ localhost in ~ [18:51:25]
$ ./a.out
b: 99
11 string
a: 11, b: 99, str: string

# root @ localhost in ~ [18:51:35]
$ ./a.out
b: 99
string
a: 0, b: 99, str: string
</script></code></pre>

需要注意的是，从命令行输入的都是字符串类型，都可以被`%s`匹配

输入缓冲区虽然方便，但是有时候却会带来别的麻烦
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int m, n;
    scanf("m=%d", &m);
    scanf("n=%d", &n);
    printf("m=%d, n=%d\n", m, n);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [19:01:19]
$ gcc a.c

# root @ localhost in ~ [19:01:49]
$ ./a.out
m=10
m=10, n=0
</script></code></pre>

我们还没有输入n，就直接打印数值了，这就是输入缓冲区导致的
输入`m=10`然后按下回车，第一个scanf()匹配成功，这时候缓冲区的内容是换行符，一般情况下scanf()会忽略它，但是当控制字符串不是以`%xxx`开头时，就不会忽略了，比如，下面这种情况就会被scanf()忽略
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int m, n;
    printf("enter 2 nums:\n");
    scanf("%d", &m);
    scanf("%d", &n);
    printf("m = %d, n = %d\n", m, n);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [19:07:31]
$ ./a.out
enter 2 nums:
10
20
m = 10, n = 20
</script></code></pre>

## 清空缓冲区
清空缓冲区主要有两种思路：一是将缓冲区的数据直接丢弃，二是将缓冲区的数据读取出来，但不使用
第一种思路：
windows下用`fflush(stdin)`，linux下用`setbuf(stdin, NULL)`，这两个函数都在头文件`stdio.h`中

第二种思路：
- 第一种方法
<pre><code class="language-c line-numbers"><script type="text/plain">int c;
while((c = getchar()) != '\n' && c != EOF);
</script></code></pre>

- 第二种方法
<pre><code class="language-c line-numbers"><script type="text/plain">scanf("%*[^\n]%*c");
/*
%*[^\n] 匹配除了换行符之外的字符，前面的*号表示不保存这些数据，直接丢弃
%c      匹配剩下的换行符，*号的作用同上
*/
</script></code></pre>


综上所述，如果不考虑移植性，就用第一种思路的方法，如果考虑移植性，建议用第二种思路的第二种方法

## getch() getche() getchar()
getchar()是有缓冲区的，而getche()和getch()没有缓冲区
