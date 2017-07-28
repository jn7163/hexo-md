---
title: c语言 - 结构体、位域、共用体、枚举、内存对齐
date: 2017-07-11 13:57:00
categories:
- c
tags:
- c
keywords: C语言 结构体 位域 共用体 枚举 内存对齐
---

> 
c语言 - 结构体、位域、共用体、枚举、内存对齐

<!-- more -->

## 结构体
数组可以存放一组相同数据类型的集合，但是如果一组数据类型不同的集合如何存放呢？那就是`结构体(struct)`
<pre><code class="language-c line-numbers"><script type="text/plain">struct 结构体名{
    结构体成员
};
</script></code></pre>

结构体是一种集合，它里面包含了多个变量或数组，它们的类型可以相同也可以不同，这样的变量或数组都称为结构体的`成员(member)`
<pre><code class="language-c line-numbers"><script type="text/plain">struct Student{
    char *name;
    int age;
    float score;
};
</script></code></pre>

Student 为结构体名，它包含了 3 个成员：name、age、score，结构体成员的定义和变量的定义没什么不同，只是不能初始化，**注意花括号后面的分号，不能省略，因为这是一个完整的语句！**

结构体也是一种数据类型，是我们自定义的一种数据类型，它可以包含多个其他类型的数据

像 int、char、float 等C语言本身提供的数据类型，我们称之为`基本数据类型`，不能再拆解；
而结构体可以包含多个基本数据类型的数据，也可以包含其他的结构体，我们称之为`复杂数据类型`或`构造数据类型`

注意，对于基本类型，我们用 int、short、char 等C语言本身提供的标识符表示，而对于我们自定义的结构体数据类型，以上面的为例子，我们用`struct Student`来描述这种构造类型

它们都是描述数据类型的标识符，没有区别，只是有的是C语言自带的，有的是我们自定义的

还有一点要注意：
上面的代码只是定义了一个新的数据类型，它本身不占用内存，只有用它来定义变量时，才会给变量分配内存！
就如同 int 一样，int 本身是不占用空间的，只有执行 `int i;` 才会分配 4 个字节的内存给变量 a

下面我们来用我们自定义的类型来定义变量：
`struct Student stu1, stu2;`

也可以在定义结构体的同时定义变量：
<pre><code class="language-c line-numbers"><script type="text/plain">struct Student{
    char *name;
    int age;
    float score;
} stu1, stu2;
</script></code></pre>

也可以省略结构体名，相当于一个匿名的结构体，如果没有结构体名，在之后的代码中就不能再用它来定义变量了，除了在定义结构体的同时定义的变量
<pre><code class="language-c line-numbers"><script type="text/plain">struct{
    char *name;
    int age;
    float score;
} stu1, stu2;
</script></code></pre>

理论上讲，结构体中的成员在内存中是连续的，但是实际上在编译器的具体实现中，各个成员之间可能会存在缝隙，这是因为C语言内存对齐的原因

**什么是内存对齐，为什么要对齐？**
- 现代计算机中内存空间都是按照 byte 划分的，从理论上讲似乎对任何类型的变量的访问可以从任何地址开始，但实际情况是在访问特定变量的时候经常在特定的内存地址访问，这就需要各类型数据按照一定的规则在空间上排列，而不是顺序的一个接一个的排放，这就是对齐。
- 对齐的作用和原因：各个硬件平台对存储空间的处理上有很大的不同。一些平台对某些特定类型的数据只能从某些特定地址开始存取。其他平台可能没有这种情况，但是最常见的是如果不按照适合其平台的要求对数据存放进行对齐，会在存取效率上带来损失。
比如有些平台每次读都是从偶地址开始，如果一个 int 型（假设为32位）如果存放在偶地址开始的地方，那么一个读周期就可以读出，而如果存放在奇地址开始的地方，就可能会需要 2 个读周期，并对两次读出的结果的高低字节进行拼凑才能得到该 int 数据。显然在读取效率上下降很多，这也是空间和时间的博弈。

“内存对齐”应该是编译器的“管辖范围”。
编译器为程序中的每个“数据单元”安排在适当的位置上。
但是C语言的一个特点就是太灵活，太强大，它允许你干预“内存对齐”

**对齐规则**
每个特定平台上的编译器都有自己默认的“对齐系数”，我们可以通过预处理指令`#pragma pack(n), n=1, 2, 4, 8, 16...`来改变这一系数，这个 n 就是对齐系数
- **`数据成员`**对齐规则：`结构(struct)`或`联合(union)`的数据成员，第一个数据成员放在 offset 为 0 的地方，以后的每个数据成员的对齐按照`#pragma pack(n)`指定的 `n` 值和该数据成员本身的长度 `len = sizeof(type)` 中，较小的那个进行，如果没有显示指定`n`值，则以`len`为准，进行对齐
- **`结构/联合`**整体对齐规则：在数据成员对齐完成之后，结构/联合本身也要对齐，对齐按照`#pragma pack(n)`指定的`n`值和该结构/联合最大数据成员长度`max_len_of_members`中，较小的那个进行，如果没有显示指定`n`值，则以`max_len_of_members`为准，进行对齐
- 结合1、2可推断：当`n`值均超过(或等于)所有数据成员的长度时，这个`n`值的大小将不产生任何效果

