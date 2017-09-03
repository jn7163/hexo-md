---
title: C++ 模板
date: 2017-08-28 09:08:00
categories:
- cpp
tags:
- cpp
keywords: C++ 模板 函数模板 类模板 泛型编程
---

> 
C++ 模板，`模板（Template）`是 C++ 中的一种**参数化类型机制**，**`模板`是 C++ 的`泛型编程`中不可缺少的一部分**；
模板是 C++ 程序员绝佳的武器，特别是结合了`多重继承`与`运算符重载`之后；C++ 的标准函数库提供的许多有用的函数大多结合了模板的概念，如`STL`以及`iostream`；

<!-- more -->

## 函数模板
为了交换不同类型的变量的值，我们可以通过函数重载，定义几个仅仅数据类型不同的相同名称的函数，来达到此目的：
<pre><code class="language-cpp line-numbers"><script type="text/plain">void swap(int *a, int *b) {
    int tmp = *a;
    *a = *b;
    *b = tmp;
}

void swap(float *a, float *b) {
    float tmp = *a;
    *a = *b;
    *b = tmp;
}

void swap(char *a, char *b) {
    char tmp = *a;
    *a = *b;
    *b = tmp;
}

void swap(bool *a, bool *b) {
    bool tmp = *a;
    *a = *b;
    *b = tmp;
}
</script></code></pre>


这些函数虽然在调用时方便了一些，但从本质上说还是定义了三个功能相同、函数体相同的函数，只是数据的类型不同而已，这看起来有点浪费代码，能不能把它们压缩成一个函数呢？

我们知道，数据的值可以通过函数参数传递，在函数定义时数据的值是未知的，只有等到函数调用时接收了实参才能确定其值；这就是**值的参数化**；

在C++中，数据的类型也可以通过参数来传递，在函数定义时可以不指明具体的数据类型，当发生函数调用时，编译器可以根据传入的实参自动推断数据类型；这就是**类型的参数化**；

`类型（Type）`和`值（Value）`是数据的两个主要特征，它们在C++中都可以被参数化；

所谓函数模板，实际上是建立一个通用函数，它所用到的数据的类型（包括返回值类型、形参类型、局部变量类型）可以不具体指定，而是用一个虚拟的类型来代替（实际上是用一个标识符来占位），等发生函数调用时再根据传入的实参来逆推出真正的类型；这个通用函数就称为`函数模板（Function Template）`；

在函数模板中，数据的值和类型都被参数化了，发生函数调用时编译器会根据传入的实参来推演形参的值和类型；换个角度说，函数模板除了支持值的参数化，还支持类型的参数化；

一但定义了函数模板，就可以将类型参数用于函数定义和函数声明了；说得直白一点，原来使用 int、float、char 等内置类型的地方，都可以用类型参数来代替；

下面我们就来实践一下，将上面的`swap()`函数压缩为一个函数模板：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

template <typename T>
void swap(T *a, T *b) {
    T tmp = *a;
    *a = *b;
    *b = tmp;
}

int main() {
    int a = 10, b = 20;
    swap<int>(&a, &b);  // 显示的指明类型参数 T 的具体类型为 int
    printf("%d, %d\n", a, b);
    swap(&a, &b);   // 不指明类型参数 T 的具体类型，编译器会根据传入的实参类型进行逆推
    printf("%d, %d\n", a, b);
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:04:35] C:1
$ g++ a.cpp

# root @ arch in ~/work on git:master x [10:04:37]
$ ./a.out
20, 10
10, 20
</script></code></pre>


`template <typename T>`被称为**模板头**，`T`为**类型参数**，T 可用于函数的返回值，形参列表、函数体内部，就和使用 int、float、bool 等基本类型一样；

**类型参数**的命名规则跟其他标识符的命名规则一样，不过使用 T、T1、T2、Type 等已经成为了一种惯例；

类型参数可以有多个，它们之间以`逗号,`分隔；类型参数列表以`<>`包围，形式参数列表以`()`包围；

> 
`typename`关键字也可以使用`class`关键字替代，它们没有任何区别；
C++ 早期对模板的支持并不严谨，没有引入新的关键字，而是用`class`来指明类型参数，但是`class`关键字本来已经用在类的定义中了，这样做显得不太友好；
所以后来 C++ 又引入了一个新的关键字`typename`，专门用来定义类型参数；不过至今仍然有很多代码在使用`class`关键字，包括 C++ 标准库、一些开源程序等；

**函数模板也可以提前声明，不过声明时需要带上模板头**

## 类模板
C++ 除了支持`函数模板`，还支持`类模板（Class Template）`；
函数模板中定义的类型参数可以用在函数声明和函数定义中，类模板中定义的类型参数可以用在类声明和类实现中；类模板的目的同样是将数据的类型参数化；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

