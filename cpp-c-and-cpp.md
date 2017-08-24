---
title: C++ 入门初步
date: 2017-08-23 16:07:00
categories:
- cpp
tags:
- cpp
keywords: c++ c C++入门初步 c++与c语言
---

> 
C++ 入门初步，C语言与C++的区别与联系，C++类和对象、命名空间等

<!-- more -->

## C++类和对象
C++是一门`面向对象`的编程语言，理解C++，首先要理解`类`（Class）和`对象`（Object）这两个概念；

`C++中的类（Class）可以看做C语言中结构体（Struct）的升级版`；

C语言的结构体是一种构造类型，可以包含若干成员变量，每个成员变量的类型可以不同；可以通过结构体来定义结构体变量，每个变量拥有相同的性质；
C++中的类也是一种构造类型，但是进行了一些扩展，类的成员不但可以是变量，还可以是函数；通过类定义出来的变量也有特定的称呼，叫做“对象”；

c语言中的结构：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>

struct Student {
    const char *name;
    int age;
    float score;
};

void print(struct Student stu) {
    printf("name: %s, age: %d, score: %.1f\n", stu.name, stu.age, stu.score);
}

int main() {
    struct Student stu = {"Otokaze", 18, 135.5};
    print(stu);
    return 0;
}
</script></code></pre>


c++中的类：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

class Student {
    public:
        Student(const char *name, int age, float score) : m_name(name), m_age(age), m_score(score) {}
        void print() const;
    private:
        const char *m_name;
        int m_age;
        float m_score;
};

void Student::print() const {
    cout << "name: " << m_name << ", age: " << m_age << ", score: " << m_score << endl;
}

int main() {
    Student stu("Otokaze", 18, 135.5);
    stu.print();
    return 0;
}
</script></code></pre>


> 
C语言中的 struct 只能包含变量，而C++中的 class 除了可以包含变量，还可以包含函数；
print() 是用来处理成员变量的函数，在C语言中，我们将它放在了 struct Student 外面，它和成员变量是分离的；
而在C++中，我们将它放在了 class Student 内部，使它和成员变量聚集在一起，看起来更像一个整体；

结构体和类都可以看做一种由用户自己定义的复杂数据类型，在C语言中可以通过结构体名来定义变量，在C++中可以通过类名来定义变量；
不同的是，通过结构体定义出来的变量还是叫变量，而通过类定义出来的变量有了新的名称，叫做`对象`（Object）；

类只是一张图纸，起到说明的作用，不占用内存空间；对象才是具体的零件，要有地方来存放，才会占用内存空间；
在C++中，通过类名就可以创建对象，即将图纸生产成零件，这个过程叫做`类的实例化`，因此也称对象是类的一个`实例`（Instance）
有些资料也将类的`成员变量`称为`属性`（Property），将类的`成员函数`称为`方法`（Method）；

> 
面向对象编程在代码执行效率上绝对没有任何优势，它的主要目的是方便程序员组织和管理代码，快速梳理编程思路，带来编程思想上的革新；

面向对象编程是针对开发中大规模的程序而提出来的，目的是提高软件开发的效率；不要把面向对象和面向过程对立起来，面向对象和面向过程不是矛盾的，而是各有用途、互为补充的；

## 运行C++程序
c++和c语言类似，都要经过`预处理`、`编译`、`汇编`、`链接`四个阶段，程序才能运行；

GNU编译器套件（GCC），提供了`gcc`、`g++`两个命令来分别编译c、c++程序，`g++`命令的用法及参数基本同`gcc`；

c语言源文件的后缀为`.c`，而c++源文件的后缀通常为`.cpp`；

## C++命名空间
一个中大型软件往往由多名程序员共同开发，会使用大量的变量和函数，不可避免地会出现变量或函数的命名冲突；当所有人的代码都测试通过，没有问题时，将它们结合到一起就有可能会出现命名冲突；

例如小李和小韩都参与了一个文件管理系统的开发，它们都定义了一个全局变量fp，用来指明当前打开的文件，将他们的代码整合在一起编译时，很明显编译器会提示 fp 重复定义（Redefinition）错误；

