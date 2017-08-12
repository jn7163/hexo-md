---
title: c语言 - IO详解
date: 2017-07-13 14:00:00
categories:
- c
tags:
- c
keywords: c语言 io详解 输入、输出流 创建文件、文件夹
---

> 
c语言 - IO详解，我们对文件的概念已经非常熟悉了，比如常见的 Word 文档、txt 文件、源文件等。
文件是数据源的一种，最主要的作用是保存数据。
在操作系统中，为了统一对各种硬件的操作，简化接口，不同的硬件设备也都被看成一个文件。
对这些文件的操作，等同于对磁盘上普通文件的操作。
例如，通常把显示器称为`标准输出文件`，printf 就是向这个文件输出，把键盘称为`标准输入文件`，scanf 就是从这个文件获取数据

<!-- more -->

## 文件概述
- `stdin`：标准输入文件，一般指键盘；`scanf()`默认从该文件获得数据
- `stdout`：标准输出文件，一般指显示器；`printf()`默认向该文件输出数据
- `stderr`：标准错误文件，一般指显示器，`perror()`默认向该文件输出数据
- `stdprn`；标准打印文件，一般指打印机

> 
**`一切皆是文件`**是 Unix/Linux 的基本哲学之一
不仅普通的文件，目录、字符设备、块设备、 套接字等在 Unix/Linux 中都是以文件被对待；
它们虽然类型不同，但是对其提供的却是同一套操作界面

文件操作的流程：`打开文件` -> `读写文件` -> `关闭文件`

所谓打开文件，就是获取文件的有关信息，例如文件名、文件状态、当前读写位置等，这些信息会被保存到一个 FILE 类型的结构体变量中
关闭文件就是断开与文件之间的联系，释放结构体变量，同时禁止再对该文件进行操作

所有的文件(保存在磁盘)都要载入内存才能处理，所有的数据必须写入文件(磁盘)才不会丢失
数据在文件和内存之间传递的过程叫做`文件流`，类似水从一个地方流动到另一个地方
数据从文件复制到内存的过程叫做`输入流`，从内存保存到文件的过程叫做`输出流`

文件是数据源的一种，除了文件，还有数据库、网络、键盘等；
数据传递到内存也就是保存到C语言的变量（例如整数、字符串、数组、缓冲区等）
我们把数据在`数据源`和程序(`内存`)之间传递的过程叫做`数据流(Data Stream)`
相应的，数据从`数据源`到程序(`内存`)的过程叫做`输入流(Input Stream)`;
从程序(`内存`)到`数据源`的过程叫做`输出流(Output Stream)`

`输入输出(Input/Output, IO)`是指程序(内存)与外部设备(键盘、显示器、磁盘、其他计算机等)进行交互的操作
几乎所有的程序都有输入与输出操作，如从键盘上读取数据，从本地或网络上的文件读取数据或写入数据等
通过输入和输出操作可以从外界接收信息，或者是把信息传递给外界

我们可以说，打开文件就是打开了一个流

## 文件的打开和关闭
`FILE *fopen(char *filename, char *mode);`
用于打开一个文件，`filename`是文件名，`mode`是打开方式，均为字符串，`fopen()`会获取文件相关信息，并保存到一个`FILE`类型的结构体变量中，并返回该变量的指针

`FILE *fp = fopen("/etc/sysctl.conf", "r");`
只读方式打开文件"/etc/sysctl.conf"，并用指针指向该文件，这样就能用`文件指针`fp来操作文件了

**文件打开方式的区别**
mode|含义|解释
:---:|:---:|---
`r`、`rb`|`只读`|`该文件必须存在`，否则打开失败返回NULL、文件指针位于`文件开头`、可以从文件的`任意位置读取`
`w`、`wb`|`只写`|`该文件若不存在则新建，存在则清空文件内容`、文件指针位于`文件开头`、可以向文件的`任意位置写入`、且`写入时会覆盖原有位置的内容`
`a`、`ab`|`追加`|`该文件若不存在则新建，存在则不清空文件内容`、文件指针位于`文件结尾`、只能在`文件末尾追加内容`
`r+`、`rb+`|`读写`|`该文件必须存在`，否则打开失败返回NULL、文件指针位于`文件开头`、`不会清空原有内容`、可以在`任意位置读写`、`写入时会覆盖原有位置的内容`
`w+`、`wb+`|`读写`|`该文件若不存在则新建，存在则清空文件内容`、文件指针位于`文件开头`、可以在`任意位置读写`、`写入时会覆盖原有位置的内容`
`a+`、`ab+`|`读写`|`该文件若不存在则新建，存在则不清空文件内容`、文件指针位于`文件结尾`、可以在`任意位置读取`，只能在`尾部追加数据`