template <typename T1, typename T2>
class Point {
public:
    Point(T1 x, T2 y);
public:
    T1 getx() const;
    T2 gety() const;
    void setx(T1 x);
    void sety(T2 y);
private:
    T1 m_x;
    T2 m_y;
};

template <typename T1, typename T2>
Point<T1, T2>::Point(T1 x, T2 y) : m_x(x), m_y(y) {}

template <typename T1, typename T2>
T1 Point<T1, T2>::getx() const {
    return m_x;
}

template <typename T1, typename T2>
T2 Point<T1, T2>::gety() const {
    return m_y;
}

template <typename T1, typename T2>
void Point<T1, T2>::setx(T1 x) {
    m_x = x;
}

template <typename T1, typename T2>
void Point<T1, T2>::sety(T2 y) {
    m_y = y;
}

int main() {
//  Point p(1, 2);  // 错误, 编译器不能根据给定的数据推演出类型参数T1、T2
    Point<int, int> p1(1, 2);
    printf("(%d, %d)\n", p1.getx(), p1.gety());

    Point<const char *, const char *> p2("0791", "JiangXi|NanChang");
    printf("(%s, %s)\n", p2.getx(), p2.gety());

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [11:04:43]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [11:04:44]
$ ./a.out
(1, 2)
(0791, JiangXi|NanChang)
</script></code></pre>


## 泛型编程
模板所支持的类型是宽泛的，没有限制的，我们可以使用任意类型来替换，这种编程方式称为`泛型编程（Generic Programming）`；
相应地，可以将参数 T 看做是一个泛型，而将 int、float、string 等看做是一种具体的类型；除了 C++，Java、C#、Pascal（Delphi）也都支持泛型编程；

**C++ 模板也是被迫推出的，最直接的动力来源于对数据结构的封装**；
数据结构关注的是数据的存储，以及存储后如何进行增加、删除、修改和查询操作，它是一门基础性的学科，在实际开发中有着非常广泛的应用；
C++ 开发者们希望为线性表、链表、图、树等常见的数据结构都定义一个类，并把它们加入到标准库中，这样以后程序员就不用重复造轮子了，直接拿来使用即可；

但是这个时候遇到了一个无法解决的问题，就是数据结构中每份数据的类型无法提前预测；
以链表为例，它的每个节点可以用来存储小数、整数、字符串等，也可以用来存储一名学生、教师、司机等，还可以直接存储二进制数据，这些都是可以的，没有任何限制；
而 C++ 又是强类型的，数据的种类受到了严格的限制，这种矛盾是无法调和的；

要想解决这个问题，C++ 必须推陈出新，跳出现有规则的限制，开发新的技术，于是模板就诞生了；
模板虽然不是 C++ 的首创，但是却在 C++ 中大放异彩，后来也被 Java、C# 等其他强类型语言采用；

C++ 模板有着复杂的语法，可不仅仅是前面两节讲到的那么简单，它的话题可以写一本书；
C++ 模板也非常重要，整个标准库几乎都是使用模板来开发的，STL 更是经典之作；

> 
`STL（Standard Template Library，标准模板库）`就是 C++ 对**数据结构**进行封装后的称呼；

## 函数模板的重载
当需要对不同的类型使用同一种算法（同一个函数体）时，为了避免定义多个功能重复的函数，可以使用模板；
然而，并非所有的类型都使用同一种算法，有些特定的类型需要单独处理，为了满足这种需求，C++ 允许对函数模板进行重载，程序员可以像重载常规函数那样重载模板定义；

在前面，我们定义了 Swap() 函数用来交换两个变量的值，一种方案是使用指针，另外一种方案是使用引用；
这两种方案都可以交换 int、float、char、bool 等基本类型变量的值，但是却不能交换两个数组；

因此需要增加一个 Swap() 函数专门交换两个数组，与之前的 Swap() 函数模板构成重载：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

template <typename T>
void Swap(T &a, T &b) {
    T tmp = a;
    a = b;
    b = tmp;
}

template <typename T>
void Swap(T *a, T *b, int len) {
    T tmp;
    for (int i=0; i<len; i++) {
        tmp = a[i];
        a[i] = b[i];
        b[i] = tmp;
    }
}

template <typename T>
void print(T *a, int len) {
    cout << "Array[" << len << "] = { ";
    for (int i=0; i<len; i++) {
        cout << a[i] << ", ";
    }
    cout << "\b\b }" << endl;
}

int main() {
    int a, b;
    cin >> a >> b;
    Swap(a, b);
    cout << a << " " << b << endl;

    int arr1[] = {1, 2, 3, 4, 5};
    int arr2[] = {11, 22, 33, 44, 55};
    print(arr1, sizeof(arr1)/sizeof(int));
    print(arr2, sizeof(arr2)/sizeof(int));

    Swap(arr1, arr2, sizeof(arr1)/sizeof(int));
    print(arr1, sizeof(arr1)/sizeof(int));
    print(arr2, sizeof(arr2)/sizeof(int));

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:07:33]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [12:07:42]
$ ./a.out
1 2
2 1
Array[5] = { 1, 2, 3, 4, 5 }
Array[5] = { 11, 22, 33, 44, 55 }
Array[5] = { 11, 22, 33, 44, 55 }
Array[5] = { 1, 2, 3, 4, 5 }
</script></code></pre>


上面的例子中，我们额外的添加了一个形参`len`，指明数组的长度，不然无法从传入的参数中获取任何长度信息，因为数组作为函数参数时，会隐式的转换为`int *`类型，而不是之前的`int [LEN]`（`LEN`为具体的数组长度）类型；

不过，对于 C++ 的模板来说，上面的例子还可以写的更优美、好看一些：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

template <typename T>
void Swap(T &a, T &b) {
    T tmp = a;
    a = b;
    b = tmp;
}

template <typename T, int len>
void Swap(T (&a)[len], T (&b)[len]) {
    T tmp;
    for (int i=0; i<len; i++) {
        tmp = a[i];
        a[i] = b[i];
        b[i] = tmp;
    }
}

template <typename T, int len>
void print(T (&arr)[len]) {
    cout << "Array[" << len << "] = { ";
    for (int i=0; i<len; i++) {
        cout << arr[i] << ", ";
    }
    cout << "\b\b }" << endl;
}

int main() {
    int a, b;
    cin >> a >> b;
    Swap(a, b);
    cout << a << " " << b << endl;

    int arr1[] = {1, 2, 3, 4, 5};
    int arr2[] = {11, 22, 33, 44, 55};
    print(arr1);
    print(arr2);

    Swap(arr1, arr2);
    print(arr1);
    print(arr2);

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:12:07] C:127
$ g++ b.cpp

# root @ arch in ~/work on git:master x [12:12:09]
$ ./a.out
11 22
22 11
Array[5] = { 1, 2, 3, 4, 5 }
Array[5] = { 11, 22, 33, 44, 55 }
Array[5] = { 11, 22, 33, 44, 55 }
Array[5] = { 1, 2, 3, 4, 5 }
</script></code></pre>


> 这个例子目前不必细究，之后的章节会详细介绍；

## 函数模板的实参推断
在使用`类模板`创建对象时，程序员需要`显式的指明实参的类型`；

例如 Point 类：`template <typename T1, typename T2> class Point;`

我们可以在栈上创建对象，也可以在堆上创建对象：
`Point<int, int> p1(10, 20);`：在栈上创建对象
`Point<const char *, const char *> *p = new Point<const char *, const char *>("东经180度", "北纬210度");`：在堆上创建对象

因为已经显式地指明了 T1、T2 的具体类型，所以编译器就不用再自己推断了，直接拿来使用即可；

而对于函数模板，大多数时候不需要显示的指明实参的类型，编译器可以自动通过传入的实参类型进行推导：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

template <typename T>
void Swap(T &a, T &b) {
    T tmp = a;
    a = b;
    b = tmp;
}

int main() {
    int m = 10, n = 20;
    Swap(m, n);
    return 0;
}
</script></code></pre>


编译器很明显的可以知道 m、n 的数据类型，进而推导出在本次调用中 T 的实际类型为 int；

但是对于这种情况的函数模板，就不能让编译器自动推导了，因为编译器实在是无法知道 T 的实际类型：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

template <typename T>
void func() {
    T data = (T)0;
    cout << data << endl;
}

int main() {
//  func(); // 错误, 编译器无法推断 T 的实际类型
    func<int>();
    func<double>();
    return 0;
}
</script></code></pre>


**模板实参推断**
通过函数实参来确定模板实参（也就是类型参数的具体类型）的过程称为`模板实参推断`；
在模板实参推断过程中，编译器使用函数调用中的实参类型来寻找类型参数的具体类型；

**模板实参推断过程中的类型转换**
对于**普通函数（非模板函数）**，发生函数调用时会对实参的类型进行适当的转换，以适应形参的类型；这些转换包括：
- `算数转换`：例如 int 转换为 float，char 转换为 int，double 转换为 int 等；
- `派生类向基类的转换`：也就是`向上转型`；
- `const 转换`：也即将`非 const 类型`转换为`const 类型`，例如将`char *`转换为`const char *`；
- `数组或函数指针转换`：如果函数形参不是引用类型，那么数组名会转换为数组指针，函数名也会转换为函数指针；
- `用户自定的类型转换`；

而对于**函数模板**，类型转换则受到了更多的限制，仅能进行「`const转换`」和「`数组或函数指针转换`」，其他的都不能应用于函数模板；

> 当函数形参是引用类型时，数组不会转换为指针；

**为函数模板显示指明实参**
「为函数模板显式地指明实参」和「为类模板显式地指明实参」的形式是类似的，就是在函数名后面添加尖括号`<>`，里面包含具体的类型；

显式指明的模板实参会按照从左到右的顺序与对应的模板参数匹配：第一个实参与第一个模板参数匹配，第二个实参与第二个模板参数匹配，以此类推；只有尾部（最右）的类型参数的实参可以省略，而且前提是它们可以从传递给函数的实参中推断出来；

**显式地指明实参时可以应用正常的类型转换**
上面我们提到，函数模板仅能进行「`const转换`」和「`数组或函数指针转换`」两种形式的类型转换，但是当我们显式地指明类型参数的实参（具体类型）时，就可以使用正常的类型转换（非模板函数可以使用的类型转换）了；

## 模板的显式具体化
C++ 没有办法限制类型参数的范围，我们可以使用任意一种类型来实例化模板；但是模板中的语句（函数体或者类体）不一定就能适应所有的类型，可能会有个别的类型没有意义，或者会导致语法错误；

例如有下面的函数模板，它用来获取两个变量中较大的一个：
<pre><code class="language-cpp line-numbers"><script type="text/plain">template <typename T>
const T & Max(const T &a, const T &b) {
    return a > b ? a : b;
}
</script></code></pre>

请读者注意`a > b`这条语句，`>`能够用来比较 int、float、char 等基本类型数据的大小，但是却不能用来比较结构体变量、对象以及数组的大小，因为我们并没有针对结构体、类和数组重载`>`；
另外，该函数模板虽然可以用于指针，但比较的是地址大小，而不是指针指向的数据，所以也没有现实的意义；

除了`>`，`+`、`-`、`*`、`/`、`==`、`<`等运算符也只能用于基本类型，不能用于结构体、类、数组等复杂类型；

总之，编写的函数模板很可能无法处理某些类型，我们必须对这些类型进行单独处理；

模板是一种泛型技术，它能接受的类型是宽泛的、没有限制的，并且对这些类型使用的算法都是一样的（函数体或类体一样）；
但是现在我们希望改变这种“游戏规则”，让模板能够针对某种具体的类型使用不同的算法（函数体或类体不同），这在 C++ 中是可以做到的，这种技术称为`模板的显示具体化（Explicit Specialization）`；

`函数模板`和`类模板`都可以显示具体化，下面我们先讲解函数模板的显示具体化，再讲解类模板的显示具体化；

**函数模板的显示具体化**
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

struct Student {
    const char *name;
    int age;
    float score;
};

template <typename T>
const T & Max(const T &a, const T &b) {
    return a > b ? a : b;
}

template <>
const Student & Max<Student>(const Student &a, const Student &b) {
    return a.score > b.score ? a : b;
}

ostream & operator<<(ostream &out, const Student &s) {
    out << s.name << ", " << s.age << ", " << s.score;
    return out;
}

int main() {
    int a = 10, b = 20;
    cout << Max(a, b) << endl;

    Student s1 = {"zhang3", 15, 112}, s2 = {"li4", 14, 115};
    cout << Max(s1, s2) << endl;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:45:58]
$ g++ c.cpp

# root @ arch in ~/work on git:master x [13:46:19]
$ ./a.out
20
li4, 14, 115
</script></code></pre>


`Max<Student>`中的 Student 是可选的，因为函数的形参已经表明，这是 Student 类型的一个具体化，编译器能够逆推出 T 的具体类型；

**函数的调用规则**
在 C++ 中，对于给定的函数名，可以有`非模板函数`、`模板函数`、`显示具体化模板函数`以及它们的`重载版本`；
在调用函数时，它们的优先级从高到低分别为：`非模板函数` > `显示具体化模板函数` > `常规模板函数`；

**类模板的显示具体化**
除了函数模板，类模板也可以显示具体化，并且它们的语法是类似的；

在前面一节中我们定义了一个 Point 类，用来输出不同类型的坐标；
在输出结果中，横坐标 x 和纵坐标 y 是以`逗号,`为分隔的，但是由于个人审美的不同，我希望当 x 和 y 都是字符串时以`|`为分隔，是数字或者其中一个是数字时才以`逗号,`为分隔；
为了满足我这种奇葩的要求，可以使用显示具体化技术对字符串类型的坐标做特殊处理：
<pre><code class="language-cpp line-numbers"><script type="text/plain">// 类模板
template <typename T1, typename T2>
class Point {
public:
    Point(T1 x, T2 y) : m_x(x), m_y(y) {}
public:
    T1 getx() const { return m_x; }
    T2 gety() const { return m_y; }
    void setx(T1 x) { m_x = x; }
    void sety(T2 y) { m_y = y; }
    void print() const;
private:
    T1 m_x;
    T2 m_y;
};

template <typename T1, typename T2>
void Point<T1, T2>::print() const {
    cout << "x=" << m_x << ", y=" << m_y << endl;
}

// 显示具体化
template <>
class Point<const char *, const char *> {
public:
    Point(const char *x, const char *y) : m_x(x), m_y(y) {}
public:
    const char * getx() const { return m_x; }
    const char * gety() const { return m_y; }
    void setx(const char *x) { m_x = x; }
    void sety(const char *y) { m_y = y; }
    void print() const;
private:
    const char *m_x;
    const char *m_y;
};

// 注意这里不带模板头
void Point<const char *, const char *>::print() const {
    cout << "x=" << m_x << " | y=" << m_y << endl;
}
</script></code></pre>


**部分显式具体化**
在上面的显式具体化例子中，我们为所有的类型参数都提供了实参，所以最后的模板头为空，也即`template<>`；
另外 C++ 还允许只为一部分类型参数提供实参，这称为`部分显式具体化`；

**部分显式具体化只能用于`类模板`，不能用于函数模板**

仍然以 Point 为例，假设我现在希望“只要横坐标 x 是字符串类型”就以`|`来分隔输出结果，而不管纵坐标 y 是什么类型，这种要求就可以使用部分显式具体化技术来满足；请看下面的代码：
<pre><code class="language-cpp line-numbers"><script type="text/plain">// 类模板
template <typename T1, typename T2>
class Point {
public:
    Point(T1 x, T2 y) : m_x(x), m_y(y) {}
public:
    T1 getx() const { return m_x; }
    T2 gety() const { return m_y; }
    void setx(T1 x) { m_x = x; }
    void sety(T2 y) { m_y = y; }
    void print() const;
private:
    T1 m_x;
    T2 m_y;
};

template <typename T1, typename T2>
void Point<T1, T2>::print() const {
    cout << "x=" << m_x << ", y=" << m_y << endl;
}

// 部分显式具体化
template <typename T2>
class Point<const char *, T2> {
public:
    Point(const char *x, T2 y) : m_x(x), m_y(y) {}
public:
    const char * getx() const { return m_x; }
    T2 gety() const { return m_y; }
    void setx(const char *x) { m_x = x; }
    void sety(T2 y) { m_y = y; }
    void print() const;
private:
    const char *m_x;
    T2 m_y;
};

// 注意这里需要带上模板头
template <typename T2>
void Point<const char *, T2>::print() const {
    cout << "x=" << m_x << " | y=" << m_y << endl;
}
</script></code></pre>


模板头`template <typename T2>`中声明的是没有被具体化的类型参数；

## 模板中的非类型参数
模板是一种泛型技术，目的是将数据的类型参数化，以增强 C++ 语言（强类型语言）的灵活性；
C++ 对模板的支持非常自由，模板中除了可以包含类型参数，还可以包含非类型参数，例如：
`template <typename T, int N> class Demo {};`
`template <class T, int N> void func(T (&arr)[N]);`
`T`是一个`类型参数`，它通过 class 或 typename 关键字指定；
`N`是一个`非类型参数`，用来传递数据的值，而不是类型，它和`普通函数的形参一样`，都需要指明具体的类型；
`类型参数`和`非类型参数`都可以用在**函数体**或者**类体**中；

当调用一个函数模板或者通过一个类模板创建对象时，非类型参数会被用户提供的、或者编译器推断出的值所取代；

**在函数模板中使用非类型参数**
在前面一节中，我们通过 Swap() 函数来交换两个数组的值，其原型为：`template <typename T> void Swap(T a[], T b[], int len);`
形参 len 用来指明要交换的数组的长度，调用 Swap() 函数之前必须先通过 sizeof 求得数组长度再传递给它；

多出来的形参 len 给编码带来了不便，我们可以借助模板中的非类型参数将它消除，请看下面的代码：
<pre><code class="language-cpp line-numbers"><script type="text/plain">template <typename T, int len>
void Swap(T (&a)[len], T (&b)[len]) {
    T tmp;
    for (int i=0; i<len; i++) {
        tmp = a[i];
        a[i] = b[i];
        b[i] = tmp;
    }
}
</script></code></pre>


`T (&a)[len]`表明 a 是一个引用，它引用的数据的类型是`T [len]`，也即一个数组；`T (&b)[len]`也是类似的道理；
分析一个引用和分析一个指针的方法类似，编译器总是从它的名字开始读取，然后按照优先级顺序依次解析；

调用 Swap() 函数时，需要将数组名字传递给它：
编译器会使用数组类型 int 来代替类型参数 T，使用数组长度来代替非类型参数 len；

下面是一个完整的例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

template <typename T>
void Swap(T &a, T &b) {
    T tmp = a;
    a = b;
    b = tmp;
}

template <typename T, int len>
void Swap(T (&a)[len], T (&b)[len]) {
    T tmp;
    for (int i=0; i<len; i++) {
        tmp = a[i];
        a[i] = b[i];
        b[i] = tmp;
    }
}

template <typename T, int len>
void print(T (&arr)[len]) {
    cout << "Array[" << len << "] = { ";
    for (int i=0; i<len; i++) {
        cout << arr[i] << ", ";
    }
    cout << "\b\b }" << endl;
}

int main() {
    int a, b;
    cin >> a >> b;
    Swap(a, b);
    cout << a << " " << b << endl;

    int arr1[] = {1, 2, 3, 4, 5};
    int arr2[] = {11, 22, 33, 44, 55};
    print(arr1);
    print(arr2);

    Swap(arr1, arr2);
    print(arr1);
    print(arr2);

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:40:07]
$ g++ b.cpp

# root @ arch in ~/work on git:master x [14:40:18]
$ ./a.out
1 2
2 1
Array[5] = { 1, 2, 3, 4, 5 }
Array[5] = { 11, 22, 33, 44, 55 }
Array[5] = { 11, 22, 33, 44, 55 }
Array[5] = { 1, 2, 3, 4, 5 }
</script></code></pre>


**在类模板中使用非类型参数**
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <string>

using namespace std;

template <typename T, int LEN>
class Array {
public:
    Array();
    ~Array();
public:
    int length() const { return m_len; }
    T & operator[](int i) const { return m_ptr[i]; }
    template <typename XT, int XLEN>
    friend istream & operator>>(istream &in, Array<XT, XLEN> &arr);
    template <typename XT, int XLEN>
    friend ostream & operator<<(ostream &out, const Array<XT, XLEN> &arr);
private:
    T *m_ptr;
    int m_len;
};

template <typename T, int LEN>
Array<T, LEN>::Array() : m_len(LEN) {
    m_ptr = new T[m_len];
}

template <typename T, int LEN>
Array<T, LEN>::~Array() {
    delete[] m_ptr;
}

template <typename T, int LEN>
istream & operator>>(istream &in, Array<T, LEN> &arr) {
    for (int i=0; i<arr.m_len; i++) {
        in >> arr.m_ptr[i];
    }
    return in;
}

template <typename T, int LEN>
ostream & operator<<(ostream &out, const Array<T, LEN> &arr) {
    if (arr.m_len == 0) {
        out << "Array is empty" << endl;
    } else {
        out << "Array[" << arr.m_len << "] = { ";
        for (int i=0; i<arr.m_len; i++) {
            out << arr.m_ptr[i] << ", ";
        }
        out << "\b\b }" << endl;
    }
    return out;
}

int main() {
    Array<string, 5> arr;
    cin >> arr;
    cout << arr;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:45:14]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [10:45:17]
$ ./a.out
www.baidu.com www.google.com www.facebook.com www.youtube.com www.zfl9.com
Array[5] = { www.baidu.com, www.google.com, www.facebook.com, www.youtube.com, www.zfl9.com }
</script></code></pre>


**非类型参数的限制**
非类型参数的类型不能随意指定，它受到了严格的限制，只能是一个`整数`，或者是一个指向`对象`或`函数`的`指针`（也可以是`引用`）：

1) 当非类型参数是一个**整数**时，传递给它的实参，或者由编译器推导出的实参必须是一个**常量表达式**，例如10、2 * 30等，但不能是n、n + 10、n + m等（n 和 m 都是变量）；

