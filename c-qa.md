---
title: c语言 - 疑惑解答
date: 2017-07-07 12:18:00
categories:
- c
tags:
- c
keywords: 变量名，函数名，数组名，符号表，内存溢出，内存泄露，野指针，堆栈，堆
---

> 
c语言 - 疑惑解答，栈和堆在线程之间是共享的还是独立的、什么是野指针，空指针，迷途指针、内存溢出和内存泄露的区别、变量名和地址之间的关系，动态链接库和静态链接库有什么区别？

<!-- more -->

## 同一个进程的不同线程之间不共享的部分是？
**在一个进程的线程共享堆区，而进程中的线程各自维持自己堆栈**

`线程`**共享**的环境包括：
- `进程代码段`
- `进程的公有数据`(利用这些共享的数据，线程很容易的实现相互之间的通讯)
- `进程打开的文件描述符`
- `信号的处理器`
- `进程的当前目录`
- `进程用户ID`
- `进程组ID`

相关：
- `线程ID`
每个线程都有自己的线程ID，这个ID在本进程中是唯一的，进程用此来标识线程
- `寄存器组的值`
由于线程间是并发运行的，每个线程有自己不同的运行线索
当从一个线程切换到另一个线程上时，必须将原有的线程的寄存器集合的状态保存，以便将来该线程在被重新切换到时能得以恢复
- `线程的堆栈`
**堆栈是保证线程独立运行所必须的**
线程函数可以调用函数，而被调用函数中又是可以层层嵌套的，所以线程必须拥有自己的函数堆栈，使得函数调用可以正常执行，不受其他线程的影响
- `错误返回码`
由于同一个进程中有很多个线程在同时运行，可能某个线程进行系统调用后设置了errno值，而在该线程还没有处理这个错误，另外一个线程就在此时被调度器投入运行，这样错误值就有可能被修改
所以，**不同的线程应该拥有自己的错误返回码变量**
- `线程的信号屏蔽码`
由于每个线程所感兴趣的信号不同，所以线程的信号屏蔽码应该由线程自己管理，但所有的线程都共享同样的信号处理器
- `线程的优先级`
由于线程需要像进程那样能够被调度，那么就必须要有可供调度使用的参数，这个参数就是线程的优先级

## 野指针 迷途指针
- `野指针`
某些编程语言允许`未初始化的指针`的存在，而这类指针即为`野指针`
除了static存储类的指针变量，任何指针变量刚被创建时不会自动成为NULL指针，它的缺省值是随机的，它会乱指一气
所以，指针变量在创建的同时应当被初始化，要么将指针设置为NULL，要么让它指向合法的内存
- `迷途指针(悬垂指针)`
**在计算机编程领域中，`迷途指针`与`野指针`指的是不指向任何合法的对象的指针**
当所指向的对象被释放或者收回，但是对该指针没有作任何的修改，以至于该指针仍旧指向已经回收的内存地址，此情况下该指针便称为`迷途指针(悬垂指针)`
若操作系统将这部分已经释放的内存重新分配给另外一个进程，而原来的程序重新引用现在的迷途指针，则将产生无法预料的后果
因为此时迷途指针所指向的内存现在包含的已经完全是不同的数据，通常来说，若原来的程序继续往迷途指针所指向的内存地址写入数据，这些和原来程序不相关的数据将被损坏，进而导致不可预料的程序错误
这种类型的程序错误，不容易找到问题的原因，通常会导致`段错误`(Linux系统中)和`一般保护错误`(Windows系统中)

防止野指针，迷途指针的错误：
- 指针变量在定义的时候应该初始化为合法的内存或者`NULL`
- `malloc/free`、`new/delete`应该成对出现，不要混用
- 申请堆内存之后，用if判断指针是否为`NULL`
- 使用完指针后，应立即将指针置为`NULL`

