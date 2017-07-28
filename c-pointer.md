---
title: c语言 - 指针
date: 2017-07-08 04:56:00
categories:
- c
tags:
- c
keywords: C语言 指针
---

> 
c语言 - 指针

<!-- more -->

## 什么是指针
计算机中所有的数据都必须放在内存中，不同类型的数据占用的字节数不一样
例如 int 占用4个字节，char 占用1个字节
为了正确地访问这些数据，必须知道它们在内存中的准确位置，这个位置就叫做`地址(Address)`或`指针(Pointer)`
内存地址通常使用16进制数表示，地址从0开始依次增加，对于32位环境，程序所能使用的内存为4G，最小的地址为0，最大的地址为0xFFFFFFFF

下面的代码演示了如何输出一个地址：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a = 10;
    int b[5] = {0, 1, 2, 3, 4};
    char str1[] = "https://www.zfl9.com";
    char *str2 = "https://www.zfl9.com";
    printf("a: %p\nb: %p\nstr1: %p\nstr2: %p\n", &a, b, str1, str2);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [11:41:41]
$ ./a.out
a: 0x7ffcafdb1fe4
b: 0x7ffcafdb1fd0
str1: 0x7ffcafdb1fb0
str2: 0x400650
</script></code></pre>

`%p`表示输出一个地址，也可以用`%#x`，`&`是取地址符，因为数组名本身就表示数组的首地址，所以不需要`&`

**指针变量**
数据在内存中的地址也称为指针，如果一个变量存储了一份数据的指针，我们就称它为`指针变量`

在C语言中，允许用一个变量来存放指针，这种变量称为指针变量。
指针变量的值就是某份数据的地址，这样的一份数据可以是数组、字符串、函数，也可以是另外的一个普通变量或指针变量

定义指针变量
`type *ptr;`、`type *ptr = value;`
`*`表示变量`ptr`是一个指针变量，而`type`是该指针指向的数据类型

如：`int *p`，p是一个指针变量，指向的数据类型为int，是一个整型指针

注意，指针变量和普通变量没有什么区别，都是变量，只不过存放的数据类型不同而已

对于`int a`，它定义了一个变量`a`，数据的类型为`int`
对于`int a[6]`，它定义了一个变量`a`，是一个拥有6个元素的数组，元素的数据类型为`int`，数组的数据类型为`int [6]`
对于`int *p`，它定义了一个变量`p`，数据的类型为`int *`

给指针变量赋值
<pre><code class="language-c line-numbers"><script type="text/plain">int a = 10;
int *p = &a;
</script></code></pre>

指针变量需要的是一个地址，所以需要使用`&`取出变量`a`的地址，并将其赋给变量`p`

同时，指针变量的值可以随时改变，和普通的变量一样，可以多次赋值
<pre><code class="language-c line-numbers"><script type="text/plain">int a = 10, b = 20, c = 30;
int *p = &a;
p = &b;
p = &c;
</script></code></pre>

通过指针变量取得数据
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a = 10;
    int *p = &a;
    printf("addr: %p, var: %d\n", p, *p);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [12:09:20]
$ ./a.out
addr: 0x7fffadfc0fd4, var: 10
</script></code></pre>

`*`是指针运算符，用来取得某个地址上的数据
使用指针是间接获取数据，要经过两步运算，而通过变量名是直接获取数据

指针除了可以获取数据，还能修改数据
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a = 10;
    int *p = &a;
    *p = 100;
    printf("addr: %p, var: %d\n", p, *p);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [12:13:03]
$ ./a.out
addr: 0x7ffedc87fe64, var: 100
</script></code></pre>

通过指针交换两个变量的值
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a = 10, b = 20, temp;
    printf("swap before: a = %d, b = %d\n", a, b);
    int *pa = &a, *pb = &b;
    temp = *pa;
    *pa = *pb;
    *pb = temp;
    printf("swap afer: a = %d, b = %d\n", a, b);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [13:31:27]
$ gcc a.c

# root @ localhost in ~/test [13:31:29]
$ ./a.out
swap before: a = 10, b = 20
swap afer: a = 20, b = 10
</script></code></pre>

## 指针变量的运算
指针变量保存的是一个地址，本质上是一个整数，可以进行部分运算，如：加法，减法，比较等
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char a = 'a', *pa = &a;
    int b = 10, *pb = &b;
    double c = 3.14, *pc = &c;
    printf("pa: %p, pb: %p, pc: %p\n", pa, pb, pc);
    pa++; pb++; pc++;
    printf("pa: %p, pb: %p, pc: %p\n", pa, pb, pc);
    pa--; pb--; pc--;
    printf("pa: %p, pb: %p, pc: %p\n", pa, pb, pc);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [13:41:20] C:4
