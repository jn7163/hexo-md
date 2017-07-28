---
title: c语言 - 输入与输出
date: 2017-07-01 16:58:00
categories:
- c
tags:
- c
keywords: c语言输入与输出
---

> 
c语言 - 输入与输出，gcc分步编译，输入缓冲区，以及如何清空输入缓冲区

<!-- more -->

## gcc分步编译
<pre><code class="language-c line-numbers"><script type="text/plain">## 1. 编写源文件 main.c
vim main.c

## 2. 预处理 Pre-processing
gcc -E main.c -o main.i

## 3. 编译 Compiling
gcc -S main.i -o main.s

## 4. 汇编 Assembling
gcc -c main.s -o main.o

## 5. 链接 Linking
gcc main.o -o main

## 6. 运行程序 main
./main
</script></code></pre>

## 输出
C语言输出函数主要有3个，`putchar`, `puts`, `printf`, 都包含在`stdio.h`头文件中
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    putchar('A');
    putchar('\n');
    puts("this is puts");
    printf("this is printf");
    return 0;
}
</script></code></pre>

注意，`puts`会自动添加换行符`\n`，而`printf`不会，`printf`完全可以替代`putchar`和`puts`，而且`printf`可以格式化输出，非常强大

### printf 格式化输出
<pre><code class="language-c line-numbers"><script type="text/plain">printf()格式转换的一般形式如下：
%(flags)(width)(. prec)type
以括号括起来的参数为选择性参数，而 % 与 type 则是必要的，下面介绍 type 的几种形式。

1) 整数
%d  整数的参数会被转成有符号的十进制数字
%hd 短整型
%ld 长整型
%u  整数的参数会被转成无符号的十进制数字
%o  整数的参数会被转成无符号的八进制数字
%x  整数的参数会被转成无符号的十六进制数字，并以小写abcdef表示
%X  整数的参数会被转成无符号的十六进制数字，并以大写ABCDEF表示

2) 浮点数
%f  double型的参数会被转成十进制数字，并取到小数点以后六位，四舍五入(Windows下，Linux下直接截取前6位小数）
%lf 双精度浮点数
%e  double型的参数以指数形式打印，有一个数字会在小数点前，六位数字在小数点后，而在指数部分会以小写的e来表示
%E  与 %e 作用相同，唯一区别是指数部分将以大写的E表示
%g  double型的参数会自动选择以 %f 或 %e 的格式来打印，其标准是根据打印的数值及所设置的有效位数来决定。
%G  与 %g 作用相同，唯一区别在以指数形态打印时会选择 %E 格式。

3) 字符及字符串
%c  输出单个字符
%s  指向字符串的参数会被逐字输出，直到出现 NULL 字符为止
%p  如果参数是指针变量则使用十六进制格式显示

4) prec 有几种情况：
1. 正整数的最小位数
2. 在浮点数中代表小数位数
3. 格式代表有效位数的最大值
4. 在 %s 格式代表字符串的最大长度
5. 若为 × 符号则代表下个参数值为最大长度

5) width 为参数的最小长度
若此栏并非数值，而是 * 符号，则表示以下一个参数当做参数长度。

6) flags 有下列几种情况：
1. + 正数前面会添加 + 符号
2. # 在八进制和十六进制数前面时，打印前缀0、0x、0X; 在型态为 e、E、f、g 或 G 之前则会强迫数值打印小数点; 在类型为 g 或 G 之前时则同时保留小数点及小数位数末尾的零。
3. 0 当有指定参数时，无数字的参数将补上0; 默认是关闭此旗标，所以一般会打印出空白字符。
</script></code></pre>

### 转义字符
获取`'a'` `'b'` `'c'`对应的八进制ascii码
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    printf("ASCII of 'abc': %#o %#o %#o\n", 'a', 'b', 'c');
    return 0;
}
</script></code></pre>

编译并运行
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [11:59:22]
$ gcc hello.c

# root @ localhost in ~ [12:00:06]
$ ./a.out
ASCII of 'abc': 0141 0142 0143
</script></code></pre>

用ascii码代替字符
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    printf("abc\n");
    printf("\141\142\143\n");
    return 0;
}
</script></code></pre>