## 内存溢出 内存泄漏 内存越界 缓冲区溢出 栈溢出
- `内存溢出`
内存溢出就是你要求分配的内存超出了系统能给你的，系统不能满足你的需求，于是产生溢出
- `内存泄漏`
内存泄漏就是你用malloc/new向系统申请了内存空间，你用完之后并没有将其归还(free/delete)
如果你把这块内存的地址也弄丢了，那么你也不能再使用这块内存，系统也无法将其分配给需要的程序
就这样越积越多，最终导致系统的内存不够用
- `内存越界`
何谓内存访问越界，简单的说，你向系统申请了一块内存，在使用这块内存的时候，超出了你申请的范围
- `栈溢出`
栈溢出是缓冲区溢出的一种，缓冲区溢出是指当计算机向缓冲区内填充数据位数时超过了缓冲区本身的容量溢出的数据覆盖在合法数据上
操作系统所使用的缓冲区又被称为`堆栈`，在各个操作进程之间，指令会被临时储存在`堆栈`中，`堆栈`也会出现缓冲区溢出

一个简单的内存泄漏例子
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

int main(){
    while(1){
        malloc(1);
    }
    return 0;
}
</script></code></pre>

## 变量名相关
本质上，变量名、数组名、函数名都是地址的助记符，但是一般我们认为`变量名`是数据本身，而`函数名`、`数组名`则表示的是地址，前者是函数的入口地址，后者是数组第0个元素的地址

编译器会把所有`标识符`(变量名、函数名、数组名等)及它们对应的`数据类型`、`地址`、`作用域`等信息存放到一个叫做`符号表`的数据结构中

变量名只存在于我们编写的源代码中，在编译、链接之后，都会被替换为相应的地址，所以在内存中并不存在所谓的变量名

## 静态库 动态库
从C语言源文件 `main.c` 到可执行文件 `main` 转变的过程：
- `预处理`：执行预处理指令，如头文件包含、宏定义、宏展开、条件编译等，`gcc -E main.c -o main.i`
- `编译`：将源代码编译成汇编文件，`gcc -S main.i -o main.s`
- `汇编`：将汇编文件转换成二进制机器码，`gcc -c main.s -o main.o`
- `链接`：将目标文件、静态链接库、动态链接库链接成可执行文件，`gcc main.o -o main`

ELF文件格式有以下三种类型：
- `可重定位的对象文件(Relocatable file)`：目标文件`main.o`
- `可执行的对象文件(Executable file)`：可执行文件`main`
- `可被共享的对象文件(Shared object file)`：动态链接库`libxxx.so`

`节(section)`和`段(segment)`
一个ELF文件从`连接器(Linker)`的角度看，是一些`节(section)`的集合
从程序`加载器(Loader)`的角度看，它是一些`段(Segments)`的集合
ELF格式的`程序`和`共享库`具有相同的结构，只是段的集合和节的集合上有些不同

`静态链接库`和`动态链接库`
- `静态链接库`：一般命名为`libxxx.a`，是一组目标文件(如: `main.o`)的打包，也就是`归档(Archive)`，在链接时，会将静态库拷贝进可执行文件，程序运行时，与静态库无关联
- `动态链接库`：一般命名为`libxxx.so.major.minor`，major 是主版本号，minor是副版本号，也可以没有版本号，一般也会建立一个没有版本号的软链接到全名的文件
和可执行文件的结构类似，相对于静态库，在链接时，并不会将其拷贝进可执行文件中，而只是做些标记，等到程序运行时，动态的加载所需的模块

**如何创建自己的动态库、静态库？**
说明：`test.h`函数库的头文件、`test_1.c, test_2.c, test_3.c`三个函数、`main.c`主函数
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [14:40:56]
$ ll
total 20K
-rw-r--r-- 1 root root  89 Jul  9 14:40 main.c
-rw-r--r-- 1 root root  89 Jul  9 14:36 test_1.c
-rw-r--r-- 1 root root  89 Jul  9 14:37 test_2.c
-rw-r--r-- 1 root root  89 Jul  9 14:37 test_3.c
-rw-r--r-- 1 root root 121 Jul  9 14:36 test.h

