---
title: c语言 - 函数
date: 2017-07-03 08:33:00
categories:
- c
tags:
- c
keywords: c语言函数
---

> 
c语言 - 函数，函数声明，函数定义，形式参数和实际参数

<!-- more -->

## 函数定义
如果函数不需要参数，应该将其参数设为`void`，这样如果在别处调用该函数时传入了参数，在编译期间会报错，这是一个编程好习惯，虽然这并不是必须这么做，你也可以不加`void`，通常情况下也没什么不妥
函数可以有多个`return`语句，但是只有第一个`return`语句会被执行
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

void func(void){
    printf("func\n");
}

int main(){
    func();
    return 0;
}
</script></code></pre>

函数名在编译期间会被替换成函数的入口地址
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

void func1(void){}
void func2(void){}
void func3(void){}

int main(){
    printf("func1: %p, func2: %p, func3: %p, main: %p\n", func1, func2, func3, main);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [9:55:09]
$ gcc a.c

# root @ localhost in ~ [9:55:11]
$ ./a.out
func1: 0x40052d, func2: 0x400533, func3: 0x400539, main: 0x40053f

# root @ localhost in ~ [9:55:12]
$ ./a.out
func1: 0x40052d, func2: 0x400533, func3: 0x400539, main: 0x40053f

# root @ localhost in ~ [9:55:13]
$ ./a.out
func1: 0x40052d, func2: 0x400533, func3: 0x400539, main: 0x40053f
</script></code></pre>

## 形参和实参
函数定义中出现的参数称之为`形式参数`，可以看作是一个占位符
在函数调用时出现的参数称之为`实际参数`，此时，将`实参`的值赋给`形参`，也就是内存的拷贝
`形参`只有在函数被调用时分配内存，在定义函数时，不分配内存
注意，在调用函数过程中，`形参`和`实参`的值互不影响，因为它们拥有不同的内存
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

void swap(int m, int n){
    printf("(形参)交换前: m: %d, n: %d\n", m, n);
    int temp;
    temp = m;
    m = n;
    n = temp;
    printf("(形参)交换后: m: %d, n: %d\n", m, n);
}

int main(){
    int m=10, n=20;
    swap(m, n);
    printf("(实参)交换后: m: %d, n: %d\n", m, n);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [10:24:59]
$ gcc a.c

# root @ localhost in ~ [10:25:05]
$ ./a.out
(形参)交换前: m: 10, n: 20
(形参)交换后: m: 20, n: 10
(实参)交换后: m: 10, n: 20
</script></code></pre>

## 函数声明
在使用某个函数之前，我们应该先定义该函数
但是某些情况，我们想让函数在后面定义，这时候就需要函数声明
`函数声明`也称为`函数原型`，函数声明很简单，只是去掉大括号就行
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

void swap(int m, int n); //函数声明
/*
void swap(int, int); //也可以省略m, n
*/

int main(){
    int m=10, n=20;
    swap(m, n);
    printf("(实参)交换后: m: %d, n: %d\n", m, n);
    return 0;
}

//函数定义
void swap(int m, int n){
    printf("(形参)交换前: m: %d, n: %d\n", m, n);
    int temp;
    temp = m;
    m = n;
    n = temp;
    printf("(形参)交换后: m: %d, n: %d\n", m, n);
}
</script></code></pre>

## 关于函数
函数不能嵌套定义，但是可以嵌套调用，所谓嵌套调用就是可以在一个函数的定义或调用过程中出现对另一函数的调用
如果自己调用自己，那么就属于递归调用，c语言允许递归调用，但是递归调用逻辑结构不清晰，容易造成代码混乱，常被改写为循环

函数体外，除了预处理指令、变量的定义、类型的定义，不允许出现任何具有运算、逻辑处理能力的语句

`main()`函数是c程序的入口函数，是最先入栈的函数，也是最后出栈的函数，可以调用其他函数，但是不可以被其他函数调用

## main()函数
我们之前的main()函数，都是没有参数的，但是实际上，main()函数是可以接收命令行参数的
它有两种标准的原型：
<pre><code class="language-c"><script type="text/plain">int main();
int main(int argc, char *argv[])
</script></code></pre>

第一个参数是字符串的数目，第二个参数是一个指针数组

指针数组的第0个元素总是当前被执行的程序的文件名，所以，argc最小也为1

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(int argc, char *argv[]){
    printf("%d %s\n", argc, argv[0]);
    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [21:32:09]
$ gcc a.c

# root @ localhost in ~ [21:32:20]
$ ./a.out
1 ./a.out
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(int argc, char *argv[]){
    int i;
    for(i=0; i<5; i++){
        printf("%d: %s\n", i, argv[i]);
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [21:36:22]
$ ./a.out
0: ./a.out
1: (null)
2: USER=root
3: LOGNAME=root
4: HOME=/root

# root @ localhost in ~ [21:36:25]
$ ./a.out 1 2 3 4 5 6
0: ./a.out
1: 1
2: 2
3: 3
4: 4
</script></code></pre>

## 回调函数
函数指针可以作为某个函数的参数来使用，`回调函数`就是一个通过函数指针调用的函数
先看下面的例子：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

void setArrayElem(int *array, int len, int (*func)(void)){
    for(int i=0; i<len; i++){
        array[i] = func();
    }
}

int getNextRand(void){
    return rand();
}

int main(){
    int array[10], len = sizeof(array) / sizeof(int);
    setArrayElem(array, len, getNextRand);
    for(int i=0; i<len; i++){
        printf("%d ", array[i]);
    }
    printf("\n");
    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [16:41:49]
$ gcc a.c

# root @ localhost in ~ [16:41:50]
$ ./a.out
1804289383 846930886 1681692777 1714636915 1957747793 424238335 719885386 1649760492 596516649 1189641421
</script></code></pre>

被传入的函数，即`getNextRand()`，叫做`回调函数`
传入回调函数这个动作，即`setArrayElem(array, len, getNextRand);`，叫做`登记回调函数`
函数`setArrayElem()`叫做`中间函数`
中间函数的调用者，即`main()`函数，叫做`起始函数`

## 可变参数
不知道你发现没有，我们一直用的 printf()、scanf() 其实是参数个数可变的函数
所谓可变参数就是，能根据具体的需求接受可变数量的参数

printf() 原型为：`int printf(const char *fmt, ...);`
scanf() 原型为：`int scanf(const char *fmt, ...);`

除了第一个固定位置的参数`fmt`，就是`...`省略号了，这个省略号就表示可变参数

**变参函数实现原理**
C调用约定下可使用va_list系列变参宏实现变参函数，此处va意为variable-argument(可变参数)

先来认识一下va_list系列变参宏
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdarg.h>

void var_args_func(int count, ...){ // count为变参列表的长度
    va_list vl;     // 定义一个va_list类型的变量vl
    va_start(vl, count);    // 初始化变量，使其指向第一个可变参数，该宏的第二个参数是变参列表的前一个参数，即最后一个固定参数
    int value1 = va_arg(vl, int);   // 该宏返回变参列表中的当前变参值，并使其指向下一个变参，第二个参数是要返回的当前变参类型
    double value2 = va_arg(vl, double); // 依次取出变参列表中的参数
    char *value3 = va_arg(vl, char *);  // ...
    va_end(vl);     // 变参列表结束
    printf("value1: %d, value2: %.3f, value3: %s\n", value1, value2, value3);
}

int main(void){
    var_args_func(3, 100, 3.1415926d, "www.zfl9.com");
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [20:29:04]
$ gcc a.c

# root @ localhost in ~ [20:29:18]
$ ./a.out
value1: 100, value2: 3.142, value3: www.zfl9.com
</script></code></pre>

这实际上也不是真正的变参函数，我们实际上内定了参数为 int、double、char *

变参宏根据堆栈生长方向和参数入栈特点，从最靠近第一个可变参数的固定参数开始，依次获取每个可变参数的地址
变参宏的定义和实现因操作系统、硬件平台及编译器而异(但原理相似)

**函数参数入栈顺序**
我们知道，栈是由高地址向低地址延伸的，栈顶在内存的低地址，先入栈的变量，内存地址高于后入栈的变量
一般来说，c语言的函数参数入栈顺序是从右往左的，正好符合我们的思维习惯，参数也是从低到高的地址依次排列

但是实际上还是因操作系统、硬件平台、编译器而异，比如我的树莓派3b(armv7)上使用gcc编译就是从左往右入栈的
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

void func(int a, int b, int c){
    printf("%p, %p, %p\n", &a, &b, &c);
}

int main(){
    func(1, 2, 3);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ alarmpi in ~ [20:49:13]
$ gcc a.c

# root @ alarmpi in ~ [20:49:27]
$ ./a.out
0x7ea5f524, 0x7ea5f520, 0x7ea5f51c
</script></code></pre>

而且奇怪的是，我在CentOS7.3上使用gcc也是这种情况，而在VS2017下就相反了

**注意：va_arg(vl, data_type) 的 data_type 不能为下面这些数据类型:**
- `char`、`signed char`、`unsigned char`
- `short`、`signed short`、`unsigned short`
- `float`

因为在C语言中，调用者会对每个参数执行`默认实际参数提升(default argument promotions)`
- `float`类型提升为`double`
- `char`、`short`以及对应的`signed`、`unsigned`提升为`int`
- 如果`int`不能存储原值，则提升为`unsigned int`

**简单应用：利用变参函数，求一组数字的平均数**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdarg.h>

double average(int count, ...){
    va_list vl;
    va_start(vl, count);
    double sum = 0;
    for(int i=0; i<count; i++){
        sum += va_arg(vl, int);
    }
    va_end(vl);
    return sum / count;
}

int main(void){
    printf("%.3lf\n", average(5, 1, 2, 3, 4, 5));
    printf("%.3lf\n", average(3, 1, 2, 3));
    printf("%.3lf\n", average(2, 1, 2));
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ alarmpi in ~ [21:04:08]
$ gcc a.c

# root @ alarmpi in ~ [21:04:10]
$ ./a.out
3.000
2.000
1.500
</script></code></pre>

仔细想想，在使用printf()、scanf()及其家族函数时，我们明明没有像上面一样传入一个count这样存储参数个数的参数
而它们却能够准确无误的输出我们给的各种数据类型？

其实它是通过解析`const char *fmt`中的格式化字符串来判断的，有多少个`%`那就有多少个变参，然后再根据它们后面的数据类型，就可以知道它们的长度，进而取得它们的值了！