为了解决合作开发时的命名冲突问题，C++引入了`命名空间（Namespace）`的概念：
<pre><code class="language-cpp line-numbers"><script type="text/plain">namespace ns1 {
    FILE *fp = nullptr;
}

namespace ns2 {
    FILE *fp = nullptr;
}
</script></code></pre>


小李与小韩各自定义了自己的命名空间，此时再将他们的 fp 变量放在一起编译就不会有任何问题；

`namespace`是C++中的关键字，用来定义一个命名空间，语法：`namespace ns_name { ... }`；
`ns_name`是命名空间的`名字`，它里面可以包含`变量`、`函数`、`类`、`typedef`、`#define`、`命名空间`等，最后由`{}`包围；

使用命名空间的成员时，需要指明使用的命名空间，如：
<pre><code class="language-cpp line-numbers"><script type="text/plain">// 1. 直接指定
ns1::fp = fopen("a.file", "r");
ns2::fp = fopen("b.file", "w");

// 2. 使用using ns1::fp
using ns1::fp;
fp = fopen("a.file", "r");
ns2::fp = fopen("b.file", "w");

// 3. 使用using namespace ns1
using namespace ns1;
fp = fopen("a.file", "r");
ns2::fp = fopen("b.file", "w");
</script></code></pre>


`::`是一个新符号，称为`域解析操作符`，在C++中用来指明要使用的命名空间；

## 标准命名空间std
C++是在C语言的基础上开发的，早期的 C++ 还不完善，不支持命名空间，没有自己的编译器，而是将 C++ 代码翻译成C代码，再通过C编译器完成编译；
这个时候的 C++ 仍然在使用C语言的库，stdio.h、stdlib.h、string.h 等头文件依然有效；此外 C++ 也开发了一些新的库，增加了自己的头文件，和C语言一样，C++ 头文件仍然以`.h`为后缀，它们所包含的类、函数、宏等都是全局范围的；

后来 C++ 引入了命名空间的概念，计划重新编写库，将类、函数、宏等都统一纳入一个命名空间，这个命名空间的名字就是`std`；std 是 standard 的缩写，意思是“标准命名空间”；

但是这时已经有很多用老式 C++ 开发的程序了，它们的代码中并没有使用命名空间，直接修改原来的库会带来一个很严重的后果：程序员会因为不愿花费大量时间修改老式代码而极力反抗，拒绝使用新标准的 C++ 代码；

C++ 开发人员想了一个好办法，保留原来的库和头文件，它们在 C++ 中可以继续使用，然后再把原来的库复制一份，在此基础上稍加修改，把类、函数、宏等纳入命名空间 std 下，就成了新版 C++ 标准库；
这样共存在了两份功能相似的库，使用了老式 C++ 的程序可以继续使用原来的库，新开发的程序可以使用新版的 C++ 库；

为了避免头文件重名，`新版 C++ 库也对头文件的命名做了调整，去掉了后缀.h`：
所以老式 C++ 的`iostream.h`变成了`iostream`，`fstream.h`变成了`fstream`；
而对于原来C语言的头文件，也采用同样的方法，但在每个名字前还要添加一个`c`字母，所以C语言的`stdio.h`变成了`cstdio`，`stdlib.h`变成了`cstdlib`；

需要注意的是，**旧的 C++ 头文件是官方所反对使用的，已明确提出不再支持**；**旧的C头文件仍然可以使用，以保持对C的兼容性**；
实际上，编译器开发商不会停止对客户现有软件提供支持，可以预计，旧的 C++ 头文件在未来数年内还是会被支持；

**总结的 C++ 头文件的现状**
1) `旧 C++ 头文件`：如 iostream.h、fstream.h 等将会继续被支持，尽管它们不在官方标准中。这些头文件的内容`不在命名空间std中`；
2) `新 C++ 头文件`：如 iostream、fstream 等包含的基本功能和对应的旧版头文件相似，但头文件的内容`在命名空间std中`；
3) `C 头文件`：如 stdio.h、stdlib.h 等继续被支持。头文件的内容`不在命名空间std中`；
4) `C++版 C头文件`：如 cstdio、cstdlib 这样的名字。它们提供的内容和相应的旧的C头文件相同，只是内容`在命名空间std中`；