# root @ localhost in ~ [14:40:57]
$ cat test.h
#ifndef __TEST_H
#define __TEST_H

extern void test_1(void);
extern void test_2(void);
extern void test_3(void);

#endif

# root @ localhost in ~ [14:41:07]
$ cat test_1.c
#include <stdio.h>
#include "test.h"

void test_1(void){
    printf("func: test_1\n");
}

# root @ localhost in ~ [14:41:11]
$ cat test_2.c
#include <stdio.h>
#include "test.h"

void test_2(void){
    printf("func: test_2\n");
}

# root @ localhost in ~ [14:41:14]
$ cat test_3.c
#include <stdio.h>
#include "test.h"

void test_3(void){
    printf("func: test_3\n");
}

# root @ localhost in ~ [14:41:17]
$ cat main.c
#include "test.h"

int main(){
    test_1();
    test_2();
    test_3();
    return 0;
}

# root @ localhost in ~ [14:41:26]
$ gcc -c test_1.c test_2.c test_3.c

# root @ localhost in ~ [14:42:44]
$ ll
total 32K
-rw-r--r-- 1 root root   89 Jul  9 14:40 main.c
-rw-r--r-- 1 root root   89 Jul  9 14:36 test_1.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_1.o
-rw-r--r-- 1 root root   89 Jul  9 14:37 test_2.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_2.o
-rw-r--r-- 1 root root   89 Jul  9 14:37 test_3.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_3.o
-rw-r--r-- 1 root root  121 Jul  9 14:36 test.h

# root @ localhost in ~ [14:42:48]
$ ar crs libtest.a test_1.o test_2.o test_3.o

# root @ localhost in ~ [14:43:28]
$ ll
total 40K
-rw-r--r-- 1 root root 4.7K Jul  9 14:43 libtest.a
-rw-r--r-- 1 root root   89 Jul  9 14:40 main.c
-rw-r--r-- 1 root root   89 Jul  9 14:36 test_1.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_1.o
-rw-r--r-- 1 root root   89 Jul  9 14:37 test_2.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_2.o
-rw-r--r-- 1 root root   89 Jul  9 14:37 test_3.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_3.o
-rw-r--r-- 1 root root  121 Jul  9 14:36 test.h

# root @ localhost in ~ [14:43:30]
$ ar t libtest.a
test_1.o
test_2.o
test_3.o

# root @ localhost in ~ [14:43:52]
$ mkdir lib

# root @ localhost in ~ [14:44:12]
$ mv libtest.a lib/

# root @ localhost in ~ [14:44:19]
$ ll
total 32K
drwxr-xr-x 2 root root   23 Jul  9 14:44 lib
-rw-r--r-- 1 root root   89 Jul  9 14:40 main.c
-rw-r--r-- 1 root root   89 Jul  9 14:36 test_1.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_1.o
-rw-r--r-- 1 root root   89 Jul  9 14:37 test_2.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_2.o
-rw-r--r-- 1 root root   89 Jul  9 14:37 test_3.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_3.o
-rw-r--r-- 1 root root  121 Jul  9 14:36 test.h

# root @ localhost in ~ [14:44:22]
$ gcc test_1.c test_2.c test_3.c -fPIC -shared -o libtest.so

# root @ localhost in ~ [14:46:11]
$ ll
total 40K
drwxr-xr-x 2 root root   23 Jul  9 14:44 lib
-rwxr-xr-x 1 root root 8.0K Jul  9 14:46 libtest.so
-rw-r--r-- 1 root root   89 Jul  9 14:40 main.c
-rw-r--r-- 1 root root   89 Jul  9 14:36 test_1.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_1.o
-rw-r--r-- 1 root root   89 Jul  9 14:37 test_2.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_2.o
-rw-r--r-- 1 root root   89 Jul  9 14:37 test_3.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_3.o
-rw-r--r-- 1 root root  121 Jul  9 14:36 test.h