$ gcc a.c

# root @ localhost in ~/test [13:41:22]
$ ./a.out
pa: 0x7fff4574be77, pb: 0x7fff4574be70, pc: 0x7fff4574be68
pa: 0x7fff4574be78, pb: 0x7fff4574be74, pc: 0x7fff4574be70
pa: 0x7fff4574be77, pb: 0x7fff4574be70, pc: 0x7fff4574be68
</script></code></pre>

可以看出，pa、pb、pc每次加1，它们的地址分别加1、4、8，正好是char、int、double类型的长度
而每次减1，它们的地址分别减1、4、8，也正好是char、int、double类型的长度

看来指针偏移量与指针所指的数据类型相关，刚好等于数据类型的长度，而不仅仅是简单的算数加减，这就使得指针的加减运算有了实际意义

不过对于指向普通变量的指针，对其进行加减运算没有什么意义，因为我们并不知道它们前后都是什么数据
但是对于指向数组的指针，由于数组的元素时连续存储的，这时候就可以用指针来访问元素了

除了加减，还可以对指针变量进行比较，也就是比较它们的值，也就是数据的地址
不可以对指针变量进行乘法、除法、取余等运算，没有实际意义，编译器也会报错

## 数组指针
数组指针：也就是指向数组的指针
数组(Array)是一系列相同数据类型的数据的集合，每一份数据叫做一个数组元素(Element)
数组中的所有元素在内存中是连续排列的，整个数组占用的是一块内存