**二进制文件和文本文件打开方式的区别**
从根本上讲，二进制文件和文本文件在磁盘中没有区别，都是以二进制的形式存储
二进制和文本模式的区别在于**对换行符和一些非可见字符的转化**上，如非必要，是使用二进制读取会比较安全一些

因为 Windows 和 Linux 中的换行符不一致，前者使用`CRLF`(即`\r\n`)表示换行，后者则使用`LF`(即`\n`)表示换行
而C语言本身使用`LF`(即`\n`)表示换行，所以在文本模式下，需要转换格式(如Windows)，但是在 Linux 下，文本模式和二进制模式就没有什么区别

另外，以文本方式打开时，遇到结束符`CTRLZ(0x1A)`就认为文件已经结束
所以，若使用文本方式打开二进制文件，就很容易出现文件读不完整，或內容不对的错误
即使是用文本方式打开文本文件，也要谨慎使用，比如复制文件，就不应该使用文本方式

`r`、`w`、`a`、`+`、`b`、`t`含义：
- `r`：read 读取
- `w`：write 写入
- `a`：append 追加
- `+`：读取和写入
- `b`：binary 二进制文件
- `t`: text 文本文件(默认)

打开一个文件时，如果出错，fopen()将返回 NULL 指针，可用这一信息判断是否正确打开文件：
<pre><code class="language-c line-numbers"><script type="text/plain">if(fp == NULL){
    printf("打开文件出错!");
    exit(1);
}
</script></code></pre>

文本文件读入内存时，要将ASCII码转换成二进制码，而把文件以文本方式写入磁盘时，也要把二进制码转换成ASCII码，因此文本文件的读写要花费较多的转换时间

标准输入文件 stdin（键盘）、标准输出文件 stdout（显示器）、标准错误文件 stderr（显示器）是由系统打开的，可直接使用

**关闭文件**
文件使用完之后，应该调用函数`fclose()`关闭文件，释放系统资源，避免数据丢失
`int fclose(FILE *fp);`
正常关闭返回0，其他值表示关闭出错

## 文件的读取和写入
### fgetc()
`int fgetc(FILE *fp)` 读取单个字符

读取成功返回读到的字符，读取到文件结尾或者读取失败返回`EOF`

`EOF`即`end of line`，是 stdio.h 中定义的宏，值为负数，往往是-1，返回值为 int 就是为了兼容 EOF
注意，EOF不一定是 -1，也可能是其他负数，具体看编译器实现

在文件内部有个位置指针，用来标识当前读写到的位置，每次调用读写函数，位置指针会相应的移动
比如每次调用 fgetc()，都会使位置指针向后移动一个字符，所以可以连续读取多个字符

在文件 foo.txt 中预先写入一些内容，然后用 fgetc() 读取：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