# root @ localhost in ~ [14:46:13]
$ mv libtest.so lib/

# root @ localhost in ~ [14:46:17]
$ ll
total 32K
drwxr-xr-x 2 root root   41 Jul  9 14:46 lib
-rw-r--r-- 1 root root   89 Jul  9 14:40 main.c
-rw-r--r-- 1 root root   89 Jul  9 14:36 test_1.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_1.o
-rw-r--r-- 1 root root   89 Jul  9 14:37 test_2.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_2.o
-rw-r--r-- 1 root root   89 Jul  9 14:37 test_3.c
-rw-r--r-- 1 root root 1.5K Jul  9 14:42 test_3.o
-rw-r--r-- 1 root root  121 Jul  9 14:36 test.h

# root @ localhost in ~ [14:46:18]
$ ll lib
total 16K
-rw-r--r-- 1 root root 4.7K Jul  9 14:43 libtest.a
-rwxr-xr-x 1 root root 8.0K Jul  9 14:46 libtest.so

# root @ localhost in ~ [14:46:21]
$ rm -fr test_*

# root @ localhost in ~ [14:46:42]
$ ll
total 8.0K
drwxr-xr-x 2 root root  41 Jul  9 14:46 lib
-rw-r--r-- 1 root root  89 Jul  9 14:40 main.c
-rw-r--r-- 1 root root 121 Jul  9 14:36 test.h

# root @ localhost in ~ [14:46:43]
$ mkdir include

# root @ localhost in ~ [14:46:47]
$ mv test.h include

# root @ localhost in ~ [14:46:51]
$ ll
total 4.0K
drwxr-xr-x 2 root root 20 Jul  9 14:46 include
drwxr-xr-x 2 root root 41 Jul  9 14:46 lib
-rw-r--r-- 1 root root 89 Jul  9 14:40 main.c

# root @ localhost in ~ [14:46:52]
$ ll include
total 4.0K
-rw-r--r-- 1 root root 121 Jul  9 14:36 test.h

# root @ localhost in ~ [14:46:54]
$ ll lib
total 16K
-rw-r--r-- 1 root root 4.7K Jul  9 14:43 libtest.a
-rwxr-xr-x 1 root root 8.0K Jul  9 14:46 libtest.so

# root @ localhost in ~ [14:46:57]
$ gcc main.c -I include -L lib -l test -o main_share

# root @ localhost in ~ [14:49:02]
$ gcc main.c -I include -L lib -Wl,-Bstatic -l test -Wl,-Bdynamic -o main_static

# root @ localhost in ~ [14:50:20]
$ ll
total 28K
drwxr-xr-x 2 root root   20 Jul  9 14:46 include
drwxr-xr-x 2 root root   41 Jul  9 14:46 lib
-rw-r--r-- 1 root root   89 Jul  9 14:40 main.c
-rwxr-xr-x 1 root root 8.4K Jul  9 14:49 main_share
-rwxr-xr-x 1 root root 8.5K Jul  9 14:50 main_static

# root @ localhost in ~ [14:50:26]
$ ldd main_share
	linux-vdso.so.1 =>  (0x00007ffda77e5000)
	libtest.so => not found
	libc.so.6 => /lib64/libc.so.6 (0x00007f8ae4f08000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f8ae52dd000)

# root @ localhost in ~ [14:50:31]
$ ldd main_static
	linux-vdso.so.1 =>  (0x00007ffdc935a000)
	libc.so.6 => /lib64/libc.so.6 (0x00007ff8ad79a000)
	/lib64/ld-linux-x86-64.so.2 (0x00007ff8adb6e000)

# root @ localhost in ~ [14:50:35]
$ ./main_static
func: test_1
func: test_2
func: test_3