编译并运行
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [12:02:27]
$ gcc b.c

# root @ localhost in ~ [12:02:29]
$ ./a.out
abc
abc
</script></code></pre>

也可用十六进制代替
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    printf("abc\n");
    printf("\141\142\143\n");
    printf("\x61\x62\x63\n");
    return 0;
}
</script></code></pre>

运行结果
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [12:04:55]
$ gcc b.c

# root @ localhost in ~ [12:04:57]
$ ./a.out
abc
abc
abc
</script></code></pre>

## 输入
C语言输入函数主要有`getchar`、`getche`、`getch`、`gets`和`scanf`
其中`getche`和`getch`是vs中的函数(在conio.h头文件中)，gcc中没有，`gets`在gcc、vs2017中已经不能使用了

### getchar
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char c=getchar();
    putchar(c);
    return 0;
}
</script></code></pre>

运行该程序，输入任意字符(串)，回车，`getchar`只获取第一个字符，
其实其他的字符都进入了缓冲区，我们看看下面的代码

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    printf("enter 5 char: ");
    char c;
    int i;
    for(i=0; i<5; i++){
        c=getchar();
        putchar(c);
    }
    return 0;
}
</script></code></pre>

编译并运行，输入5个字符
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [10:03:10]
$ gcc hello.c

# root @ localhost in ~ [10:05:58]
$ ./a.out
enter 5 char: 12345
12345#
</script></code></pre>

### getche
获取单个字符，有回显，编译环境是vs2017
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <conio.h>
#include <stdlib.h>

int main() {
    char c;
    c = getche();
    putchar(c);
    system("pause");
    return 0;
}
</script></code></pre>

输入`a`
<pre><code class="language-c line-numbers"><script type="text/plain">aa请按任意键继续. . .
</script></code></pre>

### getch
获取单个字符，没有回显，编译环境是vs2017
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <conio.h>
#include <stdlib.h>

int main() {
    char c;
    c = getch();
    putchar(c);
    system("pause");
    return 0;
}
</script></code></pre>

输入`a`
<pre><code class="language-c line-numbers"><script type="text/plain">a请按任意键继续. . .
</script></code></pre>

### scanf
> 
`scanf`是以空白符(空格,制表符,换行符等)为分隔符的，支持格式化扫描，与`printf`很相似，`scanf`和`getchar`一样，也是存在输入缓冲区的

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    printf("a + b: ");
    int a, b;
    scanf("%d + %d", &a, &b);
    printf("%d + %d = %d\n", a, b, a+b);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [10:11:21]
$ gcc hello.c

# root @ localhost in ~ [10:13:01]
$ ./a.out
a + b: 1+1
1 + 1 = 2

# root @ localhost in ~ [10:13:05]
$ ./a.out
a + b: 2 + 4
2 + 4 = 6
</script></code></pre>

可以发现，`scanf`对于空白符的数量并不要求严格匹配

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    printf("enter 5 char: ");
    char c;
    int i;
    for(i=0; i<5; i++){
        scanf("%c", &c);
        printf("%c", c);
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [10:29:02]
$ gcc hello.c

# root @ localhost in ~ [10:29:09]
$ ./a.out
enter 5 char: 12345
12345#

# root @ localhost in ~ [10:29:11]
$ ./a.out
enter 5 char: 1234567890
12345#
</script></code></pre>

看看用`scanf`如何输入字符串，以及如何输入带空格的字符串
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char str[100];
    scanf("%s", str);
    printf("%s", str);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [8:13:15]
$ gcc a.c

# root @ localhost in ~ [8:13:17]
$ ./a.out
my name is otokaze
my#
</script></code></pre>

> 
因为`scanf`遇到空白符就停止扫描了，所以只保存了第一个单词`my`，那怎么输入带有空白符的字符串呢？
我们可以用`%[^\n]`来指定`scanf`的分隔符为`\n`换行符，这样就可以输入完整的一行字符串了

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char str[100];
    scanf("%[^\n]", str);
    printf("%s", str);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [8:18:14]
$ gcc a.c

# root @ localhost in ~ [8:18:16]
$ ./a.out
my name is otokaze
my name is otokaze#
</script></code></pre>

### 清空输入缓冲区
我们先来看这段代码
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char c1, c2;
    printf("char1: ");
    c1=getchar();
    printf("char2: ");
    c2=getchar();
    printf("ASCII of char: %d\t%d\n", c1, c2);
    return 0;
}
</script></code></pre>