**对于不带.h的头文件，所有的符号都位于命名空间 std 中，使用时需要声明`命名空间std`**；
**对于带.h的头文件，没有使用任何命名空间，所有符号都位于`全局作用域`**；

**std命名空间**
打印HelloWorld：
第一种方式：`std::cout << "Hello, World!" << std::endl;`
第二种方式：`using std::cout; using std::endl;`，`cout << "Hello, World!" << endl;`
第三种方式：`using namespace std;`，`cout << "Hello, World" << endl;`

很显然，第三种方式最方便，但是这样也就违背了C++引入命名空间的初衷；
因为C++的标准库都在命名空间`std`中，C++的标准库是非常庞大的，极易导致命名冲突；

**推荐使用`std::cout`或`using std::cout`形式，不推荐使用`using namespace std`**

不过为了方便，我还是忍不住选择了第三种，并且尽量不与标准库的命名产生冲突；

## C++标准输入输出
在C语言中，我们通常使用`scanf/printf`系列函数进行数据的输入输出操作；
在C++中，这套函数依旧能够使用，不过C++又增加了一套新的、容易使用的输入输出库；

头文件：`iostream`，标准输入`cin`、标准输出`cout`、标准错误`cerr`：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <cstdio>

using namespace std;

int main() {
    int n;
    char str[128];

    // C style
    printf("n, str: ");
    scanf("%d %s", &n, str);
    printf("n = %d, str = %s\n", n, str);

    // C++ style
    cout << "n, str: ";
    cin >> n >> str;
    cout << "n = " << n << ", str = " << str << endl;

    return 0;
}
</script></code></pre>


`cout`和`cin`都是 C++ 的`内置对象`，而不是关键字；
C++ 库定义了大量的类（Class），程序员可以使用它们来创建对象，**cout 和 cin 就分别是 ostream 和 istream 类的对象**；只不过它们是由标准库的开发者提前创建好的，可以直接拿来使用。这种在 C++ 中提前创建好的对象称为`内置对象`；

使用`cout`进行输出时需要紧跟`<<`运算符，使用`cin`进行输入时需要紧跟`>>`运算符；
这两个运算符可以自行分析所处理的数据类型，因此无需像使用 scanf 和 printf 那样给出格式控制字符串；

> 
`>>`、`<<`分别是左移、右移运算符，属于`位运算符`，之所以能够用于`cin`、`cout`的输入输出，是因为`cin`和`cout`对这两个运算符进行了重载（`运算符重载`）

`endl`和`\n`很相似，但是`endl`除了换行之外，还会刷新输出缓冲区，而`\n`则不会；endl实际上是一个函数模板；

**scanf/printf、cin/cout的取舍**
对于一般数据，如果不需要进行格式控制，那么推荐使用`cin/cout`方式进行输入输出，因为可以自动判断数据类型，并且可以通过`运算符重载`输出自定义的类，结构体等聚合类型；
对于需要大量控制输入输出格式的数据，那么我还是选择`scanf/printf`函数，因为`cin/cout`的格式化输入输出太麻烦了，我实在接受不了；

应用举例：
如同基本类型(int, char, float, char *等)一样，输入/输出自定义的类型：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <string>

using namespace std;

class Student {
    public:
        Student(string name = "", int age = 0, float score = 0.0f) : m_name(name), m_age(age), m_score(score) {}
    public:
        friend istream & operator>>(istream &in, Student &stu);
        friend ostream & operator<<(ostream &out, Student &stu);
    private:
        string m_name;
        int m_age;
        float m_score;
};

istream & operator>>(istream &in, Student &stu) {
    in >> stu.m_name >> stu.m_age >> stu.m_score;
    return in;
}