2) 当非类型参数是一个`指针（引用）`时，绑定到该指针的实参必须具有`静态的生存期`；换句话说，实参必须存储在虚拟地址空间中的`静态数据区`；局部变量位于栈区，动态创建的对象位于堆区，它们都不能用作实参；

## 模板的实例化
`模板（Template）`并不是真正的函数或类，它仅仅是**编译器用来生成函数或类的一张“图纸”**；模板不会占用内存，最终生成的函数或者类才会占用内存；

由模板生成函数或类的过程叫做`模板的实例化（Instantiate）`，相应地，针对某个类型生成的特定版本的函数或类叫做模板的一个`实例（Instantiation）`；

在学习模板以前，如果想针对不同的类型使用相同的算法，就必须定义多个极其相似的函数或类，这样不但做了很多重复性的工作，还导致代码维护困难，用于交换两个变量的值的 Swap() 函数就是一个典型的代表；

而有了模板后，这些工作都可以交给编译器了，编译器会帮助我们自动地生成这些代码；从这个角度理解，模板也可以看做是编译器的一组指令，它命令编译器生成我们想要的代码；

模板的实例化是按需进行的，用到哪个类型就生成针对哪个类型的函数或类，不会提前生成过多的代码；也就是说，编译器会根据传递给类型参数的实参（也可以是编译器自己推演出来的实参）来生成一个特定版本的函数或类，并且相同的类型只生成一次；实例化的过程也很简单，就是将所有的类型参数用实参代替；