int main(){
    FILE *fp = fopen("foo.txt", "r");
    if(fp == NULL){
        printf("IO ERROR!\n");
        exit(1);
    }
    char ch;
    while((ch = fgetc(fp)) != EOF){
        printf("%c", ch);
    }
    fclose(fp);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [15:40:03]
$ gcc a.c

# root @ localhost in ~/work [15:40:12]
$ cat foo.txt
www.google.com
www.zfl9.com
www.baidu.com

# root @ localhost in ~/work [15:40:15]
$ ./a.out
www.google.com
www.zfl9.com
www.baidu.com
</script></code></pre>

**对EOF的说明**
EOF本来表示文件结尾，意味着读取结束，但是很多函数在读取出错时也返回 EOF，我们可以借助 feof() 和 ferror() 来判断到底是读取完毕还是读取出错：
- `int feof(FILE *fp)`：当指向文件末尾时，返回非零值(true)，否则返回零值(false)
- `int ferror(FILE *fp)`：当文件操作出错是，返回非零值(true)，否则返回零值(false)

一般文件出错很少见，如果追求完美，建议加上该判断

### fputc()
`int fputc(int ch, FILE *fp)` 写入单个字符
写入成功返回写入的字符，失败返回EOF

从标准输入读取一行字符，写入文件 foo.txt，然后再从文件中读取出来，打印到标准输出
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

int main(){
    FILE *fp = fopen("foo.txt", "w");
    if(fp == NULL){
        printf("IO ERROR!\n");
        exit(1);
    }
    char ch;
    printf("input: ");
    while((ch = getchar()) != '\n'){
        fputc(ch, fp);
    }
    fclose(fp);

    fp = fopen("foo.txt", "r");
    if(fp == NULL){
        printf("IO ERROR!\n");
        exit(1);
    }
    while((ch = fgetc(fp)) != EOF){
        putchar(ch);
    }
    fclose(fp);

    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [16:00:23]
$ gcc a.c

# root @ localhost in ~/work [16:00:24]
$ ./a.out
input: https://www.google.com.hk/index.html?xxx
https://www.google.com.hk/index.html?xxx#

# root @ localhost in ~/work [16:00:43]
$ cat foo.txt
https://www.google.com.hk/index.html?xxx#
</script></code></pre>

### fgets()
`char *fgets(char *str, int n, FILE *fp)` 读取一个字符串
从文件中读取一个字符串，并保存到一个字符数组，然后返回该字符数组的首地址
`str`为字符数组，`n`为读取的字符数，`fp`为文件指针
读取成功返回str首地址，读取失败返回NULL，如果开始读取时文件指针指向文件末尾，则读取不到任何字符，也返回NULL

注意，读取到的字符会在末尾自动添加`'\0'`，n个字符也包括`'\0'`字符，也就是说实际也就读取到了`n-1`个字符
如果希望读取到100个字符，则n要为101：
<pre><code class="language-c line-numbers"><script type="text/plain">#define N 101
char str[N];
FILE *fp = fopen("foo.txt", "r");
fgets(str, N, fp);
</script></code></pre>

另外，在读取到 n-1 个字符之前，如果遇到了换行符或者到了文件结尾，就读取结束，这意味着，不管 n 多大，fgets()最多读取一行数据
所以，如果我们要按行读取文件，可以将 n 值设置到足够大，这样每次都是读取一行数据

例如：一行一行的读取数据，并在前面添加行号：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

#define N 1024+1

int main(){
    FILE *fp = fopen("foo.txt", "r");
    if(fp == NULL){
        printf("IO ERROR!\n");
        exit(1);
    }
    char str[N];
    int i = 1;
    while(fgets(str, N, fp) != NULL){
        printf("%d: %s", i, str);
        i++;
    }
    fclose(fp);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [16:32:56]
$ cat foo.txt
www.zfl9.com
www.google.com
www.baidu.com

# root @ localhost in ~/work [16:32:57]
$ gcc a.c

# root @ localhost in ~/work [16:33:03]
$ ./a.out
1: www.zfl9.com
2: www.google.com
3: www.baidu.com
</script></code></pre>

**注意：fgets()会读取到换行符，而gets()不一样，会忽略换行符**

### fputs()
`int fputs(char *str, FILE *fp)` 写入一个字符串
写入成功返回非负数，否则返回EOF

例如：向上个例子的foo.txt中添加一行数据：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define N 50

int main(){
    FILE *fp = fopen("foo.txt", "a");
    if(fp == NULL){
        printf("IO ERROR!\n");
        exit(1);
    }
    char str[N];
    printf("input: ");
    scanf("%[^\n]", str);
    strcat(str, "\n");
    fputs(str, fp);
    fclose(fp);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [16:41:37]
$ ./a.out
input: www.bing.com

# root @ localhost in ~/work [16:41:56]
$ cat foo.txt
www.zfl9.com
www.google.com
www.baidu.com
www.bing.com
</script></code></pre>

### fread()、fwrite()
`size_t fread(void *ptr, size_t size, size_t count, FILE *fp);`
`size_t fwrite(void *ptr, size_t size, size_t count, FILE *fp);`

`ptr`是内存区块的指针，可以是任何类型的数据，在fread()中用来存放读取的数据，在fwrite()中用来存放要被写入的数据
`size`表示每次读取/写入的长度，单位为字节
`count`表示总共读取/写入的次数
`fp`为文件指针

理论上，读取/写入 size*count 大小的数据

size_t 是在 stddef.h 头文件中使用 typedef 定义的数据类型，表示无符号整数，也即非负数，常用来表示数量

返回值：返回成功读写的块数，即count，如果返回值小于count：
对于 fwrite() 来说，肯定发生了写入错误，可以用 ferror() 函数检测
对于 fread() 来说，可能读到了文件末尾，可能发生了错误，可以用 ferror() 或 feof() 检测

如，从键盘读取一个数组，将其写入文件，然后再读取出来：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

#define N 5

int main(){
    FILE *fp = fopen("foo.txt", "wb+");
    if(fp == NULL){
        printf("IO ERROR!\n");
        exit(1);
    }

    int array[N], array_read[N];
    printf("input array[%d]: ", N);
    for(int i=0; i<N; i++){
        scanf("%d", array+i);
    }

    fwrite(array, sizeof(int), N, fp);

    rewind(fp);
    fread(array_read, sizeof(int), N, fp);
    for(int i=0; i<N; i++){
        printf("array[%d] = %d\n", i, array_read[i]);
    }

    fclose(fp);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [17:02:52]
$ ./a.out
input array[5]: 1 2 3 4 5
array[0] = 1
array[1] = 2
array[2] = 3
array[3] = 4
array[4] = 5
</script></code></pre>

fread()/fwrite()直接操作字节，建议用二进制模式打开文件
打开foo.txt文件查看内容，你会发现全是乱码，因为这是该数组在内存中的拷贝，原模原样的保存到了文件中

当调用fwrite()后，文件指针指向了文件末尾，所以要想读取文件内容，需要将文件指针重置到文件开头位置，这就是rewind(fp)的作用

### fprintf()、fscanf()
`int fscanf(FILE *fp, char *format, ...);` 格式化读取
`int fprintf(FILE *fp, char *format, ...);` 格式化写入

和之前用的`printf()/scanf()`很相似，就是前面多了个文件指针fp而已

`fprintf()`返回成功写入的字符个数，失败则返回负数，`fscanf()`返回参数列表中成功赋值的参数个数

如，从键盘读取学生的信息，然后将其保存到文件foo.txt，最后将其读取出来送往显示器
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

#define N 3

typedef struct Student{
    char name[50];
    int age;
    float score;
} Student;

int main(){
    Student info1[N], *ptr = info1;
    FILE *fp = fopen("foo.txt", "w+");
    if(fp == NULL){
        exit(1);
    }

    // 从键盘读取学生信息，保存到数组 info1，同时将数组 info1 的学生信息写入到文件
    for(int i=1; ptr<info1+N; i++, ptr++){
        printf("Student_Info[%d]: ", i);
        scanf("%s %d %f", ptr->name, &ptr->age, &ptr->score);
        fprintf(fp, "%8s%8d%8.1f\n", ptr->name, ptr->age, ptr->score);
    }

    Student info2[N];
    ptr = info2;
    rewind(fp);
    // 从文件读取学生信息，保存到数组 info2，同时将数组 info2 的学生信息输出到屏幕
    for(; ptr<info2+N; ptr++){
        fscanf(fp, "%s %d %f", ptr->name, &ptr->age, &ptr->score);
        printf("%8s%8d%8.1f\n", ptr->name, ptr->age, ptr->score);
    }

    ptr = NULL;
    fclose(fp);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [17:53:56]
$ ./a.out
Student_Info[1]: Zhang3 15 140
Student_Info[2]: Li4 15 130.5
Student_Info[3]: Wang5 15 148.5
  Zhang3      15   140.0
     Li4      15   130.5
   Wang5      15   148.5

# root @ localhost in ~/work [17:54:22]
$ cat foo.txt
  Zhang3      15   140.0
     Li4      15   130.5
   Wang5      15   148.5
</script></code></pre>

如果将 fp 设置为 stdin，那么 fscanf() 函数将会从键盘读取数据，与 scanf 的作用相同；
设置为 stdout，那么 fprintf() 函数将会向显示器输出内容，与 printf 的作用相同

### fseek()、rewind()
前面介绍的文件读写函数都是顺序读写，即读写文件只能从头开始，依次读写各个数据
但在实际开发中经常需要读写文件的中间部分，要解决这个问题，就得先移动文件内部的位置指针，再进行读写
这种读写方式称为随机读写，也就是说从文件的任意位置开始读写

实现随机读写的关键是要按要求移动位置指针，这称为文件的定位

`void rewind(FILE *fp)`：rewind()用于将文件指针重新指向文件开头
`int fseek(FILE *fp, long offset, int origin)`：fseek()用于将文件指针指向任意位置
`offset`为偏移量，即要移动的字节数
`origin`为起始位置，也就是从何处计算偏移量，有三种取值`SEEK_SET`文件开头、`SEEK_CUR`当前位置、`SEEK_END`文件末尾，这是三个常量，值分别为 0、1、2
如果指针移动成功，返回0值，移动失败，则不改变指针的位置，返回非0值

如：`fseek(fp, 100, 0);`移动到文件第100字节处

## 获取文件大小
头文件：stdio.h
`long ftell(FILE *fp)`；返回当前文件指针自文件开头的偏移量，单位为字节
`int fgetpos(FILE *stream, fpos_t *pos)`：获取当前文件流的读写位置，成功返回0，失败返回非0
`int fsetpos(FILE *stream, const fpos_t *pos)`：设置当前文件流的读写位置，成功返回0，失败返回非0

如；可以通过ftell()获取文件的大小
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

long fsize(FILE *fp){
    fseek(fp, 0, SEEK_END);
    return ftell(fp);
}

int main(){
    FILE *fp = fopen("/boot/vmlinuz-3.10.0-514.26.2.el7.x86_64", "rb");
    printf("len: %ld bytes\n", fsize(fp));
    fclose(fp);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [18:16:49]
$ gcc a.c

# root @ localhost in ~/work [18:16:50]
$ ./a.out
len: 5397008 bytes
</script></code></pre>

但是这样会移动FILE指针的读写位置，再修改一下：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

long fsize(FILE *fp){
    fpos_t fpos;
    long size;
    fgetpos(fp, &fpos);
    fseek(fp, 0, 2);
    size = ftell(fp);
    fsetpos(fp, &fpos);
    return size;
}

int main(void){
    FILE *fp = fopen("foo.txt", "r+");
    fputs("line1\n", fp);
    printf("size: %ld\n", fsize(fp));
    fputs("line2\n", fp);
    fclose(fp);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [15:57:52]
$ cat foo.txt
net.ipv4.ip_forward = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.ip_local_port_range = 4096 65000
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_max_syn_backlog = 4096
net.core.netdev_max_backlog =  10240
net.core.somaxconn = 2048
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_mem = 177945 216076 254208
net.ipv4.tcp_fastopen = 3

# root @ localhost in ~ [15:57:56]
$ ./a.out
size: 594

# root @ localhost in ~ [15:57:58]
$ cat foo.txt
line1
line2
forward = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.ip_local_port_range = 4096 65000
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_max_syn_backlog = 4096
net.core.netdev_max_backlog =  10240
net.core.somaxconn = 2048
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_mem = 177945 216076 254208
net.ipv4.tcp_fastopen = 3
</script></code></pre>

还有一种方法，也可以获取文件大小：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <sys/stat.h>

int main(void){
    struct stat fstat;
    stat("foo.txt", &fstat);
    printf("size: %ld\n", fstat.st_size);
    return 0;
}
</script></code></pre>

## 文件缓冲区
通过 fopen() 打开一个文件后，fopen() 会默认为其设置一个缓冲区

对于 stdin/stdout，系统默认设置了 BUFSIZ 大小的缓冲区，BUFSIZ是stdio.h定义的宏，我系统上大小为8192kb

- `int fflush(FILE *stream)`
如果文件指针为NULL，则刷新所有打开的文件的缓冲区
返回值：成功返回0，失败返回EOF
C语言标准中，fflush()是刷新输出流的缓冲区，对于输入流的缓冲区刷新，C语言并没有定义
但是某些编译器也定义了对刷新输入流的缓冲区的实现，如微软的vc/vs，而gcc没有定义
- `void setbuf(FILE *stream, char *buf)`
将缓冲区与流关联，buf为缓冲区的首地址，如`setbuf(stdin, NULL)`清空标准输入缓冲区，再比如：`char buffer[1024*4]; setbuf(stdin, buffer);`为标准输入流设置自定义缓冲区buffer
- `int setvbuf(FILE *stream, char *buf, int type, unsigned size)`
设置文件流缓冲区，buf为缓冲区首地址，type为缓冲区类型，size为缓存区大小(字节); 成功返回0，失败返回非0
type的类型：`_IOFBF`全缓冲、`_IOLBF`行缓冲、`_IONBF`无缓冲

**FILE结构体**
`FILE *fp;`

这里的FILE，实际上是在stdio.h中定义的一个结构体，该结构体中含有文件名、文件状态和文件当前位置等信息，fopen 返回的就是FILE类型的指针

注意：FILE是`文件缓冲区`的结构，fp也是指向`文件缓冲区`的指针

不同编译器 stdio.h 头文件中对 FILE 的定义略有差异，这里以标准C举例说明：
<pre><code class="language-c line-numbers"><script type="text/plain">typedef struct _iobuf {
    int cnt;  // 剩余的字符，如果是输入缓冲区，那么就表示缓冲区中还有多少个字符未被读取
    char *ptr;  // 下一个要被读取的字符的地址
    char *base;  // 缓冲区基地址
    int flag;  // 读写状态标志位
    int fd;  // 文件描述符
    // 其他成员
} FILE;
</script></code></pre>

我们知道，当我们从键盘输入数据的时候，数据并不是直接被我们得到，而是放在了缓冲区中，然后我们从缓冲区中得到我们想要的数据 
如果我们通过setbuf()或setvbuf()函数将缓冲区设置10个字节的大小，而我们从键盘输入了20个字节大小的数据，这样我们输入的前10个数据会放在缓冲区中，因为我们设置的缓冲区的大小只能够装下10个字节大小的数据，装不下20个字节大小的数据
那么剩下的那10个字节大小的数据怎么办呢？暂时放在了输入流中，等待送入缓冲区接受处理

再说一下 FILE 结构体中几个相关成员的含义：
cnt     剩余的字符，如果是输入缓冲区，那么就表示缓冲区中还有多少个字符未被读取
ptr     下一个要被读取的字符的地址
base    缓冲区基地址

在这里有点需要说明：当我们从键盘输入字符串的时候需要敲一下回车键才能够将这个字符串送入到缓冲区中，那么敲入的这个回车键(\r)会被转换为一个换行符\n，这个换行符\n也会被存储在缓冲区中并且被当成一个字符来计算！
比如我们在键盘上敲下了123456这个字符串，然后敲一下回车键（\r）将这个字符串送入了缓冲区中，那么此时缓冲区中的字节个数是7 ，而不是6

缓冲区的刷新就是将指针 ptr 变为缓冲区的基地址 ，同时 cnt 的值变为0 ，因为缓冲区刷新后里面是没有数据的！

## 文件新建、删除、复制、插入等
**创建空文件**
头文件`stdio.h`
`FILE *fp = fopen("newfile", "wb"); fclose(fp);` 新建一个空文件

**新建文件夹**
头文件`unistd.h`

`int mkdir(char *path, mode_t mode)` 创建文件夹，成功返回0，失败返回-1
`path`为文件夹路径、`mode`为文件夹权限：和Linux下的文件权限是一个意思，比如文件夹权限一般为0755或0775

`int access(char *path, int type)` 测试文件(夹)的权限，成功返回0，失败返回-1
`type`有以下四种：
- `R_OK`：可读，对应数字为 04；
- `W_OK`：可写，对应数字为 02；
- `X_OK`：可执行，对应数字为 01；
- `F_OK`：是否存在，对于数字为 00；

比如：00表示是否存在、01表示是否可执行、02表示是否可写、04表示是否可读、06表示是否可读可写、07表示是否可读可写可执行

**删除文件**
头文件`stdio.h`
`int remove(char *filename)` 删除文件，成功返回0，失败返回-1

**删除空文件夹**
头文件`unistd.h`
`int rmdir(char *dirname)` 删除空文件夹，成功返回0，失败返回-1

**struct stat结构体**
头文件`sys/stat.h`
struct stat结构体是用来描述一个linux系统中的文件属性的结构
<pre><code class="language-c line-numbers"><script type="text/plain">struct stat
{   
    mode_t      st_mode;    /* 对于的权限 */
    uid_t       st_uid;     /* UID */
    gid_t       st_gid;     /* GID */
    off_t       st_size;    /* 文件大小，字节为单位 */
    nlink_t     st_nlink;   /* 链向此文件的连接数(硬连接) */
    time_t      st_atime;   /* 访问时间 */    
    time_t      st_mtime;   /* 修改(文件内容)时间 */
    time_t      st_ctime;   /* 修改(文件属性、权限等)时间*/
    ino_t       st_ino;     /* inode节点号 */
    dev_t       st_dev;     /* 文件所在设备的ID */
    dev_t       st_rdev;    /* 设备号，针对设备文件 */
    blksize_t   st_blksize; /* 系统块的大小 */
    blkcnt_t    st_blocks;  /* 文件所占块数 */
};
</script></code></pre>

`int fstat(int fd, struct stat *filestat);`  通过文件描述符fd获取文件信息
`int stat(char *path, struct stat *filestat);` 通过文件路径path获取文件信息，当文件为软链接文件时，返回该链接所指向的文件信息
`int lstat(char *path, struct stat *filestat);` 通过文件路径path获取文件信息，当文件为软链接文件时，返回该链接文件本身的文件信息

成功返回0，失败返回非0

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <sys/stat.h>

int main(void){
    struct stat filestat;
    stat("/etc/sysctl.conf", &filestat);
    printf("size: %ld bytes, uid: %d, gid: %d, mode: %#o\n", filestat.st_size, filestat.st_uid, filestat.st_gid, filestat.st_mode);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [6:22:07]
$ gcc a.c

# root @ localhost in ~/work [6:22:08]
$ ./a.out
size: 473 bytes, uid: 0, gid: 0, mode: 0100644
</script></code></pre>

stat结构体中的st_mode定义了下列数种情况：
<pre><code class="language-c line-numbers"><script type="text/plain">S_IFREG     0100000     一般文件
S_IFDIR     0040000     目录
S_IFLNK     0120000     符号连接
S_IFSOCK    0140000     scoket
S_IFIFO     0010000     先进先出
S_IFCHR     0020000     字符装置
S_IFBLK     0060000     区块装置
S_IFMT      0170000     文件类型的位遮罩

S_ISUID     04000       文件的sid位
S_ISGID     02000       文件的gid位
S_ISVTX     01000       文件的sticky位

S_IRUSR(S_IREAD)    00400   文件所有者具可读取权限
S_IWUSR(S_IWRITE)   00200   文件所有者具可写入权限
S_IXUSR(S_IEXEC)    00100   文件所有者具可执行权限

S_IRGRP 00040   用户组具可读取权限
S_IWGRP 00020   用户组具可写入权限
S_IXGRP 00010   用户组具可执行权限

S_IROTH 00004   其他用户具可读取权限
S_IWOTH 00002   其他用户具可写入权限
S_IXOTH 00001   其他用户具可执行权限

上述的文件类型在POSIX中定义了检查这些类型的宏定义：
S_ISLNK(st_mode)    判断是否为符号连接
S_ISREG(st_mode)    是否为一般文件
S_ISDIR(st_mode)    是否为目录
S_ISCHR(st_mode)    是否为字符装置文件
S_ISBLK(s3e)        是否为先进先出
S_ISSOCK(st_mode)   是否为socket
</script></code></pre>

**文件复制、插入、删除**
`myio.h`头文件
<pre><code class="language-c line-numbers"><script type="text/plain">#ifndef __MY_IO_H
#define __MY_IO_H
#include <stdio.h>
#include <stdbool.h>

/**
 * 获取文件大小，单位字节
 * @param filename  文件名
 * @return long     成功: 返回文件大小; 失败: 返回-1
**/
extern long fsize(const char *filename);

/**
 * 文件复制
 * @param src       原文件名
 * @param dst       目标文件名
 * @return bool     true: 复制成功; false: 复制失败
**/
extern bool copy(const char *src, const char *dest);

/**
 * 从原文件的任意位置复制任意长度的数据到目标文件的任意位置
 * @param fsrc          原文件指针
 * @param src_offset    原文件位置偏移量，即从哪里开始复制
 * @param len           复制的长度
 * @param fdst          目标文件指针
 * @param dst_offset    目标文件位置偏移量，即复制到哪里
 * @return bool         true: 操作成功; false: 操作失败
**/
extern bool fcopy(FILE *fsrc, long src_offset, long len, FILE *fdst, long dst_offset);

/**
 * 插入一段数据到指定文件
 * @param filename   文件名
 * @param offset     偏移量
 * @param buffer     插入的数据的首地址
 * @param bufferlen  插入的数据的长度
 * @return bool      true: 操作成功; false: 操作失败
**/
extern bool finsert(const char *filename, long offset, const void *buffer, long bufferlen);

/**
 * 删除文件中任意一段数据
 * @param filename  文件名
 * @param offset    偏移量，也就是从哪开始删除
 * @param len       长度
 * @return bool     true: 操作成功; false: 操作失败
**/
extern bool fdelete(const char *filename, long offset, long len);

#endif
</script></code></pre>

`myio.c`源文件
<pre><code class="language-c line-numbers"><script type="text/plain">#include "myio.h"
#include <stdlib.h>
#include <math.h>

/**
 * 获取文件大小，单位字节
 * @param filename  文件名
 * @return long     成功: 返回文件大小; 失败: 返回-1
**/
long fsize(const char *filename){
    FILE *fp = fopen(filename, "rb");
    if(fp == NULL){
        return -1;
    }
    fseek(fp, 0, SEEK_END);
    long len = ftell(fp);
    fclose(fp);
    return len;
}

/**
 * 文件复制
 * @param src       原文件名
 * @param dst       目标文件名
 * @return bool     true: 复制成功; false: 复制失败
**/
bool copy(const char *src, const char *dest){
    FILE *fp_src = fopen(src, "rb");
    FILE *fp_dest = fopen(dest, "wb");
    if(fp_src == NULL || fp_dest == NULL){
        return false;
    }
    void *buffer = malloc(1024 * 4);
    if(buffer == NULL){
        fclose(fp_src);
        fclose(fp_dest);
        return false;
    }
    int rreadc;
    while((rreadc=fread(buffer, 1, 1024 * 4, fp_src)) > 0){
        fwrite(buffer, rreadc, 1, fp_dest);
    }
    fflush(fp_dest);
    free(buffer);
    fclose(fp_src);
    fclose(fp_dest);
    return true;
}

/**
 * 从原文件的任意位置复制任意长度的数据到目标文件的任意位置
 * @param fsrc          原文件指针
 * @param src_offset    原文件位置偏移量，即从哪里开始复制
 * @param len           复制的长度
 * @param fdst          目标文件指针
 * @param dst_offset    目标文件位置偏移量，即复制到哪里
 * @return bool         true: 操作成功; false: 操作失败
**/
bool fcopy(FILE *fsrc, long src_offset, long len, FILE *fdst, long dst_offset){
    if(fsrc == NULL || fdst == NULL || len == 0){
        return false;
    }
    int bufferlen = 1024 * 4;
    void *buffer = malloc(bufferlen);
    if(buffer == NULL){
        return false;
    }
    fseek(fsrc, src_offset, SEEK_SET);
    fseek(fdst, dst_offset, SEEK_SET);
    int rreadc;
    if(len < 0){
        while((rreadc=fread(buffer, 1, bufferlen, fsrc)) > 0){
            fwrite(buffer, rreadc, 1, fdst);
        }
        fflush(fdst);
        free(buffer);
        return true;
    }else{
        int loopc = (int)ceil((double)((double)len/bufferlen));
        long read_total = 0;
        for(int i=0; i<loopc; i++){
            if(len-read_total < bufferlen){
                bufferlen = len-read_total;
            }
            rreadc = fread(buffer, 1, bufferlen, fsrc);
            read_total += rreadc;
            fwrite(buffer, rreadc, 1, fdst);
        }
        fflush(fdst);
        free(buffer);
        return true;
    }
}

/**
 * 插入一段数据到指定文件
 * @param filename   文件名
 * @param offset     偏移量
 * @param buffer     插入的数据的首地址
 * @param bufferlen  插入的数据的长度
 * @return bool      true: 操作成功; false: 操作失败
**/
bool finsert(const char *filename, long offset, const void *buffer, long bufferlen){
    if(offset < 0 || bufferlen < 0){
        return false;
    }
    FILE *fp = fopen(filename, "rb+");
    if(fp == NULL){
        return false;
    }
    long filesize = fsize(filename);
    if(offset >= filesize){
        fseek(fp, 0, SEEK_END);
        fwrite(buffer, bufferlen, 1, fp);
        fflush(fp);
        fclose(fp);
        return true;
    }else{
        char *temp_filename = ".~temp~finsert~";
        FILE *fp_temp = fopen(temp_filename, "wb+");
        if(fp_temp == NULL){
            fclose(fp);
            return false;
        }
        fcopy(fp, 0, offset, fp_temp, 0);
        fwrite(buffer, bufferlen, 1, fp_temp);
        fflush(fp_temp);
        fcopy(fp, offset, -1, fp_temp, offset+bufferlen);
        freopen(filename, "wb", fp);
        fcopy(fp_temp, 0, -1, fp, 0);
        fclose(fp_temp);
        remove(temp_filename);
        fflush(fp);
        fclose(fp);
        return true;
    }
}

/**
 * 删除文件中任意一段数据
 * @param filename  文件名
 * @param offset    偏移量，也就是从哪开始删除
 * @param len       长度
 * @return bool     true: 操作成功; false: 操作失败
**/
bool fdelete(const char *filename, long offset, long len){
    long filesize = fsize(filename);
    if(len <= 0 || offset > filesize){
        return true;
    }
    FILE *fp = fopen(filename, "rb+");
    if(offset <= 0){
        freopen(filename, "wb", fp);
        fclose(fp);
        return true;
    }else{
        char *temp_filename = ".~temp~fdelete~";
        FILE *fp_temp = fopen(temp_filename, "wb+");
        if(fp_temp == NULL){
            fclose(fp);
            return false;
        }
        fcopy(fp, 0, offset, fp_temp, 0);
        fcopy(fp, offset+len, -1, fp_temp, offset);
        freopen(filename, "wb", fp);
        fcopy(fp_temp, 0, -1, fp, 0);
        fclose(fp_temp);
        remove(temp_filename);
        fflush(fp);
        fclose(fp);
        return true;
    }
}
</script></code></pre>
