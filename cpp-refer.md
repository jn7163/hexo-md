---
title: C++ 引用
date: 2017-08-25 07:07:00
categories:
- cpp
tags:
- cpp
keywords: C++ 引用 reference 指针
---

> 
C++ 引用，什么是引用，引用的本质是什么，和指针有什么区别？

<!-- more -->

## 引用的概念
参数的传递本质上是一次赋值的过程，赋值就是对内存进行拷贝；所谓内存拷贝，是指将一块内存上的数据复制到另一块内存上；

对于像 char、bool、int、float 等基本类型的数据，它们占用的内存往往只有几个字节，对它们进行内存拷贝非常快速；
而数组、结构体、对象是一系列数据的集合，数据的数量没有限制，可能很少，也可能成千上万，对它们进行频繁的内存拷贝可能会消耗很多时间，拖慢程序的执行效率；

C/C++禁止在函数调用时直接传递数组的内容，而是强制传递数组指针；而对于结构体和对象没有这种限制，调用函数时既可以传递指针，也可以直接传递内容；为了提高效率，我们一般都是传递指针；

但是在 C++ 中，我们有了一种比指针更加便捷的传递聚合类型数据的方式，那就是`引用（Reference）`；

> 
在 C/C++ 中，我们将 char、int、float 等由语言本身支持的类型称为**基本类型**，将`数组`、`结构体`、`类（对象）`等由基本类型组合而成的类型称为**聚合类型**；

**引用（Reference）是 C++ 相对于C语言的又一个扩充；引用可以看做是数据的一个别名，通过这个别名和原来的名字都能够找到这份数据**

引用的定义方式：`datatype &ref_name = origin_name;`
`datatype`是被引用的数据的类型，`ref_name`是引用的名称（新名称），`origin_name`是被引用的数据（原来的名称）；

**引用必须在定义的同时初始化，并且以后也要从一而终，不能再引用其它数据，这有点类似于常量（const 变量）**

引用的实例：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