ostream & operator<<(ostream &out, Student &stu) {
    out << "name: " << stu.m_name << ", age: " << stu.m_age << ", score: " << stu.m_score << endl;
    return out;
}

int main() {
    Student stu1, stu2;
    cin >> stu1 >> stu2;
    cout << stu1 << stu2;
    return 0;
}
</script></code></pre>


<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:16:33]
$ ./a.out
ZhangSan 16 145.5
WangWu 15 143
name: ZhangSan, age: 16, score: 145.5
name: WangWu, age: 15, score: 143
</script></code></pre>


打印一个九九乘法表：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

int main() {
    for (int i=1; i<=9; i++) {
        for (int j=1; j<=9; j++) {
            if (j <= i) {
                printf("%dx%d=%2d   ", i, j, i*j);
            } else {
                break;
            }
        }
        printf("\n");
    }
    return 0;
}
</script></code></pre>


<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:28:02]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [20:28:04]
$ ./a.out
1x1= 1
2x1= 2   2x2= 4
3x1= 3   3x2= 6   3x3= 9
4x1= 4   4x2= 8   4x3=12   4x4=16
5x1= 5   5x2=10   5x3=15   5x4=20   5x5=25
6x1= 6   6x2=12   6x3=18   6x4=24   6x5=30   6x6=36
7x1= 7   7x2=14   7x3=21   7x4=28   7x5=35   7x6=42   7x7=49
8x1= 8   8x2=16   8x3=24   8x4=32   8x5=40   8x6=48   8x7=56   8x8=64
9x1= 9   9x2=18   9x3=27   9x4=36   9x5=45   9x6=54   9x7=63   9x8=72   9x9=81
</script></code></pre>

## C++相比C改进的细节
C++ 是在C语言的基础上改进的，C语言的很多语法在 C++ 中依然广泛使用，例如：
- C++ 仍然使用 char、short、int、long、float、double 等基本数据类型；
- C++ 仍然使用 if...else、while、for、switch、break 等分支或循环结构；
- C++ 仍然使用 +、-、*、/、% 等运算符；
- C++ 仍然使用 typedef、#define、enum、struct 等；
- C++ 仍然使用C语言中经典的指针（Pointer），并且使用范围有增无减，甚至不可或缺；

C++ 改进的细节：
- `变量定义位置`
ANSI C 规定，所有局部变量都必须定义在函数开头，在定义好变量之前不能有其他的执行语句；
C99 标准取消这这条限制，但是 VC/VS 对 C99 的支持很不积极，仍然要求变量定义在函数开头；
C++ 则取消了这条限制，变量只要在使用之前定义就好了，不需要在开头就全部定义；
- `布尔类型bool`
在C语言中，关系运算和逻辑运算的结果有两种，真和假：0 表示假，非 0 表示真；
C语言并没有彻底从语法上支持“真”和“假”，只是用 1 和 0 来代表；这点在 C++ 中得到了改善，C++ 新增了`bool`类型，它一般占用`1`个字节长度。bool 类型只有两个取值，`true`和`false`：true 表示“真”，false 表示“假”；
`true`、`false`是C++中的两个关键字，用`%d`输出它们，依旧是`1`、`0`数值；
- `空指针nullptr`
`nullptr`是 C++11 标准用来表示`空指针的常量值`；
在C语言中，空指针的值表示为`#define NULL (void *)0`；
在C++中，由于对语法的类型检查更为严格，因而空指针的值就不能表示为`(void *)0`；
所以至少自 C++98 开始`#define NULL 0`；但这会在函数重载时遇到新的困难，所以加入了`nullptr`来表示空指针；
- `函数参数void`
对于C语言来说，如果一个函数不接受任何参数，那么应该将其形参定义为`void`，这样在编译期间就会检查相应的错误；
但是对于C++来说，这个`void`可有可无，没有`void`作为形参，如`void func();`那么也不能传递任何参数，当然有`void`也是这样；
- `函数占位参数`
占位参数只有形参的类型声明，没有参数名，如`void func(int x, int y, int = 0);`在函数内部也无法使用该参数；

