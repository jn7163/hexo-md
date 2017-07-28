---
title: c语言 - 数组
date: 2017-07-03 16:07:00
categories:
- c
tags:
- c
keywords: c语言数组
---

> 
c语言 - 数组，数组定义，二维数组，静态数组，变长数组，动态数组，字符串数组，数组和指针，数组其实也是一种类型

<!-- more -->

## 数组
定义数组，数组名为`a`，数组元素的数据类型为`int`整型，数组长度为`5`，数组是一个整体，它的内存是连续的
<pre><code class="language-c"><script type="text/plain">int a[5];
</script></code></pre>

给每个元素赋值，`index`索引从`0`开始
<pre><code class="language-c"><script type="text/plain">a[0] = 1;
a[1] = 2;
a[2] = 3;
a[3] = 4;
a[4] = 5;
</script></code></pre>

也可以在定义的同时进行赋值，数组`初始化`
<pre><code class="language-c"><script type="text/plain">int a[5] = {1, 2, 3, 4, 5};
int a[] = {1, 2, 3, 4, 5}; // 如果给全部元素赋值，可以省略数组长度
int a[5] = {1, 2, 3}; // 剩下的两个元素自动赋0值
int a[5] = {0}; // 全部元素赋0值

/* 数组长度 length 最好是整数或者常量表达式，例如 10、20*4 等，这样在所有编译器下都能运行通过 */

如果没有进行显示初始化，static存储类的数组，系统会自动赋0值，auto存储类的数组，元素的值为垃圾值
</script></code></pre>

访问数组元素，下标的取值范围为 `0 ≤ index < length`，过大或过小都会越界，导致数组溢出，发生不可预测的情况
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int array1[5]; // static存储类，自动赋0值
static int array2[5];  // static存储类，作用域为本文件，自动赋0值