int main() {
    int a = 10;
    int &b = a;
    printf("a: %d, b: %d, &a: %p, &b: %p\n", a, b, &a, &b);
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [8:03:13]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [8:03:15]
$ ./a.out
a: 10, b: 10, &a: 0x7fff3e3083ec, &b: 0x7fff3e3083ec
</script></code></pre>


可以发现，a 和 b 的地址是一样的，访问该内存上的数据使用哪个名字都行；
注意，在定义引用的时候需要加`&`，在使用的时候不用`&`，如果使用时加`&`表示取地址；

由于引用 b 和原始变量 a 都是指向同一地址，所以通过引用也可以修改原始变量中所存储的数据；
如果不希望通过引用来修改原来的数据，可以在定义的时候添加`const`进行限制：`const int &b = a;`，称为`常引用`；

**引用作为函数参数**
在定义或声明函数时，我们可以将函数的形参指定为引用的形式，这样在调用函数时就会将实参和形参绑定在一起，让它们都指代同一份数据；如此一来，如果在函数体中修改了形参的数据，那么实参的数据也会被修改，从而拥有“在函数内部影响函数外部数据”的效果；

先来看一个交换值的例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

void swap1(int a, int b) {
    int tmp = a;
    a = b;
    b = tmp;
}

void swap2(int *a, int *b) {
    int tmp = *a;
    *a = *b;
    *b = tmp;
}

void swap3(int &a, int &b) {
    int tmp = a;
    a = b;
    b = tmp;
}

int main() {
    int x, y;

    x = 1; y = 2;
    printf("swap1_before: x = %d, y = %d\n", x, y);
    swap1(x, y);
    printf("swap1_after: x = %d, y = %d\n\n", x, y);

    x = 1; y = 2;
    printf("swap2_before: x = %d, y = %d\n", x, y);
    swap2(&x, &y);
    printf("swap2_after: x = %d, y = %d\n\n", x, y);

    x = 1; y = 2;
    printf("swap3_before: x = %d, y = %d\n", x, y);
    swap3(x, y);
    printf("swap3_after: x = %d, y = %d\n", x, y);

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [8:18:09]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [8:18:11]
$ ./a.out
swap1_before: x = 1, y = 2
swap1_after: x = 1, y = 2

swap2_before: x = 1, y = 2
swap2_after: x = 2, y = 1

swap3_before: x = 1, y = 2
swap3_after: x = 2, y = 1
</script></code></pre>


`swap1(int, int)`值传递：由于形参a、b的作用范围仅限于函数内部，它们拥有独立的内存，和x、y互不影响；
`swap2(int*, int*)`指针传递：调用函数的时候分别将x、y的地址传递给形参a、b，这样就能在函数内部通过指针间接交换两个数值；
`swap3(int&, int&)`引用传递：调用函数的时候分别将形参a、b绑定到x、y指定的数据，这样就能在函数内部交换两个数值了；

从以上代码的编写中可以发现，按引用传参在使用形式上比指针更加直观；鼓励大量使用引用，一般可以代替指针，当然指针在 C++ 中也是不可或缺，这两者应结合使用；

**引用作为函数返回值**
引用除了可以作为函数形参，还可以作为函数返回值：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

int & func(int &n) {
    n += 10;
    return n;
}

int main() {
    int a = 10;
    int &b = func(a);   // 传递a的引用给b
    printf("a = %d, b = %d, &a = %p, &b = %p\n", a, b, &a, &b);
    b++;
    printf("a = %d, b = %d\n", a, b);

    int x = 10;
    int y = func(x);    // 用y接收func()的返回值
    printf("x = %d, y = %d, &x = %p, &y = %p\n", x, y, &x, &y);
    y++;
    printf("x = %d, y = %d\n", x, y);

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [8:38:12]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [8:38:26]
$ ./a.out
a = 20, b = 20, &a = 0x7fff159e9214, &b = 0x7fff159e9214
a = 21, b = 21
x = 20, y = 20, &x = 0x7fff159e9218, &y = 0x7fff159e921c
x = 20, y = 21
</script></code></pre>


将引用作为函数返回值时应该注意一个小问题，就是不能返回局部数据（例如局部变量、局部对象、局部数组等）的引用，因为当函数调用完成后局部数据就会被销毁，有可能在下次使用时数据就不存在了，C++ 编译器检测到该行为时也会给出警告；

## 引用的本质
虽然通过前面的内容我们知道了怎么使用引用，但又感觉糊里糊涂，不明白它到底是什么，它和指针有点相似，但又不是一个东西；

先来看一下前面的这个例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

int main() {
    int a = 10;
    int &b = a;
    printf("a = %d, b = %d, &a = %p, &b = %p\n", a, b, &a, &b);
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [8:43:08]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [8:43:09]
$ ./a.out
a = 10, b = 10, &a = 0x7ffc296ecc0c, &b = 0x7ffc296ecc0c
</script></code></pre>


我们知道，变量是要占用内存的，虽然我们称 b 为变量，但是通过`&b`获取到的却不是 b 的地址，而是 a 的地址，这会让我们觉得 b 这个变量不占用独立的内存，它和 a 指代的是同一份内存；但是实际上并不是这样，我们来看这个例子：

<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

int flag = 10;

class A {
public:
    A() : m_a(0), m_b(flag) {}
private:
    int m_a;
    int &m_b;
};

int main() {
    A *a = new A;
    printf("sizeof(A): %lu\n", sizeof(A));

    // 通过指针获取a的两个private成员
    printf("m_a: %d, m_b: %#lx, *m_b: %d, &flag: %p, flag: %d\n",
            *(int *)a,
            *(unsigned long *)((unsigned long)a + 8),
            *(int *)*(unsigned long *)((unsigned long)a + 8),
            &flag,
            flag);

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:11:37]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [9:12:20]
$ ./a.out
sizeof(A): 16
m_a: 0, m_b: 0x9b3f5a6038, *m_b: 10, &flag: 0x9b3f5a6038, flag: 10
</script></code></pre>


> 
成员变量是 private 属性的，不能直接通过对象来访问，但是借助强大的指针和类型转换，我们依然可以得到它的内容，只不过这种方法有点蹩脚；

从运行结果可以看出：
- 成员变量 m_b 是占用内存的，如果不占用的话，sizeof(A)的结果应该为 4；
- m_b 存储的内容是 0x9b3f5a6038，也即变量 flag 的地址；

这里注意一下，因为笔者使用的是64位的Linux系统，所以一个指针变量的长度为64位，即8字节长；
并且，类和结构体都遵循内存对齐规则，如果你不熟悉内存对齐，请戳 [C语言 struct内存对齐](https://www.zfl9.com/c-struct.html)；

这说明 m_b 的实现和指针非常类似；如果将 m_b 定义为`int *`类型的指针，并在构造函数中让它指向 flag，那么 m_b 占用的内存也是 8 个字节，存储的内容也是 flag 的地址；

其实引用只是对指针进行了简单的封装，它的底层依然是通过指针实现的，引用占用的内存和指针占用的内存长度一样，在 32 位环境下是 4 个字节，在 64 位环境下是 8 个字节，之所以不能获取引用的地址，是因为编译器进行了内部转换；

以下面的语句为例：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

int main() {
    int a = 10;
    int &b = a;
    printf("a = %d, b = %d, &a = %p, &b = %p\n", a, b, &a, &b);

    /* 在编译的某个时候，被转换为如下形式：
    int a = 10;
    int *b = &a;
    printf("a = %d, b = %d, &a = %p, &b = %p\n", a, *b, &a, *&b);
    */

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:28:56]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [9:28:58]
$ ./a.out
a = 10, b = 10, &a = 0x7fffdaa16bcc, &b = 0x7fffdaa16bcc
</script></code></pre>


使用`&b`取地址时，编译器会对代码进行隐式的转换，使得代码输出的是 a 的地址，而不是 b 的地址，这就是为什么获取不到引用变量的地址的原因；
也就是说，不是变量 b 不占用内存，而是编译器不让获取它的地址；

当引用作为函数参数时，也会有类似的转换，这里就不再重复演示了；

引用虽然是基于指针实现的，但它比指针更加易用，从上面的例子也可以看出来，通过指针获取数据时需要加`*`，书写麻烦，而引用不需要，它和普通变量的使用方式一样；
C++ 的发明人 Bjarne Stroustrup 也说过，他在 C++ 中引入引用的直接目的是为了让代码的书写更加漂亮，尤其是在运算符重载中，不借助引用有时候会使得运算符的使用很麻烦；

**引用和指针的其他区别**
1) 引用必须在定义时初始化，并且以后也要从一而终，不能再指向其他数据；而指针没有这个限制，指针在定义时不必赋值，以后也能指向任意数据；

2) 可以有 const 指针，但是没有 const 引用；也就是说，引用变量不能定义为下面的形式：
`int a = 10;`
`int &const b = a;`
因为 b 本来就不能改变指向，加上 const 多此一举；

3) 指针可以有多级，但是引用只能有一级；例如，`int **p`是合法的，而`int &&r`是不合法的；
如果希望定义一个引用变量来指代另外一个引用变量，那么也只需要加一个&，如下所示：
`int a = 10;`
`int &r = a;`
`int &rr = r;`

4) 指针和引用的自增、自减运算意义不一样；对指针使用`++`表示指向下一份数据，对引用使用`++`表示它所指代的数据本身加1；自减（`--`）也是类似的道理；

## 引用不能绑定到临时数据
我们知道，指针就是数据或代码在内存中的地址，指针变量指向的就是内存中的数据或代码；
这里有一个关键词需要强调，就是**内存**，**指针只能指向内存，不能指向寄存器或者硬盘，因为寄存器和硬盘没法寻址**

其实 C++ 代码中的大部分内容都是放在内存中的，例如*定义的变量*、*创建的对象*、*字符串常量*、*函数形参*、*函数体本身*、*new*或*malloc()*分配的内存等，这些内容都可以用&来获取地址，进而用指针指向它们；
除此之外，还有一些我们平时不太留意的临时数据，例如*表达式的结果*、*函数的返回值*等，它们可能会放在内存中，也可能会放在寄存器中。一旦它们被放到了寄存器中，就没法用&获取它们的地址了，也就没法用指针指向它们了；

下面的代码演示了表达式所产生的临时结果：
`int n = 100, m = 200;`
`int *p1 = &(m + n);`：`m + n`的结果为 300
`int *p2 = &(n + 100);`：`n + 100`的结果为 200
`bool *p4 = &(m < n);`：`m < n`的结果为 false
这些表达式的结果都会被放到*寄存器*中，尝试用&获取它们的地址都是错误的；

下面的代码演示了函数返回值所产生的临时结果：
<pre><code class="language-cpp line-numbers"><script type="text/plain">int func() {
    int n = 100;
    return n;
}
int *p = &(func());
</script></code></pre>

func() 的返回值 100 也会被放到寄存器中，也没法用&获取它的地址；

**什么样的临时数据会放到寄存器中**
寄存器离 CPU 近，并且速度比内存快，将临时数据放到寄存器是为了加快程序运行；
但是寄存器的数量是非常有限的，容纳不下较大的数据，所以只能将较小的临时数据放在寄存器中；
int、double、bool、char 等基本类型的数据往往不超过 8 个字节，用一两个寄存器就能存储，所以这些类型的临时数据通常会放到寄存器中；
而对象、结构体变量是自定义类型的数据，大小不可预测，所以这些类型的临时数据通常会放到内存中；

**关于常量表达式**
诸如`100`、`200+34`、`34.5*23`、`3+7/3`等不包含变量的表达式称为*常量表达式（Constant expression）*；
常量表达式由于不包含变量，没有不稳定因素，所以在编译阶段就能求值；编译器不会分配单独的内存来存储常量表达式的值，而是将常量表达式的值和代码合并到一起，放到虚拟地址空间中的代码区；从汇编的角度看，常量表达式的值就是一个立即数，会被“硬编码”到指令中，不能寻址；

总起来说，常量表达式的值虽然在内存中，但是没有办法寻址，所以也不能使用&来获取它的地址，更不能用指针指向它；

**引用也不能指代临时数据**
引用和指针在本质上是一样的，引用仅仅是对指针进行了简单的封装；
引用和指针都不能绑定到无法寻址的临时数据，并且 C++ 对引用的要求更加严格，在某些编译器下甚至连放在内存中的临时数据都不能指代；

下面的代码中，我们将引用绑定到了临时数据：
<pre><code class="language-cpp line-numbers"><script type="text/plain">typedef struct{
    int a;
    int b;
} S;

int func_int(){
    int n = 100;
    return n;
}

S func_s(){
    S a;
    a.a = 100;
    a.b = 200;
    return a;
}

S operator+(const S &A, const S &B){
    S C;
    C.a = A.a + B.a;
    C.b = A.b + B.b;
    return C;
}

int main(){
    //下面的代码在GCC和Visual C++下都是错误的
    int m = 100, n = 36;
    int &r1 = m + n;
    int &r2 = m + 28;
    int &r3 = 12 * 3;
    int &r4 = 50;
    int &r5 = func_int();

    //下面的代码在GCC下是错误的，在Visual C++下是正确的
    S s1 = {23, 45};
    S s2 = {90, 75};
    S &r6 = func_s();
    S &r7 = s1 + s2;

    return 0;
}
</script></code></pre>


> 
在 GCC 下，引用不能指代任何临时数据，不管它保存到哪里；
在 Visual C++ 下，引用只能指代位于内存中（非代码区）的临时数据，不能指代寄存器中的临时数据；

**引用作为函数参数**
当引用作为函数参数时，有时候很容易给它传递临时数据：
<pre><code class="language-cpp line-numbers"><script type="text/plain">bool isOdd(int &n) {
    if (n/2 == 0) {
        return false;
    } else {
        return true;
    }
}

int main(){
    int a = 100;
    isOdd(a);  //正确
    isOdd(a + 9);  //错误
    isOdd(27);  //错误
    isOdd(23 + 55);  //错误
    return 0;
}
</script></code></pre>


isOdd()函数用来判断一个数是否为奇数，它的参数是引用类型，只能传递变量，不能传递常量或者表达式；但用来判断奇数的函数不能接受一个数字又让人感觉很奇怪，所以类似这样的函数应该坚持使用值传递，而不是引用传递；

## const引用和临时变量
引用不能绑定到临时数据，这在大多数情况下是正确的，但是当使用 const 关键字对引用加以限定后，引用就可以绑定到临时数据了；
<pre><code class="language-cpp line-numbers"><script type="text/plain">typedef struct{
    int a;
    int b;
} S;

int func_int(){
    int n = 100;
    return n;
}

S func_s(){
    S a;
    a.a = 100;
    a.b = 200;
    return a;
}

S operator+(const S &A, const S &B){
    S C;
    C.a = A.a + B.a;
    C.b = A.b + B.b;
    return C;
}

int main(){
    int m = 100, n = 36;
    const int &r1 = m + n;
    const int &r2 = m + 28;
    const int &r3 = 12 * 3;
    const int &r4 = 50;
    const int &r5 = func_int();

    S s1 = {23, 45};
    S s2 = {90, 75};
    const S &r6 = func_s();
    const S &r7 = s1 + s2;

    return 0;
}
</script></code></pre>


这段代码在 GCC 和 Visual C++ 下都能够编译通过，这是因为将常引用绑定到临时数据时，编译器采取了一种妥协机制：编译器会为临时数据创建一个新的、无名的临时变量，并将临时数据放入该临时变量中，然后再将引用绑定到该临时变量；注意，临时变量也是变量，所有的变量都会被分配内存；

**为什么编译器为常引用创建临时变量是合理的，而为普通引用创建临时变量就不合理呢？**
1) 我们知道，将引用绑定到一份数据后，就可以通过引用对这份数据进行操作了，包括读取和写入（修改）；尤其是写入操作，会改变数据的值。而临时数据往往无法寻址，是不能写入的，即使为临时数据创建了一个临时变量，那么修改的也仅仅是临时变量里面的数据，不会影响原来的数据，这样就使得引用所绑定到的数据和原来的数据不能同步更新，最终产生了两份不同的数据，失去了引用的意义；