## C++中const的变化
在C语言中，const用来限制一个变量，表示这个变量不能被修改，我们通常称这样的变量为常量；
在C++中，const的含义并没有改变，只是对细节进行了一些调整，以下是最主要的两点：

**C++中的 const 更像编译阶段的 #define**
先来看下面的两条语句：
`const int m = 10;`
`int n = m;`

我们知道，变量是要占用内存的，即使被 const 修饰也不例外。m、n 两个变量占用不同的内存，int n = m;表示将 m 的值赋给 n，这个赋值的过程在C和C++中是有区别的：
在C语言中，编译器会先到 m 所在的内存取出一份数据，再将这份数据赋给 n；
而在C++中，编译器会直接将 10 赋给 n，没有读取内存的过程，和int n = 10;的效果一样；
C++ 中的常量更类似于#define命令，是一个值替换的过程，只不过#define是在预处理阶段替换，而常量是在编译阶段替换；

C++ 对 const 的处理少了读取内存的过程，优点是提高了程序执行效率，缺点是不能反映内存的变化，一旦 const 变量被修改，C++ 就不能取得最新的值，这很好理解，因为 C++ 的 const 基本上可以等同于 #define 宏替换；

上面这两条语句基本等同于c语言中的：
`#define m 10`
`int n = m;`

实际上，对于 const 修饰的常量，并不意味着真的就不能修改它的值了，只不过是在语义层面上不能修改而已：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <stdio.h>

int main() {
    const int a = 10;
    int *p = (int *)&a;
    *p = 20;
    printf("%d\n", a);
    return 0;
}
</script></code></pre>

以C语言方式编译（gcc）：
<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:18:16]
$ gcc a.c

# root @ arch in ~/work on git:master x [9:19:22]
$ ./a.out
20
</script></code></pre>

以C++方式编译（g++）：
<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:19:23]
$ g++ a.c

# root @ arch in ~/work on git:master x [9:19:57]
$ ./a.out
10
</script></code></pre>


为什么结果会不一样？
前面说了，在C++中，const常量基本可以看作是一个宏定义，在汇编阶段之前，也就是在编译的时候，a就被替换为了10；
那么如何才能看到修改后的结果呢？只需要把printf中的 a 换成 *p 就行了：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <stdio.h>

int main() {
    const int a = 10;
    int *p = (int *)&a;
    *p = 20;
    printf("%d\n", *p);
    return 0;
}
</script></code></pre>

以C++方式编译：
<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:23:36]
$ g++ a.c

# root @ arch in ~/work on git:master x [9:24:03]
$ ./a.out
20
</script></code></pre>


**C++中全局 const 变量的可见范围是当前文件**
在C/C++中，**普通全局变量的作用域是当前文件，可见范围是整个工程中的所有文件**，使用extern声明后就可以使用；

在C语言中，const 变量和普通变量一样，在其他源文件中也是可见的；const 变量在多文件编程时的表现和普通变量一样，除了不能修改，没有其他区别；

但是 C++ 对 const 的特性做了调整，C++ 规定，**全局 const 变量的作用域仍然是当前文件，可见范围也是当前文件，在其他文件中不可见**，这和添加了static关键字的效果类似；

由于 C++ 中全局 const 变量的可见范围仅限于当前源文件，所以可以将它放在头文件中，这样即使头文件被包含多次也不会出错；

C和C++中全局 const 变量的作用域相同，都是当前文件，不同的是它们的可见范围：
C语言中 const 全局变量的可见范围是整个程序，在其他文件中使用 extern 声明后就可以使用；
而C++中 const 全局变量的可见范围仅限于当前文件，在其他文件中不可见，所以它可以定义在头文件中，多次引入后也不会出错；

如果你使用的是 GCC，那么可以通过添加 extern 关键字来增大 C++ 全局 const 变量的可见范围，如：`extern const int n = 10;`
这样 n 的可见范围就变成了整个程序，在其他文件中使用 extern 声明后就可以使用了；不过这种方式只适用于 GCC，不适用于 VS；