# root @ localhost in ~ [14:50:50]
$ ./main_share
./main_share: error while loading shared libraries: libtest.so: cannot open shared object file: No such file or directory

# root @ localhost in ~ [14:50:55] C:127
$ vim /etc/ld.so.conf.d/zfl.conf

# root @ localhost in ~ [14:51:10]
$ cat /etc/ld.so.conf.d/zfl.conf
/root/lib

# root @ localhost in ~ [14:51:18]
$ ldconfig

# root @ localhost in ~ [14:51:23]
$ ./main_share
func: test_1
func: test_2
func: test_3
</script></code></pre>

如果同时存在静态库和动态库，则优先使用动态库

`-I`参数指定自定义`include path`，默认搜索路径：
- `.`、`/usr/include`、`/usr/local/include`、`/usr/lib/gcc/x86_64-redhat-linux/4.8.2/include`
- `C_INCLUDE_PATH`、`CPLUS_INCLUDE_PATH`环境变量指定的路径

`-L`参数指定自定义`library path`，注意是**编译期间库的查找路径**，默认搜索路径(优先级依次降低)：
- `-L`指定的路径
- `LIBRARY_PATH`环境变量指定的路径
- 默认路径`/lib`、`/lib64`、`/usr/lib`、`/usr/lib64`、`/usr/local/lib`、`/usr/local/lib64`

`-l`参数指定使用的库名，`libxxx.so`或`libxxx.a`，只需要给出`xxx`就行

`-static`参数强制gcc使用静态链接，如果只有动态库，没有静态库，会链接失败

`ld.so`是Unix或类Unix系统上的`动态链接器`，常见的有两个变体：
- `ld.so`针对 a.out 格式的二进制可执行文件
- `ld-linux.so`针对 ELF 格式的二进制可执行文件

当应用程序需要使用动态链接库里的函数时，由ld.so负责加载。注意这是**运行期间库的查找路径**
- `LD_LIBRARY_PATH`环境变量指定的路径
- `/etc/ld.so.cache`缓存文件，配置文件为`/etc/ld.so.conf`、`/etc/ld.so.conf.d/*.conf`，更新缓存的命令是`ldconfig`
- `/lib`、`/usr/lib`默认目录

创建静态链接库：
`gcc -c test_1.c test_2.c test_3.c`
`ar csr libtest.a test_1.o test_2.o test_3.o`

查看静态库包含的obj文件：
`ar t libtest.a`

释放静态库中的obj文件：
`ar x libtest.a`

创建动态链接库：
`gcc test_1.c test_2.c test_3.c -fPIC -shared -o libtest.so`

ldd打印程序或动态库所依赖的共享库列表：
`ldd libtest.so`、`ldd main_share`

ldconfig更新/etc/ld.so.cache缓存文件：
`ldconfig`
`ldconfig -p`打印当前缓存的动态库信息

编译时使用动态链接(default)：
`gcc main.c -I include -L lib -l test -o main_share`

编译时使用静态链接：
`gcc main.c -I include -L lib -l test -o main_static -static`，系统某些库不是静态库，所以会链接失败

编译时使用静态链接libtest.a库、动态链接其他库(混合连接动态库和静态库)：
`gcc main.c -I include -L lib -Wl,-Bstatic -l test -Wl,-Bdynamic -o main_static`

查看elf文件的信息：
`readelf`

查看obj文件的信息：
`objdump`

查看符号表信息：
`nm`

查看 text/data/bss 占用的大小：
`size`

查看二进制文件中所有可打印字符：
`strings`

二进制文件查看器：
`xxd`、`hexdump`

gcc编译时指定不同版本的c语言：
`-std=c89`      c89
`-std=c99`      c99
`-std=c11`      c11
`-std=gnu89`    带gcc扩展的c89，默认选项
`-std=gnu99`    带gcc扩展的c99
`-std=gnu11`    带gcc扩展的c11