2) const 引用和普通引用不一样，我们只能通过 const 引用读取数据的值，而不能修改它的值，所以不用考虑同步更新的问题，也不会产生两份不同的数据，为 const 引用创建临时变量反而会使得引用更加灵活和通用；

以上节的 isOdd() 函数为例：
<pre><code class="language-cpp line-numbers"><script type="text/plain">bool isOdd(const int &n){  //改为常引用
    if(n/2 == 0){
        return false;
    }else{
        return true;
    }
}
</script></code></pre>

由于在函数体中不会修改 n 的值，所以可以用 const 限制 n，这样一来，下面的函数调用就都是正确的了：
<pre><code class="language-cpp line-numbers"><script type="text/plain">int a = 100;
isOdd(a);  //正确
isOdd(a + 9);  //正确
isOdd(27);  //正确
isOdd(23 + 55);  //正确
</script></code></pre>

对于第 2 行代码，编译器不会创建临时变量，会直接绑定到变量 a；
对于第 3~5 行代码，编译器会创建临时变量来存储临时数据；也就是说，编译器只有在必要时才会创建临时变量；

## const引用与类型转换
不同类型的数据占用的内存数量不一样，处理方式也不一样，指针的类型要与它指向的数据的类型严格对应；

**类型转换的本质**
我们知道，数据是放在内存中的，变量（以及指针、引用）是给这块内存起的名字，有了变量就可以找到并使用这份数据；但问题是，该如何使用呢？