例如，给定下面的函数模板及函数调用：
<pre><code class="language-cpp line-numbers"><script type="text/plain">template <typename T>
void Swap(T &a, T &b) {
    T temp = a;
    a = b;
    b = temp;
}

int main() {
    int n1 = 100, n2 = 200, n3 = 300, n4 = 400;
    float f1 = 12.5, f2 = 56.93;

    Swap(n1, n2);  //T为int，实例化出 void Swap(int &a, int &b);
    Swap(f1, f2);  //T为float，实例化出 void Swap(float &a, float &b);
    Swap(n3, n4);  //T为int，调用刚才生成的 void Swap(int &a, int &b);

    return 0;
}
</script></code></pre>


编译器会根据不同的实参实例化出不同版本的 Swap() 函数；

另外需要注意的是**类模板的实例化**，通过类模板创建对象时并不会实例化所有的成员函数，只有等到真正调用它们时才会被实例化；
如果一个成员函数永远不会被调用，那它就永远不会被实例化；这说明类的实例化是延迟的、局部的，编译器并不着急生成所有的代码；

通过类模板创建对象时，一般只需要实例化成员变量和构造函数；成员变量被实例化后就能够知道对象的大小了（占用的字节数），构造函数被实例化后就能够知道如何初始化了；对象的创建过程就是分配一块大小已知的内存，并对这块内存进行初始化；