**内存对齐的例子**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

struct{
    char a;
    double b;
    int c;
} test;

int main(){
    printf("%d\n", sizeof(test));
    return 0;
}
</script></code></pre>

你肯定会想，占用的大小为 sizeof(char) + sizeof(double) + sizeof(int) = 1 + 8 + 4 = 13 字节，然而：
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [15:46:05]
$ gcc a.c

# root @ localhost in ~/test [15:46:07]
$ ./a.out
24
</script></code></pre>

我们按照上面的对齐规则来分析一下：
首先是成员`a`，类型为`char`，长度为`1`，放在偏移量为 0 的地方，然后偏移量变为了 1
然后是成员`b`，类型为`double`，长度为`8`，要放在偏移量为 8 的整数倍的地方，所以就放在 8 上，然后偏移量变为了 8 + 8 = 16
最后是成员`c`，类型为`int`，长度为`4`，要是 4 的整数倍，16 刚好是它的倍数，偏移量变为了 16 + 4 = 20
最后，整个结构体也要对齐，成员中最大的长度为 8，而要它的整数倍，那就是 24 了，所以整个结构体的长度就是 24 个字节

再来看定义了`#pragma pack(n)`的情况：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

#pragma pack(4)

struct{
    char a;
    double b;
    int c;
} test;

