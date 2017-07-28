---
title: c语言 - 变量与数据类型
date: 2017-07-01 17:38:00
categories:
- c
tags:
- c
keywords: c语言变量与数据类型
---

> 
c语言 - 变量与数据类型，全局变量，局部变量，常量，基本数据类型，各种运算符，存储类，自动/强制类型转换

<!-- more -->

## 全局变量和局部变量
`全局变量`: 定义在所有函数之外的变量，属于static存储类，默认初始值为0
`局部变量`: 函数的形参，函数体内定义的变量，代码块内定义的变量，属于auto存储类，需手动初始化，否则为垃圾值
注意：函数体外只能进行全局变量的初始化，不能进行赋值运算
<pre><code class="language-c"><script type="text/plain">/*
int i;      正确，i = 0;
*/

/*
int i;
i = 10;     错误，不能进行赋值
*/

/*
int i = 10;     正确，可以进行初始化
*/

/*
int i = 10;
int j = i;      错误
*/

/*
int i = 10 * 10;    正确，编译器会自动优化为 int i = 100;
*/

/*
int i = 10;
int j = i + 10;     错误，不能进行运算操作
*/
</script></code></pre>

## 常量
定义常量有两种方式
- `#define` 预处理器
- `const` 关键字

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#define PI 3.14

int main(){
    const int I = 1;
    printf("PI: %.2f, I: %d\n", PI, I);
}
</script></code></pre>

## 基本数据类型
|描述|数据类型|
|:-:|:-:|
|字符|`char`|
|短整数|`short`|
|整数|`int`|
|长整数|`long`|
|单精度浮点数|`float`|
|双精度浮点数|`double`|
|无类型|`void`|

每种数据类型占用的内存大小，这是在64位的linux下占用的大小，在其他系统会有所不同
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    printf("char: %d, short: %d, int: %d, long: %d, float: %d, double: %d, void: %d\n", sizeof(char), sizeof(short), sizeof(int), sizeof(long), sizeof(float), sizeof(double), sizeof(void));
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [12:54:08]
$ gcc type.c

# root @ localhost in ~ [12:54:10]
$ ./a.out
char: 1, short: 2, int: 4, long: 8, float: 4, double: 8, void: 1
</script></code></pre>

### NULL 空值
`NULL`在`stdio.h`实际上是`#define NULL ((void *) 0)` (C语言中)
在C++中则是`#define NULL 0`

### 浮点数
<pre><code class="language-c line-numbers"><script type="text/plain">float a = 1.0f;
double b = 1.0d;
long double ld = 1.0l;  //长浮点数
// 如果不指定后缀f，则默认为double型；
</script></code></pre>

### 无符号数
`char` `short` `int` `long`都是有符号的，首位用来存储符号位；
如果不需要使用负数，则可以使用无符号数，只要在前面加上`unsigned`即可；
如`unsigned char` `unsigned short`、`unsigned int`、`unsigned long`，其中`unsigned int`可以简写为`unsigned`

## 运算符
- 算术运算符:
`+` `-` `*` `/` `%`取余 `++`自增 `--`自减
- 关系运算符
`==` `!=` `>` `>=` `<` `<=`，结果为布尔值true/false
- 逻辑运算符
`&&`与 `||`或 `!`非
- 位运算符
假设A=60, B=13; 二进制表示如下:
A = 0011 1100
B = 0000 1101
A & B = 0000 1100
A | B = 0011 1101
A ^ B = 0011 0001
~A  = 1100 0011
A << 2 = 1111 0000
A >> 2 =  0000 1111
- 赋值运算符
`=` `+=` `-=` `*=` `/=` `%=` `>>=` `<<=` `&=` `|=` `^=`
- 其他
`sizeof(int)` 返回变量大小
`&` 返回变量地址
`?:` 条件表达式， 如`int a = (b > 10) ? 100 : 0`，如果b大于10，则a为100，否则为0

- 运算符优先级
从高到低：
后缀，一元，乘除，加减，移位，关系，相等，位与，位异或，位或，逻辑与，逻辑或，条件运算符?:，赋值

### 自增，自减
- ++在前面叫做`前自增`（例如 ++a）；前自增先进行自增操作，再进行其他操作；
- ++在后面叫做`后自增`（例如 a++）；后自增先进行其他操作，再进行自增操作；
- 自减同理。

例如:
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a=10, b=20, c=30, d=40, aa, bb, cc, dd;
    aa = a++;
    bb = ++b;
    cc = c--;
    dd = --d;
    printf("a=%d aa=%d\nb=%d bb=%d\nc=%d cc=%d\nd=%d dd=%d\n", a, aa, b, bb, c, cc, d, dd);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [14:50:17]
$ gcc test.c

# root @ localhost in ~ [14:50:19]
$ ./a.out
a=11 aa=10
b=21 bb=21
c=29 cc=30
d=39 dd=39
</script></code></pre>

## 存储类
存储类定义C程序中变量/函数的范围（可见性）和生命周期。

### auto
所有局部变量的默认存储类，只能修饰局部变量
<pre><code class="language-c line-numbers"><script type="text/plain">{
    int i=10;
    auto char c='a';
}
</script></code></pre>

### static
static 指示编译器在程序的生命周期内保持`局部变量`的存在，而不需要在每次它进入和离开作用域时进行创建和销毁
因此，使用 static 修饰局部变量可以在函数调用之间保持局部变量的值
static 也可以应用于`全局变量`和`函数`，`全局变量`属于static存储类，当 static 修饰`全局变量`或`函数`时，会将其作用域限制在声明它的文件内(默认的作用域是工程内的所有文件)
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