诸如数字、文字、符号、图形、音频、视频等数据都是以二进制形式存储在内存中的，它们并没有本质上的区别，那么，00010000 该理解为数字 16 呢，还是图像中某个像素的颜色呢，还是要发出某个声音呢？如果没有特别指明，我们并不知道；

也就是说，内存中的数据有多种解释方式，使用之前必须要确定；这种「确定数据的解释方式」的工作就是由**数据类型（Data Type）**来完成的；

例如`int a;`，表明，a 这份数据是整数，不能理解为像素、声音、视频等；

顾名思义，数据类型用来说明数据的类型，确定了数据的解释方式，让计算机和程序员不会产生歧义；
C/C++ 支持多种数据类型，包括内置类型（例如 int、double、bool 等）和自定义类型（结构体类型和类类型）；

**所谓数据类型转换，就是对数据所占用的二进制位做出重新解释**；
**如果有必要，在重新解释的同时还会修改数据，改变它的二进制位**；

对于*隐式类型转换*，编译器可以根据已知的转换规则来决定是否需要修改数据的二进制位；而对于*强制类型转换*，由于没有对应的转换规则，所以能做的事情仅仅是重新解释数据的二进制位，但无法对数据的二进制位做出修正；这就是隐式类型转换和强制类型转换最根本的区别；