## 模板和多文件编程
在将函数应用于多文件编程时，我们通常是将`函数定义`放在`源文件`（.cpp文件）中，将`函数声明`放在`头文件`（.h文件）中，使用函数时引入（`#include`命令）对应的头文件即可；

编译是针对单个源文件的，只要有函数声明，编译器就能知道函数调用是否正确；而将函数调用和函数定义对应起来的过程，可以延迟到链接时期；正是有了链接器的存在，函数声明和函数定义的分离才得以实现；

将类应用于多文件编程也是类似的道理，我们可以将类的声明和类的实现分别放到头文件和源文件中；类的声明已经包含了所有成员变量的定义和所有成员函数的声明（也可以是 inline 形式的定义），这样就知道如何创建对象了，也知道如何调用成员函数了，只是还不能将函数调用与函数实现对应起来，但是这又有什么关系呢，反正链接器可以帮助我们完成这项工作；

总起来说，不管是函数还是类，声明和定义（实现）的分离其实是一回事，都是将函数定义放到其他文件中，最终要解决的问题也只有一个，就是把函数调用和函数定义对应起来（找到函数定义的地址，并填充到函数调用处），而保证完成这项工作的就是链接器；

基于传统的编程思维，初学者往往也会将`模板（函数模板和类模板）`的`声明`和`定义`分散到不同的文件中，以期达到「模块化编程」的目的；
但事实证明这种做法是不对的，程序员惯用的做法是**将模板的声明和定义都放到头文件中**；