在定义数组的时候，需要给出数组名和数组长度，数组名在有些时候可以认为是一个指针，它指向数组的第0个元素，指向数组的首地址
**注意，数组和指针绝不等价，数组是另一种数据类型**，具体请戳[数组和指针的区别](https://www.zfl9.com/c-array.html#数组什么时候会转换为指针)

下面的例子演示了通过指针来访问数组：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a[5] = {0, 1, 2, 3, 4};
    int i, len = sizeof(a) / sizeof(int);
    for(i=0; i<len; i++){
        printf("index: %d, var: %d\n", i, *(a + i));
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [14:03:03]
$ gcc a.c

# root @ localhost in ~/test [14:03:05]
$ ./a.out
index: 0, var: 0
index: 1, var: 1
index: 2, var: 2
index: 3, var: 3
index: 4, var: 4
</script></code></pre>

`*(a + i)`等价于`a[i]`

也可以定义一个指针，指向数组，通过指针访问数组
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a[5] = {0, 1, 2, 3, 4};
    int *p = a;
    int i, len = sizeof(a) / sizeof(int);
    // int i, len = sizeof(p) / sizeof(int); 错误，sizeof(p)求得的只是一个指针变量的长度，而不是数组的长度
    for(i=0; i<len; i++){
        printf("index: %d, var: %d\n", i, *(p + i));
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [14:07:27]
$ gcc a.c

# root @ localhost in ~/test [14:07:29]
$ ./a.out
index: 0, var: 0
index: 1, var: 1
index: 2, var: 2
index: 3, var: 3
index: 4, var: 4
</script></code></pre>

注意，当数组名为第0个元素的指针时，这个指针是一个指针常量，不能改变该指针自身的值，但是可以修改指针指向的数据
比如，我们可以使用指针，通过自增来遍历数组元素
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a[5] = {0, 1, 2, 3, 4};
    int *p = a;
    int i;
    for(i=0; i<5; i++){
        printf("%d ", *(p++));
    }
    printf("\n");
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [14:12:33]
$ gcc a.c

# root @ localhost in ~/test [14:12:34]
$ ./a.out
0 1 2 3 4
</script></code></pre>

但是换成数组名就不行了，因为它是一个指针常量，是一个常量，指向的地址不能改变，下面的代码编译就会报错
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a[5] = {0, 1, 2, 3, 4};
    int i;
    for(i=0; i<5; i++){
        printf("%d ", *(a++));
    }
    printf("\n");
    return 0;
}
</script></code></pre>

假设指针p指向数组a的第n个元素，那`*p++`，`*++p`，`(*p)++`分别代表什么？
对于`*p++`，`++`优先级比`*`高，等价于`*(p++)`，由于是后自增，所以先进行其他操作，最后进行自增操作，所以是先取得第n个元素的值，然后再将指针p指向第n+1个元素
对于`*++p`，等价于`*(++p)`，前自增，先将p指向第n+1个元素，然后再取第n+1个元素的值
对于`(*p)++`，先取得第n个元素的值，然后将第n个元素的值加1

## 字符串指针
字符串指针：也就是指向字符串的指针

由于c语言没有专门的字符串类型，一般都是通过字符数组来存储字符串
字符数组也是数组，也可以用一个指针来指向这个数组

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <string.h>

int main(){
    char str[] = "https://www.zfl9.com";
    char *p = str;
    int i, len = strlen(str);
    for(i=0; i<len; i++){
        printf("%c", str[i]);
    }
    printf("\n");
    for(i=0; i<len; i++){
        printf("%c", *(str + i));
    }
    printf("\n");
    for(i=0; i<len; i++){
        printf("%c", p[i]);
    }
    printf("\n");
    for(i=0; i<len; i++){
        printf("%c", *(p + i));
    }
    printf("\n");
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [14:40:20]
$ gcc a.c

# root @ localhost in ~/test [14:40:21]
$ ./a.out
https://www.zfl9.com
https://www.zfl9.com
https://www.zfl9.com
https://www.zfl9.com
</script></code></pre>

c语言还支持另外一种表示字符串的方法，就是直接用指针指向一个字符串，叫做`字符串常量`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char str1[] = "hello, world!";
    char *str2 = "www.zfl9.com";
    printf("%s\n", str1);
    printf("%s\n", str2);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [14:44:08]
$ gcc a.c

# root @ localhost in ~/test [14:44:23]
$ ./a.out
hello, world!
www.zfl9.com
</script></code></pre>

字符串的所有字符在内存中都是连续存储的，和数组一样
注意，字符数组和字符串常量的区别，字符数组的元素的值是可以改变的，而字符串常量的值是不能改变的
因为字符数组存放在内存的栈区(局部变量)或全局数据区(全局变量)，拥有读写权限
而字符串常量中的字符串存放在内存的字符串常量区域，常量区域只有读的权限，不能写入

比如，下面的代码尝试修改字符串常量的值，这会发生段错误
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char str1[] = "hello, world!";
    char *str2 = "www.zfl9.com";
    printf("%s\n", str1);
    printf("%s\n", str2);

    str1[0] = 'H';
    printf("%s\n", str1);

    str2[0] = 'W';
    printf("%s\n", str2);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [14:51:20]
$ gcc a.c

# root @ localhost in ~/test [14:51:28]
$ ./a.out
hello, world!
www.zfl9.com
Hello, world!
[1]    76302 segmentation fault  ./a.out
</script></code></pre>

再次强调，字符串常量是字符串本身不能被修改，但是指向它的字符指针可以改变指向，指向别的数据
所以，如果需要经常修改字符串，那么应该选择字符数组，如果不需要经常变更字符串的值，就选择字符串常量

## 数组灵活多变的访问方式
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char str[20] = "www.zfl9.com";

    char *s1 = str;
    char *s2 = str+2;

    char c1 = str[4];
    char c2 = *str;
    char c3 = *(str+4);
    char c4 = *str+2;
    char c5 = (str+1)[5];

    int num1 = *str+2;
    long num2 = (long)str;
    long num3 = (long)(str+2);

    printf("  s1 = %s\n", s1);
    printf("  s2 = %s\n", s2);

    printf("  c1 = %c\n", c1);
    printf("  c2 = %c\n", c2);
    printf("  c3 = %c\n", c3);
    printf("  c4 = %c\n", c4);
    printf("  c5 = %c\n", c5);

    printf("num1 = %d\n", num1);
    printf("num2 = %ld\n", num2);
    printf("num3 = %ld\n", num3);

    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [14:58:28]
$ ./a.out
  s1 = www.zfl9.com
  s2 = w.zfl9.com
  c1 = z
  c2 = w
  c3 = z
  c4 = y
  c5 = l
num1 = 121
num2 = 140727275501696
num3 = 140727275501698
</script></code></pre>

分析：
s1 是指向数组 str 第 0 个元素的指针，所以输出 www.zfl9.com
s2 是指向数组 str 第 2 个元素的指针，所以输出 w.zfl9.com
c1 是数组 str 第 4 个元素的值，也就是 z
c2 是数组 str 第 0 个元素的值，也就是 w
c3 是数组 str 第 4 个元素的值，也就是 z
c4 是数组 str 第 0 个元素的值加2，也就是 w 的 ASCII 码加2，也就是字符 y
c5 是先计算 (str+1)，也就是指向数组 str 的第 1 个元素，然后转换为 *(str + 1 + 5)，也就是取得第 6 个元素的值，为 l
num1 是第 0 个元素的值，也就是字符 w 的 ASCII 码加2，为 121
num2 是数组 str 第 0 个元素的地址
num3 是数组 str 第 2 个元素的地址

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char str[20] = {0};
    int i;
    for(i=0; i<10; i++){
        str[i] = 97 + i; // 97为字符a的ASCII码值
    }
    printf("%s\n", str);
    printf("%s\n", str+2);
    printf("%c\n", str[2]);
    printf("%c\n", (str+2)[2]);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [15:28:43]
$ gcc a.c

# root @ localhost in ~/test [15:28:44]
$ ./a.out
abcdefghij
cdefghij
c
e
</script></code></pre>

`(str+2)[2]`相当于`*(str+2+2) == *(str+4) == str[4] == 'e'`

## 指针变量作为函数参数
在C语言中，函数的参数不仅可以是整数、小数、字符等具体的数据，还可以是指向它们的指针
用指针变量作函数参数可以将函数外部的地址传递到函数内部，使得在函数内部可以操作函数外部的数据，并且这些数据不会随着函数的结束而被销毁

像数组、字符串、动态分配的内存等都是一系列数据的集合，没有办法通过一个参数全部传入函数内部，只能传递它们的指针，在函数内部通过指针来影响这些数据集合

有的时候，对于整数、小数、字符等基本类型数据的操作也必须要借助指针，一个典型的例子就是交换两个变量的值

如果不使用指针，就不能改变函数外的变量的值
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

void swap(int m, int n){
    int temp;
    temp = m;
    m = n;
    n = temp;
}

int main(){
    int m = 10, n = 20;
    printf("before: m = %d, n = %d\n", m, n);
    swap(m, n);
    printf("after: m = %d, n = %d\n", m, n);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [15:35:50]
$ gcc a.c

# root @ localhost in ~/test [15:35:52]
$ ./a.out
before: m = 10, n = 20
after: m = 10, n = 20
</script></code></pre>

而通过指针就能交换
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

void swap(int *p1, int *p2){
    int temp;
    temp = *p1;
    *p1 = *p2;
    *p2 = temp;
}

int main(){
    int m = 10, n = 20;
    printf("before: m = %d, n = %d\n", m, n);
    swap(&m, &n);
    printf("after: m = %d, n = %d\n", m, n);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [15:37:35]
$ gcc a.c

# root @ localhost in ~/test [15:37:36]
$ ./a.out
before: m = 10, n = 20
after: m = 20, n = 10
</script></code></pre>

**用数组作函数参数**
数组是一组数据的集合，无法通过参数将它们一次性传递到函数内部，如果希望在函数内部操作数组，就应该传递数组指针
下面的例子中，max() 函数用于查找数组中值最大的元素
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int max(int *array, int len){
    int i, max = array[0]; // 假定 max = array[0]
    for(i=1; i<len; i++){
        if(array[i] > max){
            max = array[i];
        }
    }
    return max;
}

int main(){
    int array[6], i, maxVal;
    int len = sizeof(array) / sizeof(int);
    printf("enter 6 nums: ");
    for(i=0; i<len; i++){
        scanf("%d", array + i);
    }
    maxVal = max(array, len);
    printf("max: %d\n", maxVal);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [15:50:14]
$ gcc a.c

# root @ localhost in ~/test [15:50:16]
$ ./a.out
enter 6 nums: 12 55 30 8 93 27
max: 93
</script></code></pre>

参数 array 仅仅是一个数组指针，在函数内部无法通过这个指针获得数组的长度，必须传入一个数组长度的参数

函数 `int max(int *array, int len)` 也可以写成 `int max(int array[6], int len)` 或者 `int max(int array[], int len)`
看似好像传入了整个数组，其实无论那种方式都不会创建一个数组，最后都会转换为一个`int *`类型的数组指针，所以还是乖乖用指针的方式吧

为什么C语言不允许传递整个数组的元素呢？
这是考虑到效率问题，由于数组的元素个数并没有限制，可能很小也可能很大
而参数传递的本质实际上是一次赋值的过程，也就是内存的拷贝，对于基本数据类型和一些其他的类型，数据量也就几个字节，拷贝很快
而要使拷贝一个很长的数组，既浪费时间又浪费空间，所以为了不影响效率，C语言不允许传递数组所有的元素

## 指针作为函数的返回值
C语言允许函数的返回值是一个指针(地址)，我们将这样的函数称为`指针函数`
下面的例子定义了一个函数 longStr()，用于返回两个字符串中较长的那个
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <string.h>

char *longStr(char *str1, char *str2){
    int len1 = strlen(str1), len2 = strlen(str2);
    return len1 > len2 ? str1 : str2;
}

int main(){
    char str1[50], str2[50], *str;

    printf("string1: ");
    scanf("%[^\n]", str1);

    printf("string2: ");
    setbuf(stdin, NULL);
    scanf("%[^\n]", str2);

    str = longStr(str1, str2);
    printf("longStr: %s\n", str);

    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [16:09:55]
$ gcc a.c

# root @ localhost in ~/test [16:09:57]
$ ./a.out
string1: 12345 12345
string2: 1234 123 1
longStr: 12345 12345

# root @ localhost in ~/test [16:10:14]
$ ./a.out
string1: www zfl9 com
string2: https://www.zfl9.com
longStr: https://www.zfl9.com
</script></code></pre>

## 二级指针
所谓的二级指针就是指向指针的指针

指针可以指向一份普通类型的数据，例如 `int`、`double`、`char` 等，也可以指向一份指针类型的数据，例如 `int *`、`double *`、`char *` 等

如果一个指针指向的是另外一个指针，我们就称它为二级指针，或者指向指针的指针

假设有一个整型变量 a，p 是指向 a 的指针，pp 是指向 p 的指针：
<pre><code class="language-c"><script type="text/plain">int a = 10;
int *p = &a;
int **pp = &p;
</script></code></pre>

指针变量也是一种变量，也会占用存储空间，也可以使用 & 获取它的地址
C语言不限制指针的级数，每增加一级指针，在定义指针变量时就得增加一个星号
p1 是一级指针，指向普通类型的数据，定义时有一个星号
p2 是二级指针，指向一级指针 p1，定义时有两个星号

实际开发中常用的也就是一级指针和二级指针，几乎不用高级指针

想要获取指针指向的数据时，一级指针加一个星号，二级指针加两个星号，三级指针加三个星号，以此类推
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a = 10;
    int *p1 = &a;
    int **p2 = &p1;
    int ***p3 = &p2;

    printf("var of a: %d, %d, %d, %d\n", a, *p1, **p2, ***p3);
    printf("addr of p2: %p, %p\n", &p2, p3);
    printf("addr of p1: %p, %p, %p\n", &p1, p2, *p3);
    printf("addr of a: %p, %p, %p, %p\n", &a, p1, *p2, **p3);

    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [16:25:07]
$ gcc a.c

# root @ localhost in ~/test [16:25:09]
$ ./a.out
var of a: 10, 10, 10, 10
addr of p2: 0x7fff58ac1b80, 0x7fff58ac1b80
addr of p1: 0x7fff58ac1b88, 0x7fff58ac1b88, 0x7fff58ac1b88
addr of a: 0x7fff58ac1b94, 0x7fff58ac1b94, 0x7fff58ac1b94, 0x7fff58ac1b94
</script></code></pre>

## NULL指针 void指针
一个指针变量可以指向计算机中的任何一块内存，不管该内存有没有被分配，也不管该内存有没有使用权限，只要把地址给它，它就可以指向，C语言没有一种机制来保证指向的内存的正确性，程序员必须自己提高警惕

指针使用之前一定要记得初始化，使用未初始化的指针是非常危险的，因为未初始化的指针乱指一气，运行程序时，在Linux下表现为`段错误`，在Windows下直接奔溃

强烈建议对未初始化的指针变量指向`NULL`，如`int *p = NULL;`

`NULL`是零值、等于零的意思，在C语言中表示空指针
从表面上理解，空指针是不指向任何数据的指针，是无效指针，程序使用它不会产生效果

很多库函数都对传入的指针做了判断，如果是空指针就不做任何操作，或者给出提示信息

其实，`NULL`是在`stdio.h`中定义的一个宏，它的具体内容为：
`#define NULL ((void *)0)`
`(void *)0`表示把数值 0 强制转换为`void *`类型，最外层的()把宏定义的内容括起来，防止发生歧义
从整体上来看，NULL 指向了地址为 0 的内存，而不是前面说的不指向任何数据

注意，C语言没有规定 NULL 的指向，只是大部分标准库约定成俗地将 NULL 指向 0，所以不要将 NULL 和 0 等同起来

**void指针**
对于空指针 NULL 的宏定义内容，上面只是对`((void *)0)`作了粗略的介绍，这里重点说一下`void *`的含义
void 用在函数定义中可以表示函数没有返回值或者没有形式参数，用在这里表示指针指向的数据的类型是未知的

也就是说，`void *`表示一个有效指针，它确实指向实实在在的数据，只是`数据的类型尚未确定`，在后续使用过程中一般要进行强制类型转换

## 数组和指针不等价
[数组和指针绝不等价](https://www.zfl9.com/c-array.html#数组和指针绝不等价)
[数组什么时候会转换为指针](https://www.zfl9.com/c-array.html#数组什么时候会转换为指针)

## 指针数组
指针数组：是一个数组，数组的元素是一个指针

`type *array[len];`
由于`[]`的优先级比`*`高，所以应理解为`type *(array[len]);`
括号里面的`array[len]`说明这是一个数组，外面的`type *`说明数组元素的类型为指针

比如`int *a[5];`，表示：a 是一个拥有 5 个元素的数组，每个元素的数据类型为 int *，即整型指针

除了元素的数据类型不同，指针数组和普通数组没有区别
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a = 10, b = 20, c = 30;
    int *array[3] = {&a, &b, &c};
    int **p = array; // 指向指针数组的指针
    printf("%d, %d, %d\n", *array[0], *array[1], *array[2]);
    printf("%d, %d, %d\n", **(p + 0), **(p + 1), **(p + 2));
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [17:45:24]
$ gcc a.c

# root @ localhost in ~/test [17:45:27]
$ ./a.out
10, 20, 30
10, 20, 30
</script></code></pre>

`int **p = array;`应该理解为`int *(*p)`，括号里的`*p`表示这是一个指针，括号外的`int *`表示指针指向的数据类型为一个整型指针

`指针数组`还可以和`字符串数组`结合使用，请看下面的例子：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char *strings[3] = {
        "www.zfl9.com",
        "www.zflyun.com",
        "www.otokaze.top"
    };
    int i;
    for(i=0; i<3; i++){
        printf("%s\n", strings[i]);
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [17:51:44]
$ gcc a.c

# root @ localhost in ~/test [17:51:45]
$ ./a.out
www.zfl9.com
www.zflyun.com
www.otokaze.top
</script></code></pre>

需要注意的是，字符数组 strings 中存放的是字符串的首地址，不是字符串本身，字符串本身位于其他的内存区域，和字符数组是分开的

**一道题目玩转指针数组和二级指针**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char *lines[5] = {
        "COSC1283/1284",
        "Programming",
        "Techniques",
        "is",
        "great fun"
    };

    char *str1 = lines[1];
    char *str2 = *(lines + 3);
    char c1 = *(*(lines + 4) + 6);
    char c2 = (*lines + 5)[5];
    char c3 = *lines[0] + 2;

    printf("str1 = %s\n", str1);
    printf("str2 = %s\n", str2);
    printf("  c1 = %c\n", c1);
    printf("  c2 = %c\n", c2);
    printf("  c3 = %c\n", c3);

    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [18:01:51]
$ gcc a.c

# root @ localhost in ~/test [18:01:53]
$ ./a.out
str1 = Programming
str2 = is
  c1 = f
  c2 = 2
  c3 = E
</script></code></pre>

说明：
lines 是一个拥有 5 个元素的数组，数组元素的数据类型为 `char *`，即各个字符串的首地址
str1：`lines[1]`，即数组 lines 的第1个元素，Programming 的首地址，所以输出 Programming
str2：`*(lines + 3)`，等价于`lines[3]`，即数组 lines 的第3个元素，is 的首地址，输出 is
c1：`*(*(lines + 4) + 6)`，先看`*(lines + 4)`，即数组 lines 的第4个元素，great fun 的首地址，然后加6，表示指针往后偏移6个长度，然后再取值，即 f
c2: `(*lines + 5)[5]`，`*lines`等价于`*(lines+0)`等价于`lines[0]`，即第0个元素，COSC1283/1284 的首地址，然后指针往后偏移5个长度，然后看到`[5]`，等价于`*(lines[0] + 5 + 5)`，即字符2
c3：`*lines[0] + 2`，等价于`*(lines[0]) + 2`，括号内表示数组第0个元素，即 COSC1283/1284 首地址，也就是字符 C 的地址，然后再取它的值，即字符 C，然后与数值2相加，即ASCII码67 + 2 = 69，即字符 E

## 二维数组指针
二维数组指针：它是一个指针，指向一个二维数组
<pre><code class="language-c line-numbers"><script type="text/plain">int a[3][4] = {
    {1, 2, 3, 4},
    {2, 2, 3, 4},
    {3, 2, 3, 4}
};

int (*p)[4] = a;
</script></code></pre>

由于`[]`优先级比`*`高，所以这个括号是必须的
首先看`*p`，表示这是一个指针，然后看`int [4]`，表示这个指针指向一个拥有4个元素的数组

对指针 p 进行加1时，因为指向的数据类型为 int [4]，所以前进的长度为 4 * 4 = 16 字节，刚好指向数组 a 下一个元素，减1同理

1) p 指向数组 a 开头，p + 1 指向数组下一行，p - 1 指向数组上一行

2) `*(p + 1)` 表示取地址上的数据，也就是第1行的数据，注意是一行数据，也就是一个一维数组，是由4个 int 类型组成的集合，所以用 sizeof 求它的长度为 4 * 4 = 16 字节

3) `*(p + 1) + 1`，表示的是第1行第1个元素的地址，上面说了，`*(p + 1)`是一个一维数组，还记得数组什么时候会转换为指向第0个元素的地址吗，除了数组定义、sizeof、&中，其他时候都会转换为指向第0个元素的指针，所以在这里就转换为了指针，然后加1，表示是一个指向第1行第1个元素的指针

4) `*(*(p + 1) + 1)`，很明显，加了`*`操作符，表示第1行第1个元素的值
<pre><code class="language-c line-numbers"><script type="text/plain">a + i == p + i
a[i] == p[i] == *(a + i) == *(p + i)
a[i][j] == p[i][j] == *(a[i] + j) == *(p[i] + j) == *(*(a + i) + j) == *(*(p + i) + j)
</script></code></pre>

使用二维数组指针遍历二维数组
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a[3][4] = {
        1, 2, 3, 4,
        2, 2, 3, 4,
        3, 2, 3, 4,
    };
    int i, j;
    int (*p)[4] = a;
    for(i=0; i<3; i++){
        for(j=0; j<4; j++){
            printf("%d ", *(*(p + i) + j));
        }
        printf("\n");
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [18:50:48]
$ gcc a.c

# root @ localhost in ~/test [18:50:49]
$ ./a.out
1 2 3 4
2 2 3 4
3 2 3 4
</script></code></pre>

## 函数指针
函数指针，即指向函数的指针

本质上，变量名，数组名，函数名都是地址的助记符，但是我们认为变量名表示数据本身，而数组名、函数名分别表示数组第0个元素的地址、函数的入口地址

我们也可以把一个指针指向一个函数，然后通过指针来调用函数

定义的形式为：`return_type (*ptr)(param_list)`

其中参数列表可以只给出参数的类型，省略参数名，这和函数声明类似

用指针来实现对函数的调用
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int max(int a, int b){
    return a > b ? a : b;
}

int main(){
    int a, b;
    printf("enter 2 nums: ");
    scanf("%d %d", &a, &b);
    int (*p)(int, int) = max;
    printf("max: %d\n", p(a, b));
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [19:58:41]
$ gcc a.c

# root @ localhost in ~/test [19:58:43]
$ ./a.out
enter 2 nums: 10 20
max: 20
</script></code></pre>

## 彻底攻克指针
<pre><code class="language-c line-numbers"><script type="text/plain">int *p[6];      // 指针数组
int *(p[6]);    // 指针数组，等价于上面的形式
int (*p)[6];    // 二维数组指针
int (*p)(int, int); // 函数指针
int (*p)(int a, int b); //函数指针，等价于上面的形式
</script></code></pre>

C语言标准规定，对于一个符号的定义，编译器总是从它的名字开始读取，然后按照优先级顺序依次解析
对，从名字开始，不是从开头也不是从末尾，这是理解复杂指针的关键！

与指针相关的运算符优先级，由高到低
- 定义中被`()`扩起来的部分
- 后缀操作符：括号`()`表示这是一个函数，方括号`[]`表示这是一个数组
- 前缀操作符：星号`*`表示这是一个指针变量

1) `int *p[6];`
从 p 开始理解，编译器先解析 p[6]，p 是一个拥有6个元素数组，然后解析 `int *`，它说明这个数组的元素的数据类型为 `int *`，也就是指向 int 的指针，所以，从整体上讲，这是一个拥有6个元素的数组，数组元素的数据类型为整型指针，也就是`指针数组`

2) `int (*p)[6];`
从 p 开始理解，编译器先解析 *p，p 是一个指针，然后解析 int [6]，它说明这个数组的元素的数据类型为 int [6]，每个元素都是一个拥有6个元素的一维数组，所以，从整体上讲，这是一个指针，该指针指向拥有6个元素的一维数组，也就是说，这是一个`二维数组指针`

3) `int (*p)(int, int);`
从 p 开始理解，编译器先解析 *p，p 是一个指针，然后解析 int (int, int)，说明 p 指向一个函数，该函数的参数列表为两个整数，而前面的 int 说明该函数的返回值类型为 int，所以，从整体上讲，这是一个指针，该指针指向一个函数原型为 int func(int, int) 的函数，也就是说，这是一个`函数指针`

4) `char *(* c[10])(int **p);`
从 c 开始理解，`c[10]`表示这是一个拥有10个元素数组，然后解析 `(* c[10])`，前面的 `*` 说明该数组的元素的是一个指针，但是指针指向的数据类型还不确定，剩下 `char * (int **p)`，这是一个函数，函数参数为 `int **p`，返回值类型为 `char *`，所以，整体上讲，这是一个拥有10个元素的`指针数组`，每个元素(指针)都指向函数原型为 `char *func(int **p)` 的函数

5) `int (*(*(*pfunc)(int *))[5])(int *);`
从 pfunc 开始理解，`(*pfunc)`表示这是一个指针，然后看它外面一层的`* (int *)`，这是一个函数，函数参数为 `int *`，返回值为一个指针，但是还不知道该指针指向的是什么数据类型，然后看它外面一层`* [5]`，这说明该函数返回的指针指向一个指针数组，那么这个指针数组的元素又是什么类型呢？继续看，`int (int *)`，这是一个函数，函数参数为 `int *`，返回值类型为 int，所以，这个指针数组的元素(指针)指向一个函数原型为`int func(int *)`的函数，综上所述，pfunc是一个`函数指针`，该函数的返回值是一个指针，它指向一个指针数组，指针数组的元素(指针)指向原型为`int func(int *)`的函数

## 带参数的main()函数
我们之前的程序中，main()函数都是没有参数的，其实main()函数有两种标准函数原型
<pre><code class="language-c line-numbers"><script type="text/plain">int main();
int main(int argc, char *argv[]);
</script></code></pre>

如果要向程序传递命令行参数，就要使用第二种原型
第一个参数 int argc，是参数的个数，就算不传递参数，系统也会默认传递一个参数：当前执行文件的文件名，所以最小也为1
第二个参数 char *argv[]，这是一个指针数组，每一个指针都指向一个字符串的首地址，第0个元素总是当前可执行文件的文件名的首地址

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(int argc, char *argv[]){
    printf("argc: %d, argv[0]: %s\n", argc, argv[0]);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [21:01:27]
$ gcc a.c

# root @ localhost in ~/test [21:01:29]
$ ./a.out
argc: 1, argv[0]: ./a.out
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(int argc, char *argv[]){
    int i;
    for(i=0; i<argc; i++){
        printf("第 %d 个参数为: %s\n", i, argv[i]);
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [21:03:25]
$ gcc a.c

# root @ localhost in ~/test [21:03:26]
$ ./a.out
第 0 个参数为: ./a.out

# root @ localhost in ~/test [21:03:27]
$ ./a.out 1 2 3
第 0 个参数为: ./a.out
第 1 个参数为: 1
第 2 个参数为: 2
第 3 个参数为: 3
</script></code></pre>

## 指针总结
指针(Pointer)就是内存的地址，C语言允许用一个变量来存放指针，这种变量称为指针变量。
指针变量可以存放基本类型数据的地址，也可以存放数组、函数以及其他指针变量的地址。

程序在运行过程中需要的是数据和指令的地址，变量名、函数名、字符串名和数组名在本质上是一样的，它们都是地址的助记符
在编写代码的过程中，我们认为变量名表示的是数据本身，而函数名、字符串名和数组名表示的是代码块或数据块的首地址
程序被编译和链接后，这些名字都会消失，取而代之的是它们对应的地址

|定义|含义|
|:-:|:-:|
|`int *p;`|p是一个整型指针，它可以指向`int`类型的数据，也可以指向类似`int arr[n]`的数组|
|`int **p;`|p是一个二级指针，指向`int *`类型的数据|
|`int *p[n];`|p是一个指针数组，数组的每个指针都指向类型为`int *`的数据|
|`int (*p)[n];`|p是一个二维数组指针|
|`int *p();`|p是一个函数，它的返回值类型为`int *`|
|`int (*p)();`|p是一个函数指针，指向原型为`int func()`的函数|

1) 指针变量可以进行加减运算，例如p++、p+i、p-=i
指针变量的加减运算并不是简单的加上或减去一个整数，而是跟指针指向的数据类型有关

2) 给指针变量赋值时，要将一份数据的地址赋给它，不能直接赋给一个整数，例如int *p = 1000;是没有意义的，使用过程中一般会导致程序崩溃(段错误)

3) 使用指针变量之前一定要初始化，否则就不能确定指针指向哪里，如果它指向的内存没有使用权限，程序就崩溃了
对于暂时没有指向的指针，建议赋值NULL

4) 两个指针变量可以相减，如果两个指针变量指向同一个数组中的某个元素，那么相减的结果就是两个指针之间相差的元素个数

5) 数组也是有类型的，数组名的本意是表示一组类型相同的数据
在定义数组时，或者和 sizeof、& 运算符一起使用时数组名才表示整个数组，表达式中的数组名会被转换为一个指向数组的指针