**从这两点来看，C++似乎想用 const 基本替代 #define 宏的作用，官方的态度也是不建议大量的使用 #define 宏**！

## inline内联函数
我们知道，函数调用是有时间和空间开销的。程序在执行一个函数之前需要做一些准备工作，要将实参、局部变量、返回地址以及若干寄存器都压入栈中，然后才能执行函数体中的代码；函数体中的代码执行完毕后还要清理现场，将之前压入栈中的数据都出栈，才能接着执行函数调用位置以后的代码；

如果函数体代码比较多，需要较长的执行时间，那么函数调用机制占用的时间可以忽略；如果函数只有一两条语句，那么大部分的时间都会花费在函数调用机制上，这种时间开销就就不容忽视。

为了消除函数调用的时空开销，C++提供一种提高效率的方法，即在编译时将函数调用处用函数体替换，类似于C语言中的宏展开。这种在函数调用处直接嵌入函数体的函数称为内联函数（Inline Function），又称内嵌函数或者内置函数；

指定内联函数的方法很简单，只需要在**函数定义处增加`inline`关键字进行修饰**；

当函数比较复杂时，函数调用的时空开销可以忽略，大部分的CPU时间都会花费在执行函数体代码上，所以我们**一般是将非常短小的函数声明为内联函数**；

由于内联函数比较短小，我们**通常的做法是省略函数原型，将整个函数定义（包括函数头和函数体）放在本应该提供函数原型的地方**；

使用内联函数的缺点也是非常明显的，编译后的程序会存在多份相同的函数拷贝，如果被声明为内联函数的函数体非常大，那么编译后的程序体积也将会变得很大，所以再次强调，一般只将那些短小的、频繁调用的函数声明为内联函数；

最后需要说明的是，对函数作 inline 声明只是程序员对编译器提出的一个建议，而不是强制性的，并非一经指定为 inline 编译器就必须这样做。编译器有自己的判断能力，它会根据具体情况决定是否这样做；

**inline内联函数、#define宏展开**
C++中，`内联函数`和前面所说的`const常量`很相似，都可以理解为`编译阶段的宏`；
`const常量`：基本等同于C中的宏定义，如`#define NULL ((void *)0)`；
`inline内联函数`：基本等同于C中的带参数的宏定义，如`#define AND(x, y) ((x) + (y))`；

这更加肯定了前面的观点：C++试图通过`const常量`、`inline内联函数`来完全替代C中的`#define`；

所以在编写C++代码时推荐使用内联函数来替换带参数的宏；
和宏一样，内联函数可以定义在头文件中（不用加 static 关键字），并且头文件被多次#include后也不会引发重复定义错误；
这一点和非内联函数不同，非内联函数是禁止定义在头文件中的，它所在的头文件被多次#include后会引发重复定义错误；

内联函数在编译时会将函数调用处用函数体替换，编译完成后函数就不存在了，所以在链接时不会引发重复定义错误；

**使用内联函数需要注意的地方**
如果你熟悉C中的模块化多文件编程，那么我们通常会将函数的声明放在头文件中，而将函数的定义（实现）放在源文件中；
在C++中，这条规则依旧适用，不过对于内联函数来说，我们一般都是将内联函数的声明和定义都放在头文件中，而不是分开存放；

并且，内联函数的本意是替代预处理期间的宏，编译器在编译期间需要知道函数体的全部代码，才能进行类似宏一样的展开操作；
所以，一般不建议将内联函数分为声明和定义两个部分，这个完全是吃力不讨好，违反了其本意；应直接定义整个函数，不要出现声明；

`inline关键字只在函数定义处添加`，在函数声明处添加 inline 关键字是无效的，编译器会忽略函数声明处的 inline 关键字；
也就是说，**inline是一种`用于实现的关键字`，而不是一种`用于声明的关键字`**；

> 
内联函数虽然叫做函数，在定义和声明的语法上也和普通函数一样，但它已经失去了函数的本质。函数是一段可以重复使用的代码，它位于虚拟地址空间中的代码区，也占用可执行文件的体积，而内联函数的代码在编译后就被消除了，不存在于虚拟地址空间中，没法重复使用；