模板并不是真正的函数或类，它仅仅是用来生成函数或类的一张“图纸”，在这个生成过程中有三点需要明确：
- 模板的实例化是按需进行的，用到哪个类型就生成针对哪个类型的函数或类，不会提前生成过多的代码；
- 模板的实例化是由`编译器`完成的，而不是由`链接器`完成的；
- 在实例化过程中需要知道模板的所有细节，包含声明和定义；

其实这个问题很好理解，模板是用来生成目标 C++ 源代码的，是给 C++ 编译器看的，编译器需要知道模板的全部细节，不然不能进行模板的实例化；

还记得刚开始的一章中的 inline 函数吗，inline 可以理解为编译阶段的带参宏，在编译阶段完成后，已经不存在整个函数了，因为被展开了；

而现在的模板则是更高级一点的宏：函数模板展开后是一个按需生成的函数，是一个完整的函数，和普通的函数没有区别，只不过是编译器生成的，而一般的函数都是我们自己编写的；类模板也是一样的道理；

**总结**
「不能将模板的声明和定义分散到多个文件中」的根本原因是：模板的实例化是由编译器完成的，而不是由链接器完成的，这可能会导致在链接期间找不到对应的实例；

## 模板的显式实例化
前面讲到的模板的实例化是在调用函数或者创建对象时由编译器自动完成的，不需要程序员引导，因此称为`隐式实例化`；
相对应的，我们也可以通过代码明确地告诉编译器需要针对哪个类型进行实例化，这称为`显式实例化`；

