---
title: c语言 - gdb、cgdb调试器
date: 2017-07-19 17:11:00
categories:
- c
tags:
- c
keywords: C语言 gdb cgdb调试器
---

> 
c语言 - gdb、cgdb调试器

<!-- more -->

## gdb
gdb是gnu软件系统中的标准调试器，gdb中的命令有很多，但是通常只要掌握其中的10来个命令就足够我们日常的调试工作了

**安装gdb**
- CentOS/RHEL：`yum -y install gdb`
- ArchLinux：`pacman -S gdb`

配置alias别名：`alias gdb='gdb -q'`，不显示gdb版本等信息

好了，gdb装好之后，我们先来写一个阶乘的代码：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

int factorial(int n){
    if(n == 0 || n == 1){
        return 1;
    }else{
        return n * factorial(n-1);
    }
}

int main(int argc, char *argv[]){
    int result, n = atoi(argv[1]);
    result = factorial(n);
    printf("%d! = %d\n", n, result);
    return 0;
}
</script></code></pre>

编译，注意加上-g选项，添加debug信息
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [17:25:43]
$ gcc a.c -g
a.c: In function ‘main’:
a.c:12:14: warning: unused parameter ‘argc’ [-Wunused-parameter]
 int main(int argc, char *argv[]){
              ^

# root @ localhost in ~ [17:27:13]
$ ll
total 16K
-rw-r--r-- 1 root root  310 Jul 19 16:33 a.c
-rwxr-xr-x 1 root root 9.7K Jul 19 17:27 a.out
</script></code></pre>

我们先来正常执行一下程序：
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [17:27:18]
$ ./a.out 5
5! = 120
</script></code></pre>

启用gdb调试：
`l`：显示源代码，默认显示10行
`l 10`：显示第10行附近的代码
`l main`：显示main()函数的代码
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [17:31:00]
$ gdb a.out
Reading symbols from /root/a.out...done.
(gdb)
(gdb) l
4	int factorial(int n){
5	    if(n == 0 || n == 1){
6	        return 1;
7	    }else{
8	        return n * factorial(n-1);
9	    }
10	}
11
12	int main(int argc, char *argv[]){
13	    int result, n = atoi(argv[1]);
(gdb)
14	    result = factorial(n);
15	    printf("%d! = %d\n", n, result);
16	    return 0;
17	}
(gdb)
Line number 18 out of range; a.c has 17 lines.
(gdb) l 1
1	#include <stdio.h>
2	#include <stdlib.h>
3
4	int factorial(int n){
5	    if(n == 0 || n == 1){
6	        return 1;
7	    }else{
8	        return n * factorial(n-1);
9	    }
10	}
(gdb) l main
7	    }else{
8	        return n * factorial(n-1);
9	    }
10	}
11
12	int main(int argc, char *argv[]){
13	    int result, n = atoi(argv[1]);
14	    result = factorial(n);
15	    printf("%d! = %d\n", n, result);
16	    return 0;
(gdb) l factorial
1	#include <stdio.h>
2	#include <stdlib.h>
3
4	int factorial(int n){
5	    if(n == 0 || n == 1){
6	        return 1;
7	    }else{
8	        return n * factorial(n-1);
9	    }
10	}
</script></code></pre>

设置断点，所谓断点，就是当程序运行到断点处时，会暂停运行，方便调试
`b 13`：在13行处设置断点
`b 14`：在14行处设置断点
`b main`：在main()函数入口处设置断点
`i b`：查看断点信息
`d 2`：删除断点2
`d`：删除所有断点
<pre><code class="language-c line-numbers"><script type="text/plain">(gdb) l main
7	    }else{
8	        return n * factorial(n-1);
9	    }
10	}
11
12	int main(int argc, char *argv[]){
13	    int result, n = atoi(argv[1]);
14	    result = factorial(n);
15	    printf("%d! = %d\n", n, result);
16	    return 0;
(gdb) b 13
Breakpoint 1 at 0x4005bd: file a.c, line 13.
(gdb) b 14
Breakpoint 2 at 0x4005d3: file a.c, line 14.
(gdb) i b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000004005bd in main at a.c:13
2       breakpoint     keep y   0x00000000004005d3 in main at a.c:14
(gdb) d 2
(gdb) i b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000004005bd in main at a.c:13
</script></code></pre>

然后我们就可以运行程序了：
`r`：运行程序
`r 5`：传递参数5
`r 10 20 30`：传递参数10 20 30
也可以使用`set args 10 20 30`设置参数，`show args`显示当前参数，`set args`清空参数
<pre><code class="language-c line-numbers"><script type="text/plain">(gdb) l main
7	    }else{
8	        return n * factorial(n-1);
9	    }
10	}
11
12	int main(int argc, char *argv[]){
13	    int result, n = atoi(argv[1]);
14	    result = factorial(n);
15	    printf("%d! = %d\n", n, result);
16	    return 0;
(gdb) b main
Breakpoint 4 at 0x4005bd: file a.c, line 13.
(gdb) r
Starting program: /root/a.out main

Breakpoint 4, main (argc=2, argv=0x7fffffffe408) at a.c:13
13	    int result, n = atoi(argv[1]);
</script></code></pre>

因为我们在13行的地方设置了断点，所以执行run后，在13行处会暂停(注意：13行还没有执行)
这时候，我们可以使用`p`命令来查看变量的值：
`p result`：查看result的值
`p n`：查看n的值
`p argv[1]`：查看argv[1]的值
<pre><code class="language-c line-numbers"><script type="text/plain">(gdb) r
Starting program: /root/a.out 3

Breakpoint 1, main (argc=2, argv=0x7fffffffe408) at a.c:13
13	    int result, n = atoi(argv[1]);
Missing separate debuginfos, use: debuginfo-install glibc-2.17-157.el7_3.4.x86_64
(gdb) p result
$1 = 0
(gdb) p n
$2 = 0
(gdb) p argv[1]
$3 = 0x7fffffffe6b4 "3"
</script></code></pre>

单步执行代码：
`s`：step，若有函数调用时，进入函数
`n`：next，若有函数调用时，执行函数，而不进入函数内部
`c`：继续运行(直到下一个断点或结束)
<pre><code class="language-c line-numbers"><script type="text/plain">(gdb) p result
$1 = 0
(gdb) p n
$2 = 0
(gdb) p argv[1]
$3 = 0x7fffffffe6b4 "3"
(gdb) s
14	    result = factorial(n);
(gdb) l factorial
1	#include <stdio.h>
2	#include <stdlib.h>
3
4	int factorial(int n){
5	    if(n == 0 || n == 1){
6	        return 1;
7	    }else{
8	        return n * factorial(n-1);
9	    }
10	}
(gdb) p result
$4 = 0
(gdb) p n
$5 = 3
(gdb) s
factorial (n=3) at a.c:5
5	    if(n == 0 || n == 1){
(gdb) p n
$6 = 3
(gdb) s
8	        return n * factorial(n-1);
(gdb) p n
$7 = 3
(gdb) s
factorial (n=2) at a.c:5
5	    if(n == 0 || n == 1){
(gdb) p n
$8 = 2
(gdb) s
8	        return n * factorial(n-1);
(gdb) p n
$9 = 2
(gdb) s
factorial (n=1) at a.c:5
5	    if(n == 0 || n == 1){
(gdb) p n
$10 = 1
(gdb) s
6	        return 1;
(gdb) p n
$11 = 1
(gdb) s
10	}
(gdb) s
10	}
(gdb) s
10	}
(gdb)
main (argc=2, argv=0x7fffffffe408) at a.c:15
15	    printf("%d! = %d\n", n, result);
(gdb) p result
$12 = 6
(gdb) p n
$13 = 3
(gdb) c
Continuing.
3! = 6
[Inferior 1 (process 13024) exited normally]
</script></code></pre>

## gdb常用参数
**生成调试信息**
一般来说GDB主要用来调试c/c++，要调试c/c++，必须在gcc编译时，加上参数-g
如果没有-g参数，变量名、函数名、数组名等地址助记符都会被替换为运行时的内存地址

**启动gdb的方法**
- `gdb program`：program也就是你的执行文件，一般在当前目录下
- `gdb program core`：用gdb同时调试一个运行程序和core文件，core是非法执行后core dump产生的文件
- `gdb program 1234`：如果你的程序是一个服务程序，那么你可以指定这个服务程序运行时的PID，gdb会自动attach上去，并调试它，program应该在PATH环境变量中搜索得到

**程序运行时参数**
- `set args 1 2 3`：设置运行时参数为1 2 3
- `show args`：显示设置好的运行参数
- `set args`：清空之前的运行时参数
- `r`：不指定运行参数
- `r 1 2 3`：设置运行时参数为1 2 3

**工作目录**
- `cd`：切换工作目录
- `pwd`：打印当前工作目录
- `shell <command>`：执行shell命令

**程序的输入输出**
- `info terminal`：显示终端模式
- `run > out.file`：输出重定向至文件out.file

**设置断点**
**简单断点**
- `b 10`：在第10行设置断点
- `b func`：在func入口处设置断点

**多文件设置断点**
C++中可以使用class::function或function(type,type)格式来指定函数名
如果有名称空间，可以使用namespace::class::function或者function(type,type)格式来指定函数名

- `break filename:linenum`：在源文件filename的linenum行处停住
- `break filename:function`：在源文件filename的function函数的入口处停住
- `break class::function或function(type,type)`：在类class的function函数的入口处停住
- `break namespace::class::function`：在名称空间为namespace的类class的function函数的入口处停住

**查询所有断点**
- `info b`：查询所有断点

**观察点**
- `watch 变量名`：为该变量名设置一个观察点，当值发生变化时停住程序
- `rwatch 变量名`：当变量被读取时，停住程序
- `awatch 变量名`：当变量被读取或修改时，停住程序
- `info watch`：查看当前设置的所有观察点

**条件断点**
为断点设置一个条件，我们使用if关键词，后面跟其断点条件
并且，条件设置好后，我们可以用condition命令来修改断点的条件
只有break和watch命令支持if，catch目前暂不支持if
condition与break if类似，只是condition只能用在已存在的断点上

- `b test.c:8 if intValue == 5`：在test.c第8行设置一个条件断点
- `condition bnum expression`：修改断点号为bnum的停止条件为expression
- `condition bnum`：清除断点号为bnum的停止条件
- `ignore bnum count`：表示忽略断点号为bnum的停止条件count次

**维护断点**
- `clear`：清除所有的已定义的停止点
- `clear function`：清除所有设置在函数上的停止点
- `clear linenum`：清除所有设置在指定行上的停止点
- `clear filename:linenum`：清除所有设置在指定文件的指定行上的停止点
- `delete [breakpoints] [range...]`：删除指定的断点，breakpoints为断点号，如果不指定断点号，则表示删除所有的断点，range表示断点号的范围（如：3-7）
- `disable [breakpoints] [range...]`：disable所指定的停止点，breakpoints为停止点号，如果什么都不指定，表示disable所有的停止点
- `enable [breakpoints] [range...]`：enable所指定的停止点，breakpoints为停止点号
- `enable [breakpoints] once range…`：enable所指定的停止点一次，当程序停止后，该停止点马上被GDB自动disable
- `enable [breakpoints] delete range…`：enable所指定的停止点一次，当程序停止后，该停止点马上被GDB自动删除

**为停止点设定运行命令**
我们可以使用GDB提供的command命令来设置停止点的运行命令
也就是说，当运行的程序在被停止住时，我们可以让其自动运行一些别的命令，这很有利行自动化调试
`commands [bnum]`：设置断点bnum的命令列表，当程序被该断点停住时，gdb会依次运行命令列表中的命令
例如:
<pre><code class="language-c line-numbers"><script type="text/plain">break foo if x>0
commands
printf "x is %d", x
continue
end
</script></code></pre>

断点设置在函数foo中，断点条件是x>0，如果程序被断住后，也就是，一旦x的值在foo函数中大于0，GDB会自动打印出x的值，并继续运行程序
如果你要清除断点上的命令序列，那么只要简单的执行一下commands命令，并直接在打个end就行了

**调试代码**
- `run`：运行程序，可简写为r
- `next`：单步跟踪，函数调用当作一条简单语句执行，可简写为n
- `step`：单步跟踪，函数调进入被调用函数体内，可简写为s
- `finish`：退出函数
- `until`：在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体,可简写为u
- `continue`：继续运行程序，可简写为c
- `stepi`或`si`、`nexti`或`ni`：单步跟踪一条机器指令，一条程序代码有可能由数条机器指令完成，stepi和nexti可以单步执行机器指令
- `info program`：来查看程序的是否在运行，进程号，被暂停的原因

**查看运行时的数据**
- `print`：打印变量、字符串、表达式等的值，可简写为p
- `p count`：打印count的值
- `p cou1+cou2+cou3`：打印表达式值
- `p *array@10`：array是一个数组，@左边是数组第0个元素的值，@右边是数组长度

`p/<fmt>`：fmt是输出的格式，同`x`命令的fmt
print接受一个表达式，GDB会根据当前的程序运行的数据来计算这个表达式，表达式可以是当前程序运行中的const常量、变量、函数等内容，但是GDB不能使用程序中定义的宏

**程序变量**
在gdb中，可以随时查看这三种变量的值：全局变量(所有文件可见)、静态全局变量(当前文件可见)、局部变量(当前作用域中可见)
当局部变量和全局变量重名时，优先显示局部变量的值，想查看全局变量的值可以使用`::`，如`p 'main.c'::var`、`p 'module.c'::func::var`等

**自动显示**
你可以设置一些自动显示的变量，当程序停住时，或是在你单步跟踪时，这些变量会自动显示，相关的GDB命令是display
- `display expr`
- `display/fmt expr`
- `display/fmt addr`

expr是一个表达式，fmt表示显示的格式，addr表示内存地址，当你用display设定好了一个或多个表达式后，只要你的程序被停下来，GDB会自动显示你所设置的这些表达式的值

`info display`：查看display设置的自动显示的信息

`undisplay dnums…`
`delete display dnums…`
删除自动显示，dnums意为所设置好了的自动显示的编号
如果要同时删除几个，编号可以用空格分隔，如果要删除一个范围内的编号，可以用减号表示（如：2-5）

`disable display dnums…`
`enable display dnums…`
disable和enalbe不删除自动显示的设置，而只是让其失效和恢复

**历史记录**
- `show val 10`：查看编号10附近的历史记录
- `show val`：查看最后10个历史记录

**改变程序的运行**
**修改变量的值**
一种方法是用print：`p x=4`：修改变量x的值为4，但是这种方式很容易与gdb的参数冲突，推荐使用`set var x=4`这种方式

**跳转执行**
一般来说，被调试程序会按照程序代码的运行顺序依次执行
GDB提供了乱序执行的功能，也就是说，GDB可以修改程序的执行顺序，可以让程序执行随意跳跃
这个功能可以由GDB的jump命令来完成：
- `jump linespec`：指定下一条语句的运行点，可以是文件的行号，可以是file:line格式，可以是+num这种偏移量格式，表示下一条运行语句从哪里开始
- `jump *address`：这里的是代码行的内存地址

注意，jump命令不会改变当前的程序栈中的内容
所以，当你从一个函数跳到另一个函数时，当函数运行完返回时进行弹栈操作时必然会发生错误
可能结果还是非常奇怪的，甚至于产生程序Core Dump，所以最好是同一个函数中进行跳转

熟悉汇编的人都知道，程序运行时，eip寄存器用于保存当前代码所在的内存地址
所以，jump命令也就是改变了这个寄存器中的值，于是，你可以使用`set $pc`来更改跳转执行的地址
如：`set $pc = 0×485`

**强制函数返回**
如果你的调试断点在某个函数中，并还有语句没有执行完
你可以使用return命令强制函数忽略还没有执行的语句并返回
`return`、`return expression`
使用return命令取消当前函数的执行，并立即返回，如果指定了，那么该表达式的值会被认作函数的返回值

**强制调用函数**
`call expr`：表达式中可以一是函数，以此达到强制调用函数的目的，并显示函数的返回值，如果函数返回值是void，那么就不显示
`print expr`：另一个相似的命令也可以完成这一功能：print，print后面可以跟表达式，所以也可以用他来调用函数，print和call的不同是，如果函数返回void，call则不显示，print则显示函数返回值，并把该值存入历史数据中

**显示源代码**
GDB可以打印出所调试程序的源代码，当然，在程序编译时一定要加上 –g 的参数，把源程序信息编译到执行文件中，不然就看不到源程序了
当程序停下来以后，GDB会报告程序停在了那个文件的第几行上
你可以用list命令来打印程序的源代码，默认打印10行

`list linenum`：显示行号linenum附近10行的源程序
`list function`：显示函数名为function的函数的源程序
`list`：显示当前行后面的源程序
`list -`：显示当前行前面的源程序

一般是打印当前行的上5行和下5行，默认是10行，当然，你也可以定制显示的范围，使用下面命令可以设置一次显示源程序的行数

`set listsize count`：设置一次显示源代码的行数
`show listsize`：查看当前listsize的设置

**调试已运行的进程**
1、在UNIX下用ps查看正在运行的程序的PID，然后用gdb PID process-id格式挂接正在运行的程序
2、先用gdb关联上源代码，并进行gdb，在gdb中用attach process-id命令来挂接进程的PID，并用detach来取消挂接的进程

**线程**
如果你程序是多线程的话，你可以定义你的断点是否在所有的线程上，或是在某个特定的线程
`break linespec thread threadno`
`break linespec thread threadno if …`
linespec指定了断点设置在的源程序的行号
threadno指定了线程的ID，注意，这个ID是GDB分配的，你可以通过"info threads"命令来查看正在运行程序中的线程信息，如果你不指定"thread threadno"则表示你的断点设在所有线程上面

你还可以为某线程指定断点条件，如：
`break frik.c:13 thread 28 if bartab > lim`
当你的程序被GDB停住时，所有的运行线程都会被停住，这方便你你查看运行程序的总体情况
而在你恢复程序运行时，所有的线程也会被恢复运行，那怕是主进程在被单步调试时

**栈信息**
当你调用了一个函数，函数的地址、函数参数、函数内的局部变量都会被压入`栈(stack)`中
`bt`：查看当前函数调用栈的所有信息
`bt n`：当n为正整数时，只打印栈顶上n层的信息；当n为负整数时，只打印栈底下n层的信息

如果你要查看某一层的信息，你需要在切换当前的栈，一般来说，程序停止时，最顶层的栈就是当前栈，如果你要查看栈下面层的详细信息，首先要做的是切换当前栈
`frame n`：n是一个从0开始的整数，是栈中的层编号；比如：frame 0，表示栈顶，frame 1，表示栈的第二层
`frame addr`：地址为addr的栈帧
`up n`：表示向栈的上面移动n层，可以不打n，表示向上移动一层
`down n`：表示向栈的下面移动n层，可以不打n，表示向下移动一层

上面的命令，都会打印出移动到的栈层的信息
如果你不想让其打出信息，你可以使用这三个命令：
`select-frame`对应于`frame`命令
`up-silently n`对应于`up`命令
`down-silently n`对应于`down`命令

查看当前栈层的信息，你可以用以下GDB命令：
`frame`或`f`：栈的层编号，当前的函数名，函数参数值，函数所在文件及行号，函数执行到的语句
`info frame`：打印当前栈的信息

**信号**
信号是一种软中断，是一种处理异步事件的方法
一般来说，操作系统都支持许多信号，尤其是UNIX，比较重要应用程序一般都会处理信号
UNIX定义了许多信号，比如SIGINT表示中断字符信号，也就是Ctrl+C的信号；SIGBUS表示硬件故障的信号；SIGCHLD表示子进程状态改变信号；SIGKILL表示终止程序运行的信号，等等

调试程序的时候处理信号: `handle signal [keywords...]`
signal可以以SIG开头或不以SIG开头，可以用定义一个要处理信号的范围（如：SIGIO-SIGKILL，表示处理从 SIGIO信号到SIGKILL的信号，其中包括SIGIO，SIGIOT，SIGKILL三个信号），也可以使用关键字all来标明要处理所有的信号
一旦被调试的程序接收到信号，运行程序马上会被GDB停住，以供调试

keywords列表如下:
- `nostop`：当被调试的程序收到信号时，GDB不会停住程序的运行，但会打出消息告诉你收到这种信号
- `stop`：当被调试的程序收到信号时，GDB会停住你的程序
- `print`：当被调试的程序收到信号时，GDB会显示出一条信息
- `noprint`：当被调试的程序收到信号时，GDB不会告诉你收到信号的信息
- `pass`：当被调试的程序收到信号时，GDB不处理信号，这表示，GDB会把这个信号交给被调试程序处理
- `nopass`：当被调试的程序收到信号时，GDB不会让被调试程序来处理这个信号

`info signals`
`info handle`
查看有哪些信号在被GDB检测中

**捕获异常**
当event发生时，停住程序，event可以是下面的内容：
`catch throw`：一个C++抛出的异常
`catch catch`：一个C++捕捉到的异常

**指定源文件搜索路径**
某些时候，用-g编译过后的执行程序中只是包括了源文件的名字，没有路径名
GDB提供了可以让你指定源文件的路径的命令，以便GDB进行搜索
`dir dirname …`：加一个源文件路径到当前路径的前面，如果你要指定多个路径，UNIX下你可以使用":"，Windows下你可以使用";"
`dir`：清除所有的自定义的源文件搜索路径信息
`show dir`：显示定义了的源文件搜索路径

**查看内存中的值**
`x/<n/f/u> <addr>`：查看内存中的值
其中n为正整数，表示需要显示的内存单元个数
f为显示的格式，如果该地址的值是字符串，可以用s
u为一个单元的长度，默认是4bytes，可用b表示单字节、h表示双字节、w表示四字节、g表示八字节
addr表示一个内存地址，可用&取地址符

其中f的格式可以为：
c字符、s字符串、d十进制、f浮点数、t二进制、o八进制、x十六进制、u无符号型十六进制、a地址、i指令

n/f/u 三个参数可以任意组合，可以都没有，也可以都指定

**其他命令**
`i args`：查看参数信息
`i locals`：查看当前函数的局部变量

`for <regex>`：向前搜索源代码，正则方式
`rev <regex>`：向后搜索

`disass func`：查看func()函数的汇编代码

`file EXEC_FILE`：加载执行文件

`show version`：查看gdb版本信息

`q`：退出gdb shell

## gdb调试多进程
默认设置下，在调试多进程程序时gdb只会调试主进程；
gdb7以上的版本(gdb --version)支持多进程调试，只需要设置好`follow-fork-mode`(fork追踪模式)以及`detach-on-fork`(指示GDB在fork之后是否断开某个进程的调试)即可；

这两个参数的设置命令分别是：
`set follow-fork-mode [parent|child]`，默认值`parent`
`set detach-on-fork [on|off]`，默认值`on`
两者结合起来构成了GDB的调试模式；
<pre><code class="language-bash line-numbers"><script type="text/plain">follow-fork-mode  detach-on-fork    说明
    parent              on          GDB默认的调试模式：只调试主进程
    child               on          只调试子进程
    parent              off         同时调试两个进程，gdb跟主进程，子进程block在fork位置
    child               off         同时调试两个进程，gdb跟子进程，主进程block在fork位置
</script></code></pre>

首先进行一个名词解释`inferior`，GDB将每个被调试程序的执行状态的记录结构称为一个`inferior`；
一般情况下一个`inferior会对应一个进程`，当然嵌入式平台可能有不同情况；
`inferior`有时候会在进程没有启动的情况下就存在(这在命令中会有所体现，后面会详细介绍)，每个`inferior`会有不同的地址空间，并且一个`inferior`里面可以包含多个线程；

`info inferiors`：显示GDB调试的所有inferior
前面有*的是当前inferior，直接发GDB命令控制的就是当前inferior；

`inferior infno`：设置infno号inferior为当前inferior；

`add-inferior [ -copies n ] [ -exec executable ]`：增加n个inferior并且其执行程序为executable
如果不指定n则只增加一个inferior；如果不指定executable，则执行程序留空，增加后可使用file命令重新指定执行程序；
注意：这时候创建的inferior其关联的进程并没有启动；

`clone-inferior [ -copies n ] [ infno ]`：增加n个其执行程序为指定infno号的inferior
如果不指定n则只增加一个inferior；如果不指定infno，则指定跟当前inferior一样的执行程序；
注意：这个命令只是克隆了另一个infno的执行文件名称，和clone这个系统调用并没什么相关性(个人觉得用copy更不容易产生歧义)；

`remove-inferior infno`：删除一个infno号inferior
如果inferior在运行，则不能删除inferior，所以在删除以前需要先kill或者detach这个inferior；

`detach inferior infno`：detach掉infno号inferior
注意这个inferior仍然存在，可以再次用run等命令执行它，如果想删除结构需要用remove-inferior命令；

`kill inferior infno`：kill掉infno号inferior
注意这个inferior仍然存在，可以再次用run等命令执行它，如果想删除结构需要用remove-inferior命令；

`set print inferior-events on|off`
`show print inferior-events`
这个选项用来打开和关闭inferior状态的提示信息；

`set detach-on-fork on|off`
`show detach-on-fork`
这个选项用来控制fork的时候，是否detach掉父进程或者子进程(至于具体哪个就由下面的命令决定)，默认值是打开，所以想同时调试父进程和子进程的就需要关闭这个选项；

`set follow-fork-mode parent|child`
`show follow-fork-mode`
这个选项用来设置fork的时候，GDB将继续调试父进程或者子进程，上面的命令也介绍了，如果`detach-on-fork`选项打开，不被调试的那个进程将被detach；

`set follow-exec-mode new|same`
`show follow-exec-mode`
当发生exec的时候，如果这个选项是same(默认值)，因为父进程已经退出，所以自动在执行exec的inferior上控制子进程；
如果选项是new，则新建一个inferior给执行起来的子进程，而父进程的inferior仍然保留，当前保留的inferior的程序状态是没有执行；

`set schedule-multiple on|off`
`show schedule-multiple`
这个选项类似于多线程调试里的`set scheduler-locking`选项；
当选项是off(默认值)的时候，GDB发出执行指令的时候，只有当前inferior会执行；
而当选项是on的时候，GDB发出执行命令后，全部状态是执行状态的inferior都会执行；
注意，如果scheduler-locking选项设置为lock的时候，即使schedule-multiple设置为on，也只有当前进程的当前线程会执行；

`maint info program-spaces`：显示当前GDB一共管理了多少地址空间

## gdb调试多线程
`info threads`：显示当前可调试的所有线程，每个线程会有一个GDB为其分配的ID，后面操作线程的时候会用到这个ID；前面有*的是当前调试的线程；

`thread <ID>`：切换当前调试的线程为指定ID的线程；

`break test.c:123 thread all`：在所有线程中相应的行上设置断点；

`thread apply ID1 ID2 command`：让一个或者多个线程执行GDB命令command；

`thread apply all command`：让所有被调试线程执行GDB命令command；

`set scheduler-locking off|on|step`：
默认情况下，在使用step或者continue命令调试当前线程时，其他线程也是同时执行的；
怎么只让被调试的线程执行呢？通过这个命令就可以实现这个需求；
`off`：不锁定任何线程，也就是所有线程都执行，这是默认值；
`on`：只有当前被调试的线程会执行；
`step`：在单步的时候，除了next过一个函数的情况以外，只有当前线程会执行；

## cgdb
可能是我用惯了oh-my-zsh和vim，严重依赖语法高亮
前面的gdb都是黑底白字，代码也没有高亮，后来找到一个叫cgdb的调试器
其实还是调用的gdb，但是它包装的更好，支持vim风格的操作，还支持代码高亮

**安装cgdb**
<pre><code class="language-bash line-numbers"><script type="text/plain">git clone https://github.com/cgdb/cgdb.git
cd cgdb
./autogen.sh
./configure --prefix=/usr/local/cgdb
make && make install

vim /etc/profile
PATH=$PATH:/usr/local/cgdb/bin

source /etc/profile
</script></code></pre>

**cgdb简要介绍**
1. cgdb分为上下两栏，上面是一个类似vim的窗口，下面是gdb shell
2. 通过ESC可进入vim模式，i键进入gdb shell
3. 在vim窗口中，按o键，打开文件对话框窗口，ctrl+w切换布局
4. 选中某行后，按空格键设置断点，再按一次取消断点
5. -号将vim窗口缩小、=号则相反