这个程序用来打印输入的字符的ASCII码，但是很不幸，由于输入缓冲区的存在，并不能达到我们要的结果
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [10:42:22]
$ gcc a.c

# root @ localhost in ~ [10:43:54]
$ ./a.out
char1: a
char2: ASCII of char: 97	10

# root @ localhost in ~ [10:44:00]
$ ./a.out
char1: ab
char2: ASCII of char: 97	98
</script></code></pre>

我们只输入了第一个字符，就直接显示了结果，为什么会这样呢？
仔细观察可以发现，输入a，然后回车，直接打印了97和10，
查看ascii码表，发现对应的ascii字符为a和换行符

分析：
`getchar`和`scanf`都是从输入缓冲区获取数据，在键盘键入任意字符，当遇到换行符`\n`时，输入的字符(串)和`\n`会被送入输入缓冲区，`getchar`和`scanf`从输入缓冲区获取数据，直到缓冲区读完，再将键盘缓冲区中的数据送入输入缓冲区，然后再从输入缓冲区获取数据

那么我们再来分析第一次运行的结果：
我们只输入了a，然后回车，实际上在内存中的缓冲区存放的数据是`a\n`，第一个`getchar`获取`a`，第二个`getchar`获取`\n`，所以输出对应的ascii码

再来分析第二次运行的结果：
我们输入了ab，然后回车，这时候输入缓冲区的内容是`ab\n`，两个`getchar`获取了前两个字符，所以输出了a和b对应的ascii码

那我们如何实现我们想要的结果呢？只要清空输入缓冲区就行
windows用`fflush(stdin)`或`rewind(stdin)`，linux下则要用`setbuf(stdin, NULL)`，这三个函数都在`stdio.h`头文件中
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char c1, c2;
    printf("char1: ");
    c1=getchar();
    setbuf(stdin, NULL);
    printf("char2: ");
    c2=getchar();
    printf("ASCII of char: %d\t%d\n", c1, c2);
    return 0;
}
</script></code></pre>

再次运行试试
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [11:22:57]
$ gcc aa.c

# root @ localhost in ~ [11:23:29]
$ ./a.out
char1: a
char2: b
ASCII of char: 97	98
</script></code></pre>

### 非阻塞式键盘监听
输入函数被执行的时候，线程因为等待IO会被挂起，只有等我们输入完成后，线程才会被唤醒，继续执行后面的代码，这种就是`阻塞式键盘监听`
现在我们设计一个小程序，该程序每隔500ms打印一行字符串，只有当我们按下ESC键后程序才会退出
如果用之前的方法，需要每次都按键才能使得程序继续运行，显然达不到我们的要求，这时候就需要`非阻塞式键盘监听`了
`kbhit()`函数用来检测键盘缓冲区是否有按键被按下，如果有则返回非0值，没有则返回0，`kbhit()`不会读取缓冲区的字符，字符还在缓冲区
注意，下面的程序只能在windows下(vs2017)运行，因为gcc没有`conio.h`头文件
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <conio.h>
#include <stdlib.h>
#include <windows.h>

int main() {
	char c;
	int i = 0;

	while (1) {
		if (kbhit()) {  // kbhit() 位于 <conio.h> 头文件
			c = getch();
			if (c == 27) {
				printf("The ESC key is pressed!\n");
				break;
			}
		}
		printf("No.%d\n", ++i);
		Sleep(500); // sleep 500ms, 位于 <windows.h> 头文件
	}

	system("pause");
	return 0;
}
</script></code></pre>

按下`ESC`键时，会跳出循环
<pre><code class="language-c line-numbers"><script type="text/plain">No.1
No.2
No.3
No.4
No.5
No.6
The ESC key is pressed!
请按任意键继续. . .
</script></code></pre>