编译器在实例化的过程中需要知道模板的所有细节：对于函数模板，也就是函数定义；对于类模板，需要同时知道类声明和类定义；

我们必须将显式实例化的代码放在包含了模板定义的源文件中，而不是仅仅包含了模板声明的头文件中；

显式实例化的一个好处是，**可以将模板的声明和定义（实现）分散到不同的文件中了**；

**函数模板的显式实例化**
`Swap.h`
<pre><code class="language-cpp line-numbers"><script type="text/plain">#ifndef _SWAP_H
#define _SWAP_H

// 声明函数模板
template <typename T> void Swap(T &a, T &b);

// 声明模板显示实例化
extern template void Swap<int>(int &a, int &b);
extern template void Swap<float>(float &a, float &b);

#endif
</script></code></pre>

`Swap.cpp`
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include "Swap.h"

// 定义函数模板
template <typename T>
void Swap(T &a, T &b) {
    T tmp = a;
    a = b;
    b = tmp;
}

// 定义模板显示实例化
template void Swap<int>(int &a, int &b);
template void Swap<float>(float &a, float &b);
</script></code></pre>

`main.cpp`
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>
#include "Swap.h"

int main() {
    int m = 10, n = 20;
    Swap(m, n);
    printf("%d, %d\n", m, n);

    float f1 = 10, f2 = 20;
    Swap(f1, f2);
    printf("%g, %g\n", f1, f2);

    double d1 = 10, d2 = 20;
    Swap(d1, d2);   // 链接时报错，无法找到 double 版本的 Swap() 函数
    printf("%g, %g\n", d1, d2);

    return 0;
}
</script></code></pre>