int main(){
    printf("%d\n", sizeof(test));
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [15:57:12]
$ gcc a.c

# root @ localhost in ~/test [15:57:14]
$ ./a.out
16
</script></code></pre>

根据对齐规则：
对于成员`a`，对齐数为 1，因为 1 小于 4，放在偏移量为 0 的位置上，然后长度变为 1
对于成员`b`，对齐数为 4，因为 4 小于 8，放在偏移量为 4 的位置上，长度变为 4 + 8 = 12
对于成员`c`，对齐数为 4，两个数相等，放在偏移量为 12 的位置上，长度变为 12 + 4 = 16
最后是结构体，最大的成员长度 8，大于 4，所以取 4，刚好整除，所以就是 16 字节

**成员的获取与赋值**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

struct Student{
    char *name;
    int age;
    float score;
};

int main(){
    struct Student stu;
    stu.name = "Otokaze";
    stu.age = 18;
    stu.score = 119.5f;
    printf("%s 今年 %d 岁，Ta 考了 %.1f 分！\n", stu.name, stu.age, stu.score);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [14:51:37]
$ gcc a.c

# root @ localhost in ~/test [14:51:40]
$ ./a.out
Otokaze 今年 18 岁，Ta 考了 119.5 分！
</script></code></pre>

除了可以对成员一一赋值，也可以在定义的时候进行整体赋值，这和数组非常相似，注意，只能在定义的时候进行整体赋值，其他时候只能一一赋值！数组也是一样的道理

<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

struct Student{
    char *name;
    int age;
    float score;
} stu = {
    "Otokaze",
    18,
    119.5f,
};

int main(){
    printf("%s 今年 %d 岁，Ta 考了 %.1f 分！\n", stu.name, stu.age, stu.score);
    return 0;
}
</script></code></pre>

再来看看下面这个例子；
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

struct Student{
    char *name;
    int age;
    float score;
};

int main(){
    struct Student stu;
    printf("Name: ");
    scanf("%s", stu.name);
    printf("Age: ");
    scanf("%d", &stu.age);
    printf("Score: ");
    scanf("%f", &stu.score);
    printf("%s 今年 %d 岁，Ta 考了 %.1f 分！\n", stu.name, stu.age, stu.score);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [15:07:31]
$ gcc a.c

# root @ localhost in ~/test [15:07:33]
$ ./a.out
Name: Otokaze
Age: 18
Score: 118.5
Otokaze 今年 18 岁，Ta 考了 118.5 分！
</script></code></pre>

好像没什么问题，对吧，我们再来改一下，加了个循环：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

struct Student{
    char *name;
    int age;
    float score;
};

int main(){
    for(int i=0; i<5; i++){
        struct Student stu;
        printf("Name: ");
        scanf("%s", stu.name);
        printf("Age: ");
        scanf("%d", &stu.age);
        printf("Score: ");
        scanf("%f", &stu.score);
        printf("%s 今年 %d 岁，Ta 考了 %.1f 分！\n", stu.name, stu.age, stu.score);
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [15:09:29]
$ gcc b.c

# root @ localhost in ~/test [15:09:30]
$ ./a.out
Name: Otokaze
[1]    31162 segmentation fault  ./a.out
</script></code></pre>

段错误？为什么？
这是一个很多初学者都会遇到的一个问题，包括我自己，我们来一步步分析：

先来分析第一个例子：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

struct Student{
    char *name;     // 这是一个指针变量，由于我的编译环境是 64bit 系统，所以一个指针变量的长度为 64bit / 8bit = 8个字节
    int age;        // 整型变量，4字节
    float score;    // 浮点型变量，4字节
};
// 注意在上面所说的成员的占用内存大小到目前为止都还没分配，我只是先说明一下，再来看下面的代码：
int main(){
    struct Student stu; /* 分配内存，stu变量占用的内存大小我们先不考究，但是它每个成员的内存大小我们已经可以确认
stu.name分配了8个字节，stu.age分配了4个字节，stu.score分配了4个字节 */
    printf("Name: ");
    scanf("%s", stu.name); /* 将stu.name变量的值，即该指针保存的地址传递给scanf()，这时候问题来了，因为这个指针没有初始化，所以它现在指向的内存是随机的，对于它指向的内存，该指针不一定有读和写的权限，所以极有可能指向了一块没有权限的内存，就会导致段错误！ */
    printf("Age: ");
    scanf("%d", &stu.age); // 没问题，stu.age分配了有效的内存，用以存储 int 类型的数据
    printf("Score: ");
    scanf("%f", &stu.score); // 也没问题，stu.score分配了有效的内存，用以存储 float 类型的数据
    printf("%s 今年 %d 岁，Ta 考了 %.1f 分！\n", stu.name, stu.age, stu.score);
    return 0;
}
</script></code></pre>

很奇怪，为什么我们第一个例子，在运行的时候并没有发生任何异常，因为恰好它指向的内存是该指针有权限(读和写)的一块内存
如果运气不好，指向了一块只读的内存比如常量区，或者指向了代码区，这是会致命的！

为了看到效果，我们在外边加了一个 for 循环，所以问题一下子就出来了

那如何解决这个问题？那就要在使用这个指针之前，为其分配内存空间，让它指向一块有意义的内存
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

#define STRLEN 128

struct Student{
    char *name;
    int age;
    float score;
};

int main(){
    for(int i=0; i<5; i++){
        struct Student stu;
        printf("Name: ");
        stu.name = (char *)malloc(sizeof(char) * STRLEN);
        scanf("%s", stu.name);
        printf("Age: ");
        scanf("%d", &stu.age);
        printf("Score: ");
        scanf("%f", &stu.score);
        printf("%s 今年 %d 岁，Ta 考了 %.1f 分！\n", stu.name, stu.age, stu.score);
        free(stu.name);
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [15:40:34]
$ gcc b.c

# root @ localhost in ~/test [15:41:03]
$ ./a.out
Name: Otokaze
Age: 18
Score: 129
Otokaze 今年 18 岁，Ta 考了 129.0 分！
Name: 张三
Age: 10
Score: 39
张三 今年 10 岁，Ta 考了 39.0 分！
Name: 李四
Age: 17
Score: 100
李四 今年 17 岁，Ta 考了 100.0 分！
Name: 二狗子
Age: 15
Score: 0
二狗子 今年 15 岁，Ta 考了 0.0 分！
Name: 马云
Age: 45
Score: 150
马云 今年 45 岁，Ta 考了 150.0 分！
</script></code></pre>

## 结构体数组
所谓结构体数组，就是数组的每个元素都是一个结构体
定义结构体数组：
<pre><code class="language-c line-numbers"><script type="text/plain">struct Student{
    char *name;
    int age;
    float score;
} stus[3] = {
    {"Otokaze", 18, 119},
    {"ZhangSan", 17, 120},
    {"Lisi", 19, 110.5}
};
</script></code></pre>

使用也很简单：
`int myAge = stus[1].age`

## 结构体指针
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

struct Student{
    char *name;
    int age;
    float score;
} stu;

int main(){
    struct Student *p = &stu;
    (*p).name = (char *)malloc(sizeof(char) * 30);
    printf("name: ");
    scanf("%s", (*p).name);
    printf("age: ");
    scanf("%d", &((*p).age));
    printf("score: ");
    scanf("%f", &((*p).score));
    printf("Name: %s, Age: %d, Score: %.1f\n", (*p).name, (*p).age, (*p).score);
    free((*p).name);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [16:23:45]
$ gcc a.c

# root @ localhost in ~/test [16:23:47]
$ ./a.out
name: Otokaze
age: 18
</script></code></pre>

但是每次都用`*p`取数据，太麻烦了，我们可以直接用`->`来通过指针直接获取结构体的成员，这也是`->`在c语言中唯一的用途
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

struct Student{
    char *name;
    int age;
    float score;
} stu;

int main(){
    struct Student *p = &stu;
    p->name = (char *)malloc(sizeof(char) * 30);
    printf("name: ");
    scanf("%s", p->name);
    printf("age: ");
    scanf("%d", &p->age);
    printf("score: ");
    scanf("%f", &p->score);
    printf("Name: %s, Age: %d, Score: %.1f\n", p->name, p->age, p->score);
    free(p->name);
    return 0;
}
</script></code></pre>

**结构体数组指针的应用**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

struct Student{
    char *name;
    int age;
    int num;
    char group;
    float score;
} stus[] = {
    {"Zhou ping", 5, 18, 'C', 145.0},
    {"Zhang ping", 4, 19, 'A', 130.5},
    {"Liu fang", 1, 18, 'A', 148.5},
    {"Cheng ling", 2, 17, 'F', 139.0},
    {"Wang ming", 3, 17, 'B', 144.5}
};

int main(){
    struct Student *ptr = stus;
    int len = sizeof(stus) / sizeof(struct Student);
    printf("%10s%10s%10s%10s%10s\n", "Name", "Age", "Num", "Group", "Score");
    for(; ptr<stus+len; ptr++){
        printf("%10s%10d%10d%10c%10.1f\n", ptr->name, ptr->age, ptr->num, ptr->group, ptr->score);
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [16:39:55]
$ gcc a.c

# root @ localhost in ~/test [16:39:56]
$ ./a.out
      Name       Age       Num     Group     Score
 Zhou ping         5        18         C     145.0
Zhang ping         4        19         A     130.5
  Liu fang         1        18         A     148.5
Cheng ling         2        17         F     139.0
 Wang ming         3        17         B     144.5
</script></code></pre>

**结构体指针作为函数参数传递**
结构体变量名代表的是`整个集合本身`，作为函数参数时传递的整个集合，也就是所有成员，而不是像数组一样被编译器转换成一个指针
如果结构体成员较多，尤其是成员为数组时，传送的时间和空间开销会很大，影响程序的运行效率
所以最好的办法就是使用`结构体指针`，这时由实参传向形参的只是一个地址，非常快速

如：计算全班总成绩、平均成绩、成绩低于140的人数
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

struct Student{
    char *name;
    int age;
    int num;
    char group;
    float score;
} stus[] = {
    {"Zhou ping", 5, 18, 'C', 145.0},
    {"Zhang ping", 4, 19, 'A', 130.5},
    {"Liu fang", 1, 18, 'A', 148.5},
    {"Cheng ling", 2, 17, 'F', 139.0},
    {"Wang ming", 3, 17, 'B', 144.5}
};

void average(struct Student *ptr, int len){
    float sum = 0, aver;
    int less_140 = 0;
    struct Student * const ptr_cp = ptr;
    for(; ptr<ptr_cp+len; ptr++){
        sum += ptr->score;
        if(ptr->score < 140){
            less_140++;
        }
    }
    aver = sum / len;
    printf("总分: %.1f, 平均分: %.1f, 少于140分的人数: %d\n", sum, aver, less_140);
}

int main(){
    struct Student *ptr = stus;
    int len = sizeof(stus) / sizeof(struct Student);
    average(ptr, len);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [16:57:22]
$ gcc a.c

# root @ localhost in ~/test [16:57:24]
$ ./a.out
总分: 707.5, 平均分: 141.5, 少于140分的人数: 2
</script></code></pre>

## 枚举 Enum
在实际编程中，有些数据的取值往往是有限的，只能是非常少量的整数，并且最好为每个值都取一个名字，以方便在后续代码中使用，比如一个星期只有七天，一年只有十二个月，一个班每周有六门课程等

以一周7天为例，我们可以用`#define`来给每天指定一个名字：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

#define Mon 1
#define Tues 2
#define Wed 3
#define Thurs 4
#define Fri 5
#define Sat 6
#define Sun 7

int main(){
    int day;
    scanf("%d", &day);
    switch(day){
        case Mon:
            printf("Monday\n");
            break;
        case Tues:
            printf("Tuesday\n");
            break;
        case Wed:
            printf("Wednesday\n");
            break;
        case Thurs:
            printf("Thursday\n");
            break;
        case Fri:
            printf("Friday\n");
            break;
        case Sat:
            printf("Saturday");
            break;
        case Sun:
            printf("Sunday\n");
            break;
        default:
            printf("Error!");
    }
    return 0;
}
</script></code></pre>

`#define`命令虽然能解决问题，但也带来了不小的副作用，导致宏名过多，代码松散，看起来总有点不舒服
C语言提供了一种`枚举(Enum)`类型，能够列出所有可能的取值，并给它们取一个名字

定义枚举类型：`enum NAME{var1, var2, var3...};`

例如：列出一个星期有几天：`enum week{ Mon, Tues, Wed, Thurs, Fri, Sat, Sun };`
可以看到，我们仅仅给出了名字，却没有给出名字对应的值，这是因为枚举值默认从 0 开始，往后逐个加 1 (递增)
也就是说，week 中的 Mon、Tues ...... Sun 对应的值分别为 0、1 ...... 6

我们也可以给每个名字指定一个值：`enum week{ Mon = 1, Tues = 2, Wed = 3, Thurs = 4, Fri = 5, Sat = 6, Sun = 7 };`
更为简单的方法是只给第一个名字指定值：`enum week{ Mon = 1, Tues, Wed, Thurs, Fri, Sat, Sun };`
这两种方式都是等效的，推荐使用后者

枚举是一种类型，通过它可以定义枚举变量：`enum week a, b, c;`

也可以在定义枚举类型的同时定义变量：`enum week{ Mon = 1, Tues, Wed, Thurs, Fri, Sat, Sun } a, b, c;`
然后就可以把列表中的值赋给变量：`a = Mon, b = Wed, c = Fri;`

判断用户输入的是星期几：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    enum week{ Mon = 1, Tues, Wed, Thurs, Fri, Sat, Sun } day;
    scanf("%d", &day);
    switch(day){
        case Mon: puts("Monday"); break;
        case Tues: puts("Tuesday"); break;
        case Wed: puts("Wednesday"); break;
        case Thurs: puts("Thursday"); break;
        case Fri: puts("Friday"); break;
        case Sat: puts("Saturday"); break;
        case Sun: puts("Sunday"); break;
        default: puts("Error!");
    }
    return 0;
}
</script></code></pre>

注意，枚举列表中的 Mon、Tues、Wed 标识符的作用范围是全局的(严格来说是main()函数内部)，不能再定义与他们相同名字的变量
Mon、Tues、Wed 等都是常量，不能对它们赋值，只能将它们的值赋给其他的变量

枚举和宏其实非常类似：宏在预处理阶段将名字替换成对应的值，枚举在编译阶段将名字替换成对应的值。我们可以将枚举理解为编译阶段的宏

对于上面的代码，在编译的某个时刻会变成类似下面的样子：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    enum week{ Mon = 1, Tues, Wed, Thurs, Fri, Sat, Sun } day;
    scanf("%d", &day);
    switch(day){
        case 1: puts("Monday"); break;
        case 2: puts("Tuesday"); break;
        case 3: puts("Wednesday"); break;
        case 4: puts("Thursday"); break;
        case 5: puts("Friday"); break;
        case 6: puts("Saturday"); break;
        case 7: puts("Sunday"); break;
        default: puts("Error!");
    }
    return 0;
}
</script></code></pre>

Mon、Tues、Wed 这些名字都被替换成了对应的数字
这意味着，Mon、Tues、Wed 等都不是变量，它们不占用数据区(常量区、全局数据区、栈区和堆区)的内存，而是直接被编译到命令里面，放到代码区，所以不能用 & 取得它们的地址，这就是枚举的本质

case 关键字后面必须是一个整数，或者是结果为整数的表达式，但不能包含任何变量，正是由于 Mon、Tues、Wed 这些名字最终会被替换成一个整数，所以它们才能放在 case 后面

枚举类型变量需要的是一整数，它的长度应该和 int 相同:
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(){
    enum Test{A = 1, B, C, D, E, F, G};
    enum Test test;
    printf("%d, %d, %d, %d\n", sizeof(enum Test), sizeof(test), sizeof(A), sizeof(int));
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [17:29:08]
$ gcc a.c

# root @ localhost in ~/test [17:29:10]
$ ./a.out
4, 4, 4, 4
</script></code></pre>

## 共用体
还有一种和结构体类似的构造类型，叫做`共用体(union)`，也叫做联合、联合体
<pre><code class="language-c line-numbers"><script type="text/plain">union 联合体名{
    成员列表
};
</script></code></pre>

区别在于：结构体的各个成员会占用不同的内存，互相之间没有影响；而共用体的所有成员占用同一段内存，修改一个成员会影响其余所有成员

`结构体`占用的内存`大于等于`所有成员占用的`内存的总和`(成员之间可能会存在缝隙)
`共用体`占用的内存`等于最长的成员占用的内存`
共用体使用了内存覆盖技术，**同一时刻只能保存一个成员的值，如果对新的成员赋值，就会把原来成员的值覆盖掉**

共用体也是一种自定义类型，也可以用来创建变量
<pre><code class="language-c line-numbers"><script type="text/plain">union Data{
    char c;
    int i;
    double f;
} data1, data2, data3;
</script></code></pre>

联合体 Data 中，最长的成员为double: 8字节，所以成员 c、i、f 共用这 8 个字节的内存，在同一个时刻，只能保存一个值

看下面这个例子：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

union Data{
    char ch;
    short sh;
    int n;
} data;

int main(){
    data.ch = 0x65;
    printf("%#x, %#x, %#x\n", data.ch, data.sh, data.n);
    data.ch = 0x12345678;
    printf("%#x, %#x, %#x\n", data.ch, data.sh, data.n);
    data.sh = 0x87654321;
    printf("%#x, %#x, %#x\n", data.ch, data.sh, data.n);
    data.n = 0x87654321;
    printf("%#x, %#x, %#x\n", data.ch, data.sh, data.n);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [17:51:42]
$ gcc a.c
a.c: In function ‘main’:
a.c:12:5: warning: overflow in implicit constant conversion [-Woverflow]
     data.ch = 0x12345678;
     ^
a.c:14:5: warning: overflow in implicit constant conversion [-Woverflow]
     data.sh = 0x87654321;
     ^

# root @ localhost in ~/test [17:51:43]
$ ./a.out
0x65, 0x65, 0x65
0x78, 0x78, 0x78
0x21, 0x4321, 0x4321
0x21, 0x4321, 0x87654321
</script></code></pre>

因为赋的值超过了 ch、sh 所能容纳的大小，所以高位会被截去，gcc也会警告，我们先不管
对于16进制数`0x12`：`0x1`占用4个bit，`0x2`占用4个bit，所以两个16进制数的字符就是一个字节长
从这个例子中可以知道，我这台电脑是`小端字节序`
- `小端字节序`：是指将数据的低位(如 0x1234 的 0x34)存放在内存的低地址，而数据的高位(如 0x1234 的 0x12)存放在内存的高地址上
- `大端字节序`：是指将数据的高位(如 0x1234 的 0x12)存放在内存的低地址，而数据的低位(如 0x1234 的 0x34)存放在内存的高地址上
一般而言：x86架构的cpu是小端模式，51单片机是大端模式，很多arm、dsp也是小端模式(部分arm可以由硬件来选择大小端)

共用体一般在单片机中应用较多，对于pc机，经常使用到的一个实例是：
现有一张关于学生信息和教师信息的表格，学生信息包括姓名、编号、性别、职业、分数，教师的信息包括姓名、编号、性别、职业、教学科目
f 和 m 分别表示女性和男性，s 表示学生，t 表示教师
可以看出，学生和教师所包含的数据是不同的。现在要求把这些信息放在同一个表格中，并设计程序输入人员信息然后输出

如果把每个人的信息都看作一个结构体变量的话，那么教师和学生的前 4 个成员变量是一样的，第 5 个成员变量可能是 score 或者 course。
当第 4 个成员变量的值是 s 的时候，第 5 个成员变量就是 score；当第 4 个成员变量的值是 t 的时候，第 5 个成员变量就是 course。

经过上面的分析，我们可以设计一个包含共用体的结构体，请看下面的代码：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>

#define TOTAL 4
#define STRLEN 30

struct{
    char *name;
    int num;
    char sex;
    char profession;
    union{
        float score;
        char *course;
    } sc;
} info[TOTAL];

int main(){
    for(int i=0; i<TOTAL; i++){
        info[i].name = (char *)malloc(sizeof(char) * STRLEN);
        printf("input info: ");
        scanf("%s %d %c %c", info[i].name, &info[i].num, &info[i].sex, &info[i].profession);
        if(info[i].profession == 's'){
            scanf("%f", &info[i].sc.score);
        }else if(info[i].profession == 't'){
            info[i].sc.course = (char *)malloc(sizeof(char) * STRLEN);
            scanf("%s", info[i].sc.course);
        }
    }
    printf("%13s%13s%13s%13s%13s\n", "Name", "Num", "Sex", "Profession", "Score/Course");
    for(int i=0; i<TOTAL; i++){
        if(info[i].profession == 's'){
            printf("%13s%13d%13c%13c%13.1f\n", info[i].name, info[i].num, info[i].sex, info[i].profession, info[i].sc.score);
        }else if(info[i].profession == 't'){
            printf("%13s%13d%13c%13c%13s\n", info[i].name, info[i].num, info[i].sex, info[i].profession, info[i].sc.course);
            free(info[i].sc.course);
        }
        free(info[i].name);
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [20:53:12]
$ gcc a.c

# root @ localhost in ~/test [20:53:13]
$ ./a.out
input info: HanXiaoXiao 501 f s 89.5
input info: ZhaoFeiYan 982 m s 95.0
input info: LiuZhenTao 109 f t English
input info: YanWeiMin 1011 m t math
         Name          Num          Sex   Profession Score/Course
  HanXiaoXiao          501            f            s         89.5
   ZhaoFeiYan          982            m            s         95.0
   LiuZhenTao          109            f            t      English
    YanWeiMin         1011            m            t         math
</script></code></pre>

## 位域
有些数据在存储时并不需要占用一个完整的字节，只需要占用一个或几个二进制位即可。
例如开关只有通电和断电两种状态，用 0 和 1 表示足以，也就是用一个二进位。
正是基于这种考虑，C语言又提供了一种叫做`位域`的数据结构

**在结构体定义时，我们可以指定某个成员变量所占用的二进制位数（Bit），这就是位域** 请看下面的例子：
<pre><code class="language-c line-numbers"><script type="text/plain">struct bs{
    unsigned int a;
    unsigned int b:4;
    unsigned char c:1;
};
</script></code></pre>

成员a占用4个字节，成员b占用4个bit，成员c占用1个bit
因为占用的内存有限，所以数值稍大一些就会发生溢出：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdbool.h>

struct{
    unsigned int a;
    unsigned int b:4;
    unsigned char c:1;
} test;

int main(){
    test.a = 0x56781234;
    test.b = 0x6;
    test.c = true;
    printf("%d\n", sizeof(test));
    printf("%#x, %#x, %#x\n", test.a, test.b, test.c);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/test [21:08:55]
$ gcc a.c

# root @ localhost in ~/test [21:08:58]
$ ./a.out
8
0x56781234, 0x6, 0x1
</script></code></pre>

C语言标准规定，位域的宽度不能超过它所依附的数据类型的长度
通俗地讲，成员变量都是有类型的，这个类型限制了成员变量的最大长度，后面的数字不能超过这个长度

我们可以这样认为，位域技术就是在成员变量所占用的内存中选出一部分位宽来存储数据

C语言标准还规定，只有有限的几种数据类型可以用于位域。
在 ANSI C 中，这几种数据类型是 int、signed int 和 unsigned int(int 默认就是 signed int)；到了 C99，_Bool 也被支持了

但编译器在具体实现时都进行了扩展，额外支持了 char、signed char、unsigned char 以及 enum 类型，所以上面的代码虽然不符合C语言标准，但它依然能够被编译器支持

C语言标准并没有规定位域的具体存储方式，不同的编译器有不同的实现，但它们都尽量压缩存储空间

**位域的对齐规则**
1) 当**相邻成员的类型相同**时
如果它们的**位宽之和小于类型的 sizeof 大小**，那么后面的成员紧邻前一个成员存储，直到不能容纳为止
如果它们的**位宽之和大于类型的 sizeof 大小**，那么后面的成员将从新的存储单元开始，其偏移量为类型大小的整数倍

2) 当**相邻成员的类型不同**时
不同的编译器有不同的实现方案，GCC 会压缩存储，而 VC/VS 不会

3) 如果**成员之间穿插着非位域成员**，那么不会进行压缩

通过上面的分析，我们发现位域成员往往不占用完整的字节，有时候也不处于字节的开头位置，因此使用 & 获取位域成员的地址是没有意义的
C语言也禁止这样做，地址是字节(Byte)的编号，而不是位(Bit)的编号

**无名位域**
位域成员可以没有名称，只给出数据类型和位宽，无名位域一般用来作填充或者调整成员位置，因为没有名称，无名位域不能使用
<pre><code class="language-c line-numbers"><script type="text/plain">struct Data{
    int a:12;
    int :20;
    int c:4;
};
</script></code></pre>

## typedef
C语言允许为一个数据类型起一个新的别名，就像给人起“绰号”一样
起别名的目的不是为了提高程序运行效率，而是为了编码方便
例如有一个结构体的名字是 Student，要想定义一个结构体变量就得这样写：
`struct Student stu;`
前面的`struct`看起来好多余，但是不加上又会报错
我们可以用 typedef 给来给它取个名字 Student：
`typedef struct Student Student;`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

struct Student{
    char *name;
    int age;
    float score;
};

typedef struct Student Student;

int main(){
    Student stu = {"Otokaze", 18, 111.5f};
    printf("Name: %s, Age: %d, Score: %.1f\n", stu.name, stu.age, stu.score);
    return 0;
}
</script></code></pre>

怎么理解`typedef struct Student Student;`？
首先，`typedef`可以看作是一个修饰符，是一个关键字，抛开它，剩下`struct Student Student;`
这不就是定义一个结构体变量的写法嘛，后面那个`Student`可以看作是定义的变量名，前面的`struct Student`是数据类型
只不过在`typedef`的修饰下，这个`Student`有了新的意义，它表示一种数据类型，即`struct Student`这个结构体类型

同样可以用 typedef 给数组、指针、指针数组、函数指针、二维数组指针、二级指针等定义别名：
`typedef int int_array[20];`；数组`int [20]`
`typedef int *int_ptr;`：指针`int *`，(可指向一个整数或一个整型数组)
`typedef int *ptr_array[20];`：指针数组`int * [20]`
`typedef int (*func_ptr)(int, int);`：函数指针，指向原型为`int func(int, int);`的函数
`typedef int (*array_ptr)[20];`：二维数组指针，指向类型为`int [20]`的数组
`typedef int **ptr_ptr;`：二级指针

**`typedef`和`#define`的区别**
`typedef` 在表现上有时候类似于 `#define`，但它和宏替换之间存在一个关键性的区别
正确思考这个问题的方法就是把 typedef 看成一种彻底的“封装”类型，声明之后不能再往里面增加别的东西

比如：`typedef long long int llong;`之后不能出现类似`unsigned llong var;`但是宏却可以

在连续定义几个变量的时候，typedef 能够保证定义的所有变量均为同一类型，而 #define 则无法保证

因为 `#define`仅仅是宏展开，字符替换而已，而`typedef`是一种封装类型

## const修饰符
有时候我们希望定义这样一种变量，它的值不能被改变，在整个作用域中都保持固定
例如，用一个变量来表示班级的最大人数，或者表示缓冲区的大小
为了满足这一要求，可以使用const关键字对变量加以限定
`const int MAX = 100;`
这样 MAX 的值就不能更改了，在后续的代码中，如果尝试更改它的内容，编译器会报错
我们经常将const变量称为常量，定义const常量的格式为：
`const type name = value;`
或者`type const name = value;`
推荐使用前者

由于常量一旦被创建后其值就不能再改变，所以常量必须在定义的同时赋值（初始化），后面的任何赋值行为都将引发错误

**const和指针一起用时**
- `const int *ptr`：表示指针指向的数据是只读的，但是指针本身可以改变指向
- `int const *ptr`：同上
- `int * const ptr`：表示指针本身是只读的，但是指针指向的数据是可以读写的

**const和函数形参**
在C语言中，单独定义 const 变量没有明显的优势，完全可以使用#define命令代替
const 通常用在函数形参中，如果形参是一个指针，为了防止在函数内部修改指针指向的数据，就可以用 const 来限制

在C语言标准库中，有很多函数的形参都被 const 限制了

我们自己在定义函数时也可以使用 const 对形参加以限制，例如查找字符串中某个字符出现的次数：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <string.h>

void strnchr(const char *str, char chr){
    int count = 0;
    int len = strlen(str);
    for(int i=0; i<len; i++){
        if(str[i] == chr){
            count++;
        }
    }
    printf("%d\n", count);
}

int main(){
    char str[] = "www.zfl9.com", chr = 'w';
    strnchr(str, chr);
    return 0;
}
</script></code></pre>

根据 strnchr() 的功能可以推断，函数内部要对字符串 str 进行遍历，不应该有修改的动作，用 const 加以限制，不但可以防止由于程序员误操作引起的字符串修改，还可以给用户一个提示，函数不会修改你提供的字符串，请你放心

**const 和非 const 类型转换**
当一个指针变量 str1 被 const 限制时，并且类似`const char *str1`这种形式，说明指针指向的数据不能被修改
如果将 str1 赋值给另外一个未被 const 修饰的指针变量 str2，就有可能发生危险
因为通过 str1 不能修改数据，而赋值后通过 str2 能够修改数据了，意义发生了转变，所以编译器不提倡这种行为，会给出错误或警告

也就是说，`const char *`和`char *`是不同的类型，不能将`const char *`类型的数据赋值给`char *`类型的变量
但反过来是可以的，编译器允许将`char *`类型的数据赋值给`const char *`类型的变量

这种限制很容易理解，`char *`指向的数据有读取和写入权限，而`const char *`指向的数据只有读取权限，降低数据的权限不会带来任何问题，但提升数据的权限就有可能发生危险

不过，我们依旧可以通过指针修改const变量的值，并且gcc没有警告：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

int main(void){
    const int array[5] = {1, 2, 3, 4, 5};
    int *ptr = (int *)&(*array);
    for(int i=0; i<5; i++){
        ptr[i] = array[i] + 100;
    }
    for(int i=0; i<5; i++){
        printf("array[%d] = %d\n", i, array[i]);
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~ [10:50:05]
$ gcc -std=gnu99 -Wall -Wextra -o main a.c

# root @ localhost in ~ [10:50:53]
$ ./main
array[0] = 101
array[1] = 102
array[2] = 103
array[3] = 104
array[4] = 105
</script></code></pre>