比如，`int`和`float`类型的指针，是不能随便互相指向对方的数据的，因为整数和浮点数在内存中的存储是大不一样的：
- 对于 int，程序把最高 1 位作为符号位，把剩下的 31 位作为数值位；
- 对于 float，程序把最高 1 位作为符号位，把最低的 23 位作为尾数位，把中间的 8 位作为指数位；

让指针指向「相关的（相近的）但不是严格对应的」类型的数据，表面上看起来是合理的，但是细思极恐，这样会给程序留下很多意想不到的、难以发现的 Bug，所以编译器禁止这样做是非常合理的；
当然，如果你想通过强制类型转换达到这个目的，那编译器也会放任不管，给你自由发挥的余地；

引用（Reference）和指针（Pointer）在本质上是一样的，引用仅仅是对指针进行了简单的封装，「类型严格一致」这条规则同样也适用于引用；

**const 引用与类型转换**
「类型严格一致」是为了防止发生让人匪夷所思的操作，但是这条规则仅仅适用于普通引用，当对引用添加 const 限定后，情况就又发生了变化，编译器允许引用绑定到类型不一致的数据；请看下面的代码：
<pre><code class="language-cpp line-numbers"><script type="text/plain">int n = 100;
int &r1 = n;  //正确
const float &r2 = n;  //正确