void print(){
    static int i=1;
    printf("i = %d\n", i++);
}

int main(){
    print();
    print();
    print();
    print();
    print();
    return 0;
}
</script></code></pre>

编译并运行
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [17:17:36]
$ gcc a.c

# root @ localhost in ~ [17:17:39]
$ ./a.out
i = 1
i = 2
i = 3
i = 4
i = 5
</script></code></pre>

### register
用于定义存储在寄存器中而不是 RAM 中的局部变量
这意味着变量的最大尺寸等于寄存器的大小（通常是一个词）
且不能对它应用`&`运算符（因为它没有内存位置）
用了register修饰符，并不意味着该变量就存储在寄存器中，这取决于硬件和实现的限制
<pre><code class="language-c line-numbers"><script type="text/plain">{
    register int i=1;
}
</script></code></pre>

### extern
放在函数或者变量前，以标示函数或者变量的定义在别的文件(也可以在当前文件)，用来声明全局变量或函数

a.c
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    extern int i; // 可简写为 extern i;
    printf("i = %d\n", i);
    return 0;
}
</script></code></pre>

b.c
<pre><code class="language-c line-numbers"><script type="text/plain">int i = 10;
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [15:47:05]
$ gcc a.c b.c

# root @ localhost in ~ [15:47:09]
$ ./a.out
i = 10
</script></code></pre>

> 
函数默认都隐式声明为extern，所以可以省略extern

a.c
<pre><code class="language-c line-numbers"><script type="text/plain">extern void print(); //可省略extern

int main(){
    print();
    return 0;
}
</script></code></pre>

b.c
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

void print(){
    printf("print\n");
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [15:56:55]
$ gcc a.c b.c

# root @ localhost in ~ [15:56:59]
$ ./a.out
print
</script></code></pre>

> 
注意`变量`的`声明`、`定义`的区别，`函数`的`声明`、`定义`很容易区分
**`定义`**: 分配内存空间，如`int i`，分配4个字节的内存
**`声明`**: 不需要分配内存空间，如`extern int i`，这只是告诉编译器，`i`这个变量已经在别的文件定义了，不用为其分配内存，直接用就行
**函数或者变量的声明都是为了提前使用，如果不需要提前使用，没有提前声明的必要性**

<pre><code class="language-c line-numbers"><script type="text/plain">// 函数的声明
void print();
int max(int m, int n);
int min(int, int);

// 函数的定义
void print(){
    printf("hello, world!\n");
}

// 变量的声明
extern int i;
extern i;   // 可省略数据类型

// 变量的定义
int i;

// 变量的赋值
i = 10;

// 变量的初始化(定义的同时进行赋初值)
int i = 10;
</script></code></pre>

## 变量初始值
- static存储类的变量(全局变量，static变量)，其初始值为0；
- auto存储类的变量，系统并不会对其进行初始化，它的值是垃圾值，所以在使用前一定要进行手动初始化

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

char c;
short s;
int i;
long l;
float f;
double d;

int main(){
    static int static_main;
    int a1 = 10;
    int a2 = 20;
    int a = a1 * a2;
    int auto_main;
    printf("char: %c\nshort: %hd\nint: %d\nlong: %ld\nfloat: %f\ndouble: %lf\nstatic_main: %d\nauto_main: %d\n", c, s, i, l, f, d, static_main, auto_main);
    return 0;
}
</script></code></pre>

编译并运行
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [17:30:37]
$ gcc a.c

# root @ localhost in ~ [17:30:38]
$ ./a.out
char:
short: 0
int: 0
long: 0
float: 0.000000
double: 0.000000
static_main: 0
auto_main: -2099247248
</script></code></pre>

## 数据类型转换
### 自动类型转换
1. 若参与运算的数据类型不同，则先转换成同一类型，然后进行运算。
2. 转换按数据长度增加的方向进行，以保证精度不降低。例如int型和long型运算时，先把int量转成long型后再进行运算。
3. 所有的浮点运算都是以双精度进行的，即使仅含float单精度量运算的表达式，也要先转换成double型，再作运算。
4. char型和short型参与运算时，必须先转换成int型。
5. 在赋值运算中，赋值号两边的数据类型不同时，需要把右边表达式的类型将转换为左边变量的类型。如果右边表达式的数据类型长度比左边长时，将丢失一部分数据，这样会降低精度。

### 强制类型转换
<pre><code class="language-c line-numbers">#include &lt;stdio.h&gt;

int main(){
    int a=10, b=3;
    double result;
    result = (double) a / b;
    printf(&quot;(double) a / b = %f\n&quot;, result);
    result = (double) (a / b);
    printf(&quot;(double) (a / b) = %f\n&quot;, result);
    return 0;
}
</code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [17:45:07]
$ gcc a.c

# root @ localhost in ~ [17:45:09]
$ ./a.out
(double) a / b = 3.333333
(double) (a / b) = 3.000000
</script></code></pre>

注意，(double)的优先级更高，所以第一个结果是这样算的：
先将a转换为double类型，然后再和b相除，这时，b也自动转换为double类型，除出来的结果是3.333333...
第二个结果是先将a和b相除，因为都是int类型，所以后面的小数部分直接舍去了，然后再转换为double类型，结果就是3.000000