链接时报错：
<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:23:35]
$ g++ -c Swap.cpp

# root @ arch in ~/work on git:master x [17:23:39]
$ g++ -c main.cpp

# root @ arch in ~/work on git:master x [17:23:44]
$ g++ main.o Swap.o -o main
main.o: In function `main':
main.cpp:(.text+0xca): undefined reference to `void Swap<double>(double&, double&)'
collect2: error: ld returned 1 exit status
</script></code></pre>


因为我们只在 Swap.cpp 文件中显示实例化了 int、float 版本的 Swap() 函数；所以在链接 double 版本的函数时会找不到该函数；

**类模板的显式实例化**
类模板的显式实例化和函数模板类似；

以上节的 Point 类为例，针对`char *`类型的显式实例化（定义形式）代码为：`template class Point<char *, char *>;`
相应地，它的声明形式为：`extern template class Point<char *, char *>;`

不管是声明还是定义，都要带上class关键字，以表明这是针对类模板的；
另外需要注意的是，显式实例化一个类模板时，会一次性实例化该类的所有成员，包括成员变量和成员函数；

**总结**
函数模板和类模板的实例化语法是类似的，我们不妨对它们做一下总结：
`extern template declaration;`：实例化声明
`template declaration;`：实例化定义

对于函数模板来说，declaration 就是一个函数原型；对于类模板来说，declaration 就是一个类声明；

**显式实例化的缺陷**
C++ 支持显式实例化的目的是为「模块化编程」提供一种解决方案，这种方案虽然有效，但是也有明显的缺陷：程序员必须要在模板的定义文件（实现文件）中对所有使用到的类型进行实例化；

这就意味着，每次更改了模板使用文件（调用函数模板的文件，或者通过类模板创建对象的文件），也要相应地更改模板定义文件，以增加对新类型的实例化，或者删除无用类型的实例化；

一个模板可能会在多个文件中使用到，要保持这些文件的同步更新是非常困难的；而对于库的开发者来说，他不能提前假设用户会使用哪些类型，所以根本就无法使用显式实例化，只能将模板的声明和定义（实现）全部放到头文件中；

C++ 标准库几乎都是用模板来实现的，这些模板的代码也都位于头文件中；

总起来说，如果我们开发的模板只有我们自己使用，那也可以勉强使用显式实例化；如果希望让其他人使用（例如库、组件等），那只能将模板的声明和定义都放到头文件中了；

## 模板默认参数
在 C++11 中，模板和函数一样，可以有默认的参数；

基本语法规则可以参考 C++ 对函数的默认参数的要求：
- 默认参数只能放在形参列表的最后，而且一旦为某个形参指定了默认值，那么它后面的所有形参都必须有默认值；
- 在同一个作用域中只能指定一次默认参数；
C/C++ 共有四种作用域：`函数原型作用域`、`局部作用域（函数作用域）`、`块作用域`、`文件作用域（全局作用域）`


## 模板与inline声明
`inline`同样适用于函数模板、类模板，并且和 inline 函数一样，在函数定义处添加`inline`关键字进行修饰；
不过需要注意添加的位置，要在`模板头之后`、`函数返回值类型之前`添加`inline`关键字；

比如：
<pre><code class="language-cpp line-numbers"><script type="text/plain">// 函数模板声明
template <typename T1, typename T2>
auto Add(T1 a, T2 b) -> decltype(a + b);

// 函数模板定义
template <typename T1, typename T2>
inline auto Add(T1 a, T2 b) -> decltype(a + b) {
    return a + b;
}
</script></code></pre>