int main(){
    int array3[5];  // auto存储类，垃圾值
    static int array4[5];   // static存储类，自动赋0值

    int i;
    for(i=0; i<5; i++){
        printf("array1[%d] = %d\n", i, array1[i]);
    }
    for(i=0; i<5; i++){
        printf("array2[%d] = %d\n", i, array2[i]);
    }
    for(i=0; i<5; i++){
        printf("array3[%d] = %d\n", i, array3[i]);
    }
    for(i=0; i<5; i++){
        printf("array4[%d] = %d\n", i, array4[i]);
    }

    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [11:25:07]
$ gcc a.c

# root @ localhost in ~ [11:25:09]
$ ./a.out
array1[0] = 0
array1[1] = 0
array1[2] = 0
array1[3] = 0
array1[4] = 0
array2[0] = 0
array2[1] = 0
array2[2] = 0
array2[3] = 0
array2[4] = 0
array3[0] = 4195856
array3[1] = 0
array3[2] = 4195392
array3[3] = 0
array3[4] = -803210736
array4[0] = 0
array4[1] = 0
array4[2] = 0
array4[3] = 0
array4[4] = 0
</script></code></pre>

## 二维数组
所谓二维数组，就是数组的每个元素都是数组
假设我们要存放5个同学的语数英分数，就可以使用一个5行3列的二维数组
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main() {
    /* 定义一个5行3列的二维数组 */
    int a[5][3] = {
        {98, 87, 89},
        {78, 90, 76},
        {79, 94, 84},
        {67, 56, 69},
        {55, 67, 98},
    };

    int i, j;
    for (i=0; i<5; i++){
        printf("Student %d: ", i+1);
        for (j=0; j<3; j++){
            printf("%d ", a[i][j]);
        }
        printf("\n");
    }

    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [11:40:18]
$ gcc a.c

# root @ localhost in ~ [11:40:19]
$ ./a.out
Student 1: 98 87 89
Student 2: 78 90 76
Student 3: 79 94 84
Student 4: 67 56 69
Student 5: 55 67 98
</script></code></pre>

如果是给全部元素赋值，也可以省略`{}`，结果是一样的
<pre><code class="language-c"><script type="text/plain">int a[2][2] = {{1, 2}, {3, 4}};
int a[2][2] = {1, 2, 3, 4};
int a[2][2] = {{0}, {0}};   // 全部元素赋0值
int a[][2] = {1, 2, 3, 4};  // 给全部元素赋值时，也可以省略第一维数组的长度
</script></code></pre>

## 数组元素查询
对于无序数组，我们只能通过遍历整个数组进行查询
比如，我们输入一个数字，如果该数字在数组中，则打印其下标
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int nums[10] = {1, 10, 6, 296, 177, 23, 0, 100, 34, 999};
    int num, i, index = -1;

    printf("enter num: ");
    scanf("%d", &num);

    for(i=0; i<10; i++){
        if(num == nums[i]){
            index = i;
            break;
        }
    }

    if(index == -1){
        printf("%d isn't in the array!\n", num);
    }else{
        printf("%d in the array, index: %d\n", num, index);
    }

    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [12:38:24]
$ gcc a.c

# root @ localhost in ~ [12:41:47]
$ ./a.out
enter num: 3
3 isn't in the array!

# root @ localhost in ~ [20:11:57]
$ ./a.out
enter num: 6
6 in the array, index: 2

# root @ localhost in ~ [20:12:47]
$ ./a.out
enter num: 100
100 in the array, index: 7

# root @ localhost in ~ [20:12:52]
$ ./a.out
enter num: 11
11 isn't in the array!
</script></code></pre>

对于有序数组，并不需要遍历所有元素，只需要遍历部分元素就可以判断给出的数字是否在数组中
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int nums[10] = {1, 10, 66, 296, 377, 523, 660, 700, 734, 999};
    int num, i, index = -1;

    printf("enter num: ");
    scanf("%d", &num);

    for(i=0; i<10; i++){
        if(nums[i] >= num){
            if(nums[i] == num){
                index = i;
                break;
            }
        }
    }

    if(index == -1){
        printf("%d isn't in the array!\n", num);
    }else{
        printf("%d in the array, index: %d\n", num, index);
    }

    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [20:22:22]
$ gcc a.c

# root @ localhost in ~ [20:22:52]
$ ./a.out
enter num: 1
1 in the array, index: 0

# root @ localhost in ~ [20:22:54]
$ ./a.out
enter num: 100
100 isn't in the array!
</script></code></pre>

## 字符数组
由于c语言没有字符串数据类型，只能使用`字符数组`或者`字符串指针`来表示字符串`string`
<pre><code class="language-c"><script type="text/plain">char str[13] = {'w', 'w', 'w', '.', 'z', 'f', 'l', '9', '.', 'c', 'o', 'm'};    // 末尾自动添加`\0`字符，因此长度为 12 + 1
/* 同理，如果给全部元素赋值，可以省略数组长度 */
char str[] = {'w', 'w', 'w', '.', 'z', 'f', 'l', '9', '.', 'c', 'o', 'm'};

/* C语言允许直接把字符串赋值给字符数组 */
char str[] = {"www.zfl9.com"};
char str[] = "www.zfl9.com";    // 更加简洁，常用
</script></code></pre>

注意，字符串总是以`'\0'`字符结尾，因此字符数组长度总是比字符串长度大1，`puts()`、`printf()`扫描到`'\0'`就结束输出了
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    char str1[] = "www.zfl9.com";
    char str2[] = "www\0.zfl9.com";
    puts(str1);
    puts(str2);
    printf("%s\n", str1);
    printf("%s\n", str2);
    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [20:41:53]
$ gcc a.c

# root @ localhost in ~ [20:41:54]
$ ./a.out
www.zfl9.com
www
www.zfl9.com
www
</script></code></pre>

## string.h
<pre><code class="language-c"><script type="text/plain">## strlen(str) 返回字符串长度(不包括'\0'字符)
char str[] = "www.zfl9.com";
int len = strlen(str);  // len = 12

## strcat(str1, str2)   将str2拼接在str1后面，并删除str1的'\0'字符，要保证str1有足够的空间，否则会越界
char str1[30] = "www.zfl9.com";
char str2[] = "www\0.zfl9.com";
strcat(str1, str2); // str1: "www.zfl9.comwww"
char* str = strcat(str1, str2); // strcat的返回值为str1的首地址，字符串指针str指向str1第一个字符的地址
printf("str: %s\n", str);   // "www.zfl9.comwww"

## strcpy(str1, str2)   将str2复制到str1(覆盖)，结尾'\0'一起复制
char str1[13] = "$";
char str2[] = "www.zfl9.com";
strcpy(str1, str2); // str1: "www.zfl9.com\0", str2: "www.zfl9.com\0"

## strcmp(str1, str2)   比较每个字符的ASCII码，从第0个字符开始比较，若相等，则比较下一个，直到遇见不同字符，或者到末尾
如果str1和str2字符串相同，则返回0，如果str1大于str2，则返回大于0的值，反之则返回小于0的值
char str1[] = "abce";
char str2[] = "abcd";
strcmp(str1, str2); // 'e'比'd'的ASCII码大1，所以返回1
strcmp(str2, str1); // 'd'比'e'的ASCII码小1，所以返回-1
strcmp(str1, str1); // 返回0
</script></code></pre>

## 字符串输入输出
<pre><code class="language-c"><script type="text/plain">// 输出 printf() puts()
char str[] = "www.zfl9.com";    // 字符数组
char* string = "www.zfl9.com";  // 字符串指针(字符串常量)
printf("char[]: %s\tchar*: %s\n", str, string);
puts(str);
puts(string);


// 输入 scanf() gets()
char str[100];
scanf("%s", str);   // 注意，数组名代表第0个元素的地址，所以不需要 & 取地址符
gets(str);
</script></code></pre>

## 静态数组
> 
在C语言中，数组一旦定义之后，其占用的内存空间就是固定的，不能增加或删除元素，只能修改元素的值，这样的数组称之为`静态数组`
当使用下标访问数组元素时，如果下标大于数组长度或者小于0，就会发生`越界`(out of bounds)，小于0发生`下限越界`(off normal lower)，大于数组长度发生`上限越界`(off normal upper)
C语言为了提高效率，并不会对数组越界进行检查，即使越界了，也能通过编译，只有在运行期间才可能会发生问题

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a[3] = {1, 2, 3};
    int i;
    for(i=-2; i<5; i++){
        printf("a[%d] = %d\n", i, a[i]);
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [11:09:03]
$ gcc a.c

# root @ localhost in ~ [11:09:05]
$ ./a.out
a[-2] = 4195392
a[-1] = 0
a[0] = 1
a[1] = 2
a[2] = 3
a[3] = 3
a[4] = 0
</script></code></pre>

我们也不知道数组前后的是什么数据，上面的程序貌似没有出现什么大问题，还能正常运行，但是，如果不小心访问到了其他程序的内存或者系统保护的内存等等，程序就会直接蹦了
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int a[3] = {1, 2, 3};
    int num = a[100000];
    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [11:12:40]
$ gcc a.c

# root @ localhost in ~ [11:12:42]
$ ./a.out
[1]    54630 segmentation fault  ./a.out
</script></code></pre>

发生`段错误`，因为访问了无法访问的内存，程序就被系统直接kill了

> 
`数组溢出`：当赋予数组的元素个数超过数组长度时，就会发生`溢出`(overflow)

<pre><code class="language-c"><script type="text/plain">int a[3] = {1, 2, 3, 4, 5}; // 只会保存前3个元素，后2个被丢弃
// vc/vs发生数组溢出时，编译会报错，不能通过编译，但是gcc只是警告
// 对于字符数组，一定要留有足够的空间，以存放末尾的'\0'字符
</script></code></pre>

## 变长数组
目前主要有三个c语言标准：C89/ANSI C、C99、C11
对于C89，基本上所有的编译器都支持，如微软的VC/VS，GNU的gcc
对于C99，老版本的VC/VS并不支持，而gcc很早就支持了

在C89，必须使用数值常量指明数组长度，不能包含变量
而对于C99，数组长度可以为变量，也就是可以在运行时确定长度，这种就叫做`变长数组`

注意，`变长数组`仍属于`静态数组`，一旦确定长度，就不可再次更改

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    unsigned int len;
    printf("array len: ");
    scanf("%d", &len);
    char str[len];
    printf("enter string: ");
    scanf("%s", str);
    printf("%s\n", str);
    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [20:58:56]
$ gcc a.c

# root @ localhost in ~ [20:58:58]
$ ./a.out
array len: 20
enter string: www.zfl9.com
www.zfl9.com
</script></code></pre>

**注意：变长数组不能在定义的同时进行初始化！**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int max(int a, int b){
    return a > b ? a : b;
}

int main(void){
    int a = 2, b = 3;
    int len = a * b;
    int arr1[6] = {0};      // 正确
    int arr2[2 * 3] = {0};  // 正确
    int arr3[a * b] = {0};  // 错误，变长数组不能在定义的同时进行赋值！
    int arr4[len];          // 正确
    int arr5[max(a, b)];    // 正确
    return 0;
}
</script></code></pre>

## 动态数组
> 
`静态数组`由于要在创建数组的时候就确定长度，并且长度`length`还不能为变量，只能是常数，有时候我们并不知道我们到底需要多长的数组，并且很多时候数组要能动态扩容的，这时候就需要`动态数组`了
所谓`动态数组`，其实就是在`堆`(heap)中创建的数组，通过执行代码为其分配内存，我们自己负责内存的释放

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

int main(){
    int len, i;
    int* array = NULL;

    printf("array len: ");
    scanf("%d", &len);

    array = (int*)malloc(sizeof(int) * len);
    if(array == NULL){
        printf("分配内存失败！");
        exit(1);
    }

    for(i=0; i<len; i++){
        array[i] = i+100;
    }

    for(i=0; i<len; i++){
        printf("array[%d] = %d\n", i, array[i]);
    }

    free(array);
    array = NULL;

    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [11:34:56]
$ gcc a.c

# root @ localhost in ~ [11:35:52]
$ ./a.out
array len: 10
array[0] = 100
array[1] = 101
array[2] = 102
array[3] = 103
array[4] = 104
array[5] = 105
array[6] = 106
array[7] = 107
array[8] = 108
array[9] = 109
</script></code></pre>

也可使用指针的方式赋值和取值
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

int main(){
    int len, i;
    int* array = NULL;
    int* arrayCp = NULL;

    printf("array len: ");
    scanf("%d", &len);

    arrayCp = array = (int*)malloc(sizeof(int) * len);
    if(array == NULL){
        printf("分配内存失败！");
        exit(1);
    }

    for(i=0; i<len; i++){
        *arrayCp++ = i+100;
    }

    arrayCp = array;
    for(i=0; i<len; i++){
        printf("array[%d] = %d\n", i, *arrayCp++);
    }

    free(array);
    arrayCp = array = NULL;

    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [11:44:56]
$ gcc a.c

# root @ localhost in ~ [11:46:07]
$ ./a.out
array len: 5
array[0] = 100
array[1] = 101
array[2] = 102
array[3] = 103
array[4] = 104
</script></code></pre>

## 数组小结
`数组`(Array)是一系列相同类型的数据的集合，可以是一维的、二维的、多维的
最常用的是`一维数组`和`二维数组`，多维数组较少用到

数组的定义格式为：
`type array[length]`
`type` 为数据类型，`array` 为数组名，`length` 为数组长度
数组长度 length 最好是整数或者常量表达式，例如 10、20*4 等，这样在所有编译器下都能运行通过
数组在内存中占用一段连续的空间，数组名表示的是这段内存空间的`首地址`

访问数组中某个元素的格式为：
`array[index]`
`0 <= index <= length-1`

数组赋值
<pre><code class="language-c"><script type="text/plain">// 对单个元素赋值
int a[3];
a[0] = 3;
a[1] = 100;
a[2] = 34;

// 整体赋值 (不指明数组长度)
float b[] = { 23.3, 100.00, 10, 0.34 };

// 整体赋值 (指明数组长度)
int m[10] = { 100, 30, 234 };

// 字符数组赋值
char str1[] = "https://www.zfl9.com";

// 将数组所有元素都初始化为0
int arr[10] = {0};
char str2[20] = {0};
</script></code></pre>

排序和查找
简单应用，给一个数组，计算其最大和最小值
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    int i, max, min, array[10] = {0};
    // 初始化数组
    printf("enter 10 nums: ");
    for(i=0; i<10; i++){
        scanf("%d", &array[i]);
    }
    // 假设最大值和最小值都为 array[0]
    max = min = array[0];
    // 查找最大值和最小值
    for(i=1; i<10; i++){
        if(array[i] > max){
            max = array[i];
        }
        if(array[i] < min){
            min = array[i];
        }
    }

    printf("max: %d, min: %d\n", max, min);

    return 0;
}
</script></code></pre>

<pre><code class="language-c"><script type="text/plain"># root @ localhost in ~ [12:01:36]
$ gcc a.c

# root @ localhost in ~ [12:01:38]
$ ./a.out
enter 10 nums: 1 2 3 4 5 6 7 8 9 10
max: 10, min: 1

# root @ localhost in ~ [12:01:52]
$ ./a.out
enter 10 nums: 2 123 45 100 575 240 799 710 10 90
max: 799, min: 2
</script></code></pre>

## 数组什么时候会转换为指针
数组名的本意是表示一组数据的集合，和普通变量一样，都用来指代一块内存
但在使用过程中，数组名有时候会转换成指向第0个元素的指针

当数组名作为`数组定义的标识符(也就是定义或声明数组时)`、`sizeof` 或 `&` 的操作数时，它才表示`整个数组本身`
在其他的表达式中，数组名会被转换为指向第 0 个元素的指针(地址)

**再谈数组下标[]**
数组的下标与指针的偏移量相同，都是1个元素占用的字节数，如：
`int a[] = {1, 2, 3, 4, 5}, *p, i = 2;`
可以通过下面任何一种方式访问`a[i]`：
`p = a; p[i];`、`p = a; *(p + i);`、`p = a + i; *p;`

对数组的引用`a[i]`在编译时总是被编译器改写成`*(a+i)`的形式，C语言标准也要求编译器必须具备这种行为
所以，对于数组a，如果要访问第3个元素的值，可以用`a[3]`，也可以用`3[a]`，都会改写为`*(a+3)`、`*(3+a)`，这并没有任何区别

**数组作为函数参数**
C语言标准规定，作为`类型的数组`的形参应该调整为`类型的指针`
在函数形参定义这个特殊情况下，编译器必须把数组形式改写成指向数组第 0 个元素的指针形式
编译器只向函数传递`数组的地址`，而不是整个数组的拷贝

这种隐式的转换，意味着下面的三种方式完全是等价的
<pre><code class="language-c"><script type="text/plain">void func(int *parr){ ...... }
void func(int arr[]){ ...... }
void func(int arr[5]){ ...... }
</script></code></pre>

在函数内部，arr 会被转换成一个指针变量，编译器为 arr 分配 4 个字节(32位平台)的内存
用 sizeof(arr) 求得的是指针变量的长度，而不是数组长度
要想在函数内部获得数组长度必须额外增加一个参数，在调用函数之前求得数组长度

## 数组和指针绝不等价
数组和指针不等价的一个典型案例就是求数组的长度，这个时候只能使用数组名，不能使用数组指针
因为这时候数组名代表的是一组数据的集合，而求数组指针的长度其实只是求一个指针变量的长度

数组是一系列数据的集合，没有开始和结束标志
p 仅仅是一个指向 int 类型的指针，编译器不知道它指向的是一个整数还是一堆整数
对 p 使用 sizeof 求得的是指针变量本身的长度
也就是说，编译器并没有把 p 和数组关联起来，p 仅仅是一个指针变量，不管它指向哪里，sizeof 求得的永远是它本身所占用的字节数

站在编译器的角度讲，变量名、数组名都是一种符号，它们最终都要和数据绑定起来。变量名用来指代一份数据，数组名用来指代一组数据(数据集合)，它们都是有`类型`的，以便推断出所指代的数据的长度

没错，数组也是由类型的，就像基本数据类型一样，数组是一种由基本类型派生而来的稍复杂一些的类型，sizeof是根据数据的类型来计算长度的

对于数组`int a[6] = {1, 2, 3, 4, 5, 6};`，其类型为`int [6]`，表示一个拥有6个int数据的集合
对于指针`int *p;`，其类型为`int *`，32位系统下是4字节，64位系统下是8字节

归根结底，a和p这两种符号的类型不同，指代的数据不同，自然用sizeof求得的长度不同

对于二维数组，也是同样的道理，`int a[2][3] = {1, 2, 3, 4, 5, 6}`，其类型为`int a[2][3]`

编译器在编译过程中会创建一张专门的表格用来保存名字以及名字对应的数据类型、地址、作用域等信息
sizeof 是一个操作符，不是函数，使用 sizeof 时可以从这张表格中查询到符号的长度

与普通变量名相比，数组名既有一般性也有特殊性：
一般性表现在数组名也用来指代特定的内存块，也有类型和长度
特殊性表现在数组名有时候会转换为一个指针，而不是它所指代的数据本身的值