char c = '@';
char &r3 = c;  //正确
const int &r4 = c;  //正确
</script></code></pre>


当引用的类型和数据的类型不一致时，如果它们的类型是相近的，并且遵守「数据类型的自动转换」规则，那么编译器就会创建一个临时变量，并将数据赋值给这个临时变量（这时候会发生自动类型转换），然后再将引用绑定到这个临时的变量，这与「将 const 引用绑定到临时数据时」采用的方案是一样的；

注意，临时变量的类型和引用的类型是一样的，在将数据赋值给临时变量时会发生自动类型转换；
当引用的类型和数据的类型不遵守「数据类型的自动转换」规则，那么编译器将报错，绑定失败；

结合上节讲到的知识，总结起来说，给引用添加 const 限定后，不但可以将引用绑定到临时数据，还可以将引用绑定到类型相近的数据，这使得引用更加灵活和通用，它们背后的机制都是临时变量；

**引用类型的函数形参请尽可能的使用 const**
当引用作为函数参数时，如果在函数体内部不会修改引用所绑定的数据，那么请尽量为该引用添加 const 限制；

概括起来说，将引用类型的形参添加 const 限制的理由有三个：
- 使用 const 可以避免无意中修改数据的编程错误；
- 使用 const 能让函数接收 const 和非 const 类型的实参，否则将只能接收非 const 类型的实参；
- 使用 const 引用能够让函数正确生成并使用临时变量；