## 函数的默认参数
在C++中，定义函数时可以给形参指定一个默认的值，这样调用函数时如果没有给这个形参赋值（没有对应的实参），那么就使用这个默认的值；
所谓默认参数，指的是当函数调用中省略了实参时自动使用的一个值，这个值就是给形参指定的默认值；

**C++规定，默认参数只能放在形参列表的最后，而且一旦为某个形参指定了默认值，那么它后面的所有形参都必须有默认值**；

如：`void func(int n = 20);`、`void func(int x, int y = 10, int z = 30);`

还有一点要注意，**C++规定，在同一个作用域中只能指定一次默认参数**
C/C++有这四种作用域：`函数原型作用域`、`局部作用域（函数作用域）`、`块作用域`、`文件作用域（全局作用域）`；

所以一般的做法是在头文件的函数声明中指定默认参数，在源文件中不要指定默认参数，一般就没什么问题了；

注意，这种方式的函数声明也是正确的，不过不是很推荐，有点多此一举的感觉：
`void func(int a = 10, int b);`
`void func(int a, int b = 20);`

## 函数重载overload
在实际开发中，有时候我们需要实现几个功能类似的函数，只是有些细节不同；
例如希望交换两个变量的值，这两个变量有多种类型，可以是 int、float、char、bool 等，我们需要通过参数把变量的地址传入函数内部；
在C语言中，程序员往往需要分别设计出三个不同名的函数，如：
`void swap1(int *a, int *b);`：交换两个 int 类型的变量；
`void swap2(float *a, float *b);`：交换两个 float 类型的变量；
`void swap3(char *a, char *b);`：交换两个 char 类型的变量；
`void swap4(bool *a, bool *b);`：交换两个 bool 类型的变量；

但在C++中，这完全没有必要，C++ 允许多个函数拥有相同的名字，只要它们的参数列表不同就可以，这就是`函数的重载（Function Overloading）`；借助重载，一个函数名可以有多种用途；

**`参数列表`又叫`参数签名`，包括`参数的类型`、`参数的个数`和`参数的顺序`，只要有一个不同就叫做参数列表不同**

所以，我们可以将上面的几个命名毫无意义的函数都这样写：
`void swap(int *a, int *b);`：交换两个 int 类型的变量；
`void swap(float *a, float *b);`：交换两个 float 类型的变量；
`void swap(char *a, char *b);`：交换两个 char 类型的变量；
`void swap(bool *a, bool *b);`：交换两个 bool 类型的变量；

这四个函数之间互相构成了重载关系，在调用的时候，C++会逐个的去匹配合适的函数，进而调用对应的swap交换函数；

注意，这里仅仅是演示C++中的函数重载，在C++标准库中，已经有了swap系列函数，并且名字就叫做`swap`；

重载就是在一个作用范围内（同一个类、同一个命名空间等）有多个名称相同但参数不同的函数；
重载的结果是让一个函数名拥有了多种用途，使得命名更加方便，调用更加灵活；

**注意，参数列表不同包括参数的个数不同、类型不同或顺序不同，仅仅参数名称不同是不可以的，函数返回值也不能作为重载的依据**

**构成重载的规则、要素**
- 函数名称必须相同
- 参数列表必须不同（个数不同、类型不同、参数排列顺序不同）
- 函数的返回类型可以相同也可以不相同
- 仅仅返回类型不同不足以成为函数的重载

**C++是如何做到函数重载的**
C++代码在编译时会根据参数列表对函数进行重命名，例如`void swap(int a, int b)`会被重命名为`_swap_int_int`，`void swap(float a, float b)`会被重命名为`_swap_float_float`；
当发生函数调用时，编译器会根据传入的实参去逐个匹配，以选择对应的函数，如果匹配失败，编译器就会报错，这叫做`重载决议（Overload Resolution）`；

不同的编译器有不同的重命名方式，这里仅仅举例说明，实际情况可能并非如此；

> 
从这个角度讲，函数重载仅仅是语法层面的，本质上它们还是不同的函数，占用不同的内存，入口地址也不一样；

**函数重载过程中的二义性及类型转换**
上节我们讲到，发生函数调用时编译器会根据传入的实参的个数、类型、顺序等信息去匹配要调用的函数，这在大部分情况下都能够精确匹配；
但当实参的类型和形参的类型不一致时情况就会变得稍微复杂，例如函数形参的类型是int，调用函数时却将short类型的数据交给了它，编译器就需要先将short类型转换为int类型才能匹配成功；

C++ 标准规定，在进行重载决议时编译器应该按照下面的优先级顺序来处理实参的类型：
- **精确匹配**
 - 不做类型转换，`直接匹配`
 - 只是做`微不足道的转换`，如：从数组名到数组指针、从函数名到指向函数的指针、从非 const 类型到 const 类型；
- **类型提升后匹配**
 - `整型提升`，如：从 bool、char、short 提升为 int，或者从 char16_t、char32_t、wchar_t 提升为 int、long、long long；
 - `小数提升`，如：从 float 提升为 double；
- 使用**自动类型转换后匹配**
 - `整型转换`，如：从 char 到 long、short 到 long、int 到 short、long 到 char；
 - `小数转换`，如：从 double 到 float；
 - `整数和小数转换`，如：从 int 到 double、short 到 float、float 到 int、double 到 long；
 - `指针转换`，如：从`int *`到`void *`；

C++标准还规定，编译器应该按照**从高到低的顺序**来搜索重载函数：首先是`精确匹配`，然后是`类型提升`，最后才是`类型转换`；
**一旦在某个优先级中找到唯一的一个重载函数就匹配成功，不再继续往下搜索**

如果在一个优先级中找到多个（两个以及以上）合适的重载函数，编译器就会陷入两难境地，不知道如何抉择，编译器会将这种模棱两可的函数调用视为一种错误，因为这些合适的重载函数同等“优秀”，没有一个脱颖而出，调用谁都一样。这就是函数重载过程中的`二义性错误`；

注意，类型提升和类型转换不是一码事！类型提升是积极的，是为了更加高效地利用计算机硬件，不会导致数据丢失或精度降低；而类型转换是不得已而为之，不能保证数据的正确性，也不能保证应有的精度。类型提升只有上表中列出的几种情况，其他情况都是类型转换；

当重载函数有多个参数时也会产生二义性，而且情况更加复杂；
C++ 标准规定，如果有且只有一个函数满足下列条件，则匹配成功：
- 该函数对每个实参的匹配都不劣于其他函数；
- 至少有一个实参的匹配优于其他函数；

**总结**
在设计重载函数时，参数类型过少或者过多都容易引起二义性错误，因为这些类型相近，彼此之间会相互转换；

## new、delete操作符
在C语言中，动态`分配内存`用`malloc()`函数，`释放内存`用`free()`函数；
在C++中，这两个函数仍然可以使用，但是C++又新增了两个关键字，`new`和`delete`：new用来动态分配内存，delete用来释放内存；

用 `malloc/free` 分配、释放内存：
`int *p = (int *)malloc(sizeof(int) * 10);`：分配10个int型的内存空间；
`free(p);`：释放内存；

用 `new/delete` 分配、释放内存：
`int *p = new int;`：分配1个int型的内存空间；
`delete p;`：释放内存；

用 `new[]/delete[]` 分配、释放一组连续的数据：
`int *p = new int[10];`：分配10个int型的内存空间；
`delete[] p;`：释放内存；

和 malloc() 一样，new 也是在堆区分配内存，必须手动释放，否则只能等到程序运行结束由操作系统回收；
为了避免内存泄露，通常 new 和 delete、new[] 和 delete[] 操作符应该成对出现，并且不要和C语言中 malloc()、free() 一起混用；

在C++中，建议使用 new 和 delete 来管理内存，它们可以使用C++的一些新特性，最明显的是可以自动调用构造函数和析构函数；
