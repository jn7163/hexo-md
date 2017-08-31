---
title: C++ 运算符重载
date: 2017-08-26 15:39:00
categories:
- cpp
tags:
- cpp
keywords: C++ 运算符重载 operator reload overload
---

> 
C++ 运算符重载，`函数重载（Function Overloading）`可以让一个函数名有多种功能，在不同情况下进行不同的操作；`运算符重载（Operator Overloading）`也是一个道理，同一个运算符可以有不同的功能；

<!-- more -->

## 运算符重载的概念
实际上，我们已经在不知不觉中使用了运算符重载；
例如，`+号`可以对不同类型（int、float 等）的数据进行加法操作；`<<`既是位移运算符，又可以配合 cout 向控制台输出数据；C++ 本身已经对这些运算符进行了重载；

C++ 也允许程序员自己重载运算符，这给我们带来了很大的便利；

**以成员函数形式重载运算符**
下面的代码定义了一个复数类，通过运算符重载，可以用`+号`实现复数的加法运算：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class complex {
public:
    complex(double real = 0.0, double imag = 0.0) : m_real(real), m_imag(imag) {}
    void print() const;
    complex operator+(const complex &A) const;
private:
    double m_real;
    double m_imag;
};

void complex::print() const {
    printf("%g + %gi\n", m_real, m_imag);
}

complex complex::operator+(const complex &A) const {
    return complex(m_real + A.m_real, m_imag + A.m_imag);
}

int main() {
    complex A(1.24, 5.7), B(4, 3.14), C;
    C = A + B;
    C.print();
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:51:24]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [15:51:36]
$ ./a.out
5.24 + 8.84i
</script></code></pre>


本例中定义了一个复数类 complex，m_real 表示实部，m_imag 表示虚部，第 9 行声明了运算符重载，第 19 行进行了实现（定义）；认真观察这两行代码，可以发现运算符重载的形式与函数非常类似；

运算符重载其实就是定义一个函数，在函数体内实现想要的功能，当用到该运算符时，编译器会自动调用这个函数；也就是说，运算符重载是通过函数实现的，它本质上是函数重载；

对于上面的`+`运算符重载：`complex operator+(const complex &A) const;`
函数名为`operator+`，返回值类型为`complex`，参数为`const complex &`，并且是一个const成员；
在 25 行中，`C = A + B;`会被转换为类似的函数调用形式：`C = A.operator+(B);`，这实质上就是调用对象的成员函数；

**运算符重载函数除了函数名有特定的格式，其它地方和普通函数并没有区别**；

**在全局范围内重载运算符**
运算符重载函数不仅可以作为类的成员函数，还可以作为全局函数；更改上面的代码，在全局范围内重载+，实现复数的加法运算：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class complex {
public:
    complex(double real = 0.0, double imag = 0.0) : m_real(real), m_imag(imag) {}
    void print() const;
    friend complex operator+(const complex &A, const complex &B);
private:
    double m_real;
    double m_imag;
};

void complex::print() const {
    printf("%g + %gi\n", m_real, m_imag);
}

complex operator+(const complex &A, const complex &B) {
    return complex(A.m_real + B.m_real, A.m_imag + B.m_imag);
}

int main() {
    complex A(1.24, 5.7), B(4, 3.14), C;
    C = A + B;
    C.print();
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:02:16]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [16:02:29]
$ ./a.out
5.24 + 8.84i
</script></code></pre>


运算符重载函数不是 complex 类的成员函数，但是却用到了 complex 类的 private 成员变量，所以必须在 complex 类中将该函数声明为`友元函数`；

当执行`C = A + B;`语句时，编译器检测到+号两边都是 complex 对象，就会转换为类似这样的函数调用：`C = operator+(A, B);`

**小结**
虽然运算符重载所实现的功能完全可以用函数替代，但运算符重载使得程序的书写更加人性化，易于阅读；
运算符被重载后，原有的功能仍然保留，没有丧失或改变；通过运算符重载，扩大了C++已有运算符的功能，使之能用于对象；

## 运算符重载的规则
运算符重载是通过函数重载实现的，概念上大家都很容易理解，这节我们来说一下运算符重载的注意事项：

1) 并不是所有的运算符都可以重载，能够重载的运算符包括：
`+`、`-`、`*`、`/`、`%`、`^`、`&`、`|`、`~`、`!`、`=`、`<`、`>`、`+=`、`-=`、`*=`、`/=`、`%=`、`^=`、`&=`、`|=`、`<<`、`>>`、`<<=`、`>>=`、`==`、`!=`、`<=`、`>=`、`&&`、`||`、`++`、`--`、`,`、`->*`、`->`、`()`、`[]`、`new`、`new[]`、`delete`、`delete[]`

上述运算符中，`[]`是下标运算符，`()`是函数调用运算符；自增自减运算符的前置和后置形式都可以重载；长度运算符`sizeof`、条件运算符`: ?`、成员选择符`.`和域解析运算符`::`不能被重载；

2) **重载不能改变运算符的优先级和结合性**；

3) 重载不会改变运算符的用法，原有有几个操作数、操作数在左边还是在右边，这些都不会改变；例如`~`号右边只有一个操作数，`+`号总是出现在两个操作数之间，重载后也必须如此；

4) 运算符重载函数不能有默认的参数，否则就改变了运算符操作数的个数，这显然是错误的；

5) 运算符重载函数既可以作为类的成员函数，也可以作为全局函数；

将运算符重载函数作为类的成员函数时，二元运算符的参数只有一个，一元运算符不需要参数；之所以少一个参数，是因为这个参数是隐含的；

将运算符重载函数作为全局函数时，二元操作符就需要两个参数，一元操作符需要一个参数，而且其中必须有一个参数是对象，好让编译器区分这是程序员自定义的运算符，防止程序员修改用于内置类型的运算符的性质；
如果有两个参数，这两个参数可以都是对象，也可以一个是对象，一个是C++内置类型的数据；

另外，将运算符重载函数作为全局函数时，一般都需要在类中将该函数声明为友元函数；原因很简单，该函数大部分情况下都需要使用类的 private 成员；

6) 箭头运算符`->`、下标运算符`[]`、函数调用运算符`()`、赋值运算符`=`只能以**成员函数**的形式重载；

## 重载数学运算符
四则运算符（`+`、`-`、`*`、`/`、`+=`、`-=`、`*=`、`/=`）和关系运算符（`>`、`<`、`<=`、`>=`、`==`、`!=`）都是数学运算符，它们在实际开发中非常常见，被重载的几率也很高，并且有着相似的重载格式；

本节以复数类 Complex 为例对它们进行重载，重在演示运算符重载的语法以及规范：
复数能够进行完整的四则运算，但不能进行完整的关系运算：我们只能判断两个复数是否相等，但不能比较它们的大小，所以不能对`>`、`<`、`<=`、`>=`进行重载；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <cmath>

using namespace std;

class complex {
public:
    complex(double real = 0.0, double imag = 0.0) : m_real(real), m_imag(imag) {}
    friend ostream & operator<<(ostream &out, complex &A);
    friend complex operator+(const complex &A, const complex &B);
    friend complex operator-(const complex &A, const complex &B);
    friend complex operator*(const complex &A, const complex &B);
    friend complex operator/(const complex &A, const complex &B);
    friend bool operator==(const complex &A, const complex &B);
    friend bool operator!=(const complex &A, const complex &B);
    void operator+=(const complex &A);
    void operator-=(const complex &A);
    void operator*=(const complex &A);
    void operator/=(const complex &A);
private:
    double m_real;
    double m_imag;
};

ostream & operator<<(ostream &out, complex &A) {
    out << A.m_real << " + " << A.m_imag << "i";
    return out;
}

complex operator+(const complex &A, const complex &B) {
    return complex(A.m_real + B.m_real, A.m_imag + B.m_imag);
}

complex operator-(const complex &A, const complex &B) {
    return complex(A.m_real - B.m_real, A.m_imag - B.m_imag);
}

complex operator*(const complex &A, const complex &B) {
    return complex(A.m_real*B.m_real - A.m_imag*B.m_imag, A.m_imag*B.m_real + A.m_real*B.m_imag);
}

complex operator/(const complex &A, const complex &B) {
    return complex((A.m_real*B.m_real + A.m_imag*B.m_imag)/(pow(B.m_real, 2) + pow(B.m_imag, 2)), (A.m_imag*B.m_real - A.m_real*B.m_imag)/(pow(B.m_real, 2) + pow(B.m_imag, 2)));
}

bool operator==(const complex &A, const complex &B) {
    if (A.m_real == B.m_real && A.m_imag == B.m_imag) {
        return true;
    } else {
        return false;
    }
}

bool operator!=(const complex &A, const complex &B) {
    if (A.m_real != B.m_real || A.m_imag != B.m_imag) {
        return true;
    } else {
        return false;
    }
}

void complex::operator+=(const complex &A) {
    m_real += A.m_real;
    m_imag += A.m_imag;
}

void complex::operator-=(const complex &A) {
    m_real -= A.m_real;
    m_imag -= A.m_imag;
}

void complex::operator*=(const complex &A) {
    m_real = m_real * A.m_real - m_imag * A.m_imag;
    m_imag = m_imag * A.m_real + m_real * A.m_imag;
}

void complex::operator/=(const complex &A) {
    m_real = (m_real*A.m_real + m_imag*A.m_imag) / (pow(A.m_real, 2) + pow(A.m_imag, 2));
    m_imag = (m_imag*A.m_real - m_real*A.m_imag) / (pow(A.m_real, 2) + pow(A.m_imag, 2));
}

int main() {
    complex c1(10, 20), c2(3, 5), c3;

    c3 = c1 + c2;
    cout << c3 << endl;

    c3 = c1 - c2;
    cout << c3 << endl;

    c3 = c1 * c2;
    cout << c3 << endl;

    c3 = c1 / c2;
    cout << c3 << endl;

    c3 += c1;
    cout << c3 << endl;

    c3 -= c2;
    cout << c3 << endl;

    c3 *= c1;
    cout << c3 << endl;

    c3 /= c2;
    cout << c3 << endl;

    if (c1 == c2) {
        cout << "c1 == c2" << endl;
    }

    if (c1 != c2) {
        cout << "c1 != c2" << endl;
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [18:09:09]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [18:09:13]
$ ./a.out
13 + 25i
7 + 15i
-70 + 110i
3.82353 + 0.294118i
13.8235 + 20.2941i
10.8235 + 15.2941i
-197.647 + -3800i
-576.263 + -250.55i
c1 != c2
</script></code></pre>


需要注意的是，我们以全局函数的形式重载了`+`、`-`、`*`、`/`、`==`、`!=`，以成员函数的形式重载了`+=`、`-=`、`*=`、`/=`，而且应该坚持这样做，不能一股脑都写作成员函数或者全局函数，具体原因我们将在下节讲解；

## 以成员函数和全局函数重载运算符的区别
在上节的例子中，我们以全局函数的形式重载了`+`、`-`、`*`、`/`、`==`、`!=`，以成员函数的形式重载了`+=`、`-=`、`*=`、`/=`，而没有一股脑都写成全局函数或者成员函数，这样做是有原因的，这节我们就来分析一下：

**简单地了解转换构造函数**

全局函数（友元函数）形式重载 + 号；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

class complex {
public:
    complex(double real = 0.0, double imag = 0.0) : m_real(real), m_imag(imag) {}
    friend ostream & operator<<(ostream &out, complex &A);
    friend complex operator+(const complex &A, const complex &B);
private:
    double m_real;
    double m_imag;
};

ostream & operator<<(ostream &out, complex &A) {
    out << A.m_real << " + " << A.m_imag << "i";
    return out;
}

complex operator+(const complex &A, const complex &B) {
    return complex(A.m_real + B.m_real, A.m_imag + B.m_imag);
}

int main() {
    complex a(1, 1), b;
    b = a + a;
    cout << b << endl;
    b = a + 1.1;
    cout << b << endl;
    b = 2.2 + a;
    cout << b << endl;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [19:11:20]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [19:11:21]
$ ./a.out
2 + 2i
2.1 + 1i
3.2 + 1i
</script></code></pre>


成员函数形式重载 + 号：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

class complex {
public:
    complex(double real = 0.0, double imag = 0.0) : m_real(real), m_imag(imag) {}
    friend ostream & operator<<(ostream &out, complex &A);
    complex operator+(const complex &A) const;
private:
    double m_real;
    double m_imag;
};

ostream & operator<<(ostream &out, complex &A) {
    out << A.m_real << " + " << A.m_imag << "i";
    return out;
}

complex complex::operator+(const complex &A) const {
    return complex(m_real + A.m_real, m_imag + A.m_imag);
}

int main() {
    complex a(1, 1), b;
    b = a + a;
    cout << b << endl;
    b = a + 1.1;
    cout << b << endl;
//  b = 2.2 + a;
//  cout << b << endl;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [19:14:53]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [19:15:03]
$ ./a.out
2 + 2i
2.1 + 1i
</script></code></pre>


给构造函数提供了实部和虚部的默认参数 0.0 后，使该构造函数同时也成为了一个`转换构造函数`；

> 
C++ 会对成员函数的参数进行类型转换，而不会对调用成员函数的对象进行类型转换；

对于全局函数形式的重载 + ：
先来看`b = a + 1.1;`，实际上会被转换为`b = operator+(a, 1.1);`这样的形式进行调用；因为存在转换构造函数，并且第一个参数为 double，那么编译器就会将 1.1 转换为一个匿名 complex 对象`complex(1.1)`，进而在运算符重载函数内部将它们两个的实部，虚部相加，然后返回一个匿名 complex 对象并赋值给 b；

而对于`b = 2.2 + a;`，也是一样的道理，被转换为`b = operator+(2.2, a);`这样的形式，然后又调用转换构造函数将 2.2 转换为一个匿名 complex 对象`complex(2.2)`，后面的步骤同上；

对于成员函数形式的重载 + ：
对于`b = a + 1.1;`，被转换为`b = a.operator+(1.1);`，1.1 也是被转换为`complex(1.1)`；
对于`b = 2.2 + a;`，被转换为`b = (2.2).operator+(a);`，这很显然是不正确的，进而编译报错；

**为什么以全局函数方式重载运算符 +**
> 
以全局函数的形式重载 +，是为了保证 + 运算符的操作数能够被**对称的处理**；换句话说，`小数（double）`在 + 左边和右边都是正确的；

**为什么以成员函数方式重载运算符 +=**
我们首先要明白，运算符重载的初衷是给类添加新的功能，方便类的运算，它作为类的成员函数是理所应当的，是首选的；
不过，类的成员函数不能对称地处理数据，程序员必须在（参与运算的）所有类型的内部都重载当前的运算符；

以上面的情况为例，我们必须在 Complex 和 double 内部都重载 + 运算符，这样做不但会增加运算符重载的数目，还要在许多地方修改代码，这显然不是我们所希望的，所以 C++ 进行了折中，允许以全局函数（友元函数）的形式重载运算符；

采用全局函数能使我们定义这样的运算符，它们的参数具有逻辑的对称性；
与此相对应的，把运算符定义为成员函数能够保证在调用时对第一个（最左的）运算对象不出现类型转换，也就是上面提到的「C++ 不会对调用成员函数的对象进行类型转换」；

总结起来说，有一部分运算符重载既可以是成员函数也可以是全局函数，虽然没有一个必然的、不可抗拒的理由选择成员函数，但我们应该优先考虑成员函数，这样更符合运算符重载的初衷；
另外有一部分运算符重载必须是全局函数，这样能保证参数的对称性；

除了 C++ 规定的几个特定的运算符外，暂时还没有发现必须以成员函数的形式重载的运算符；

> 
C++ 规定，`箭头运算符->`、`下标运算符[]`、`函数调用运算符()`、`赋值运算符=`只能以**成员函数**的形式重载；

## 重载>>和<<（输入输出运算符）
在C++中，标准库本身已经对`左移运算符<<`和`右移运算符>>`分别进行了重载，使其能够用于不同数据的输入输出，但是输入输出的对象只能是 C++ 内置的数据类型（例如 bool、int、double 等）和标准库所包含的类类型（例如 string、complex、ofstream、ifstream 等）；

如果我们自己定义了一种新的数据类型，需要用输入输出运算符去处理，那么就必须对它们进行重载；
本节以前面的 complex 类为例来演示输入输出运算符的重载；

其实 C++ 标准库已经提供了 complex 类，能够很好地支持复数运算；
我们自己又定义了一个 complex 类的目的仅仅是为了做演示，并没有别的意图；

本节要达到的效果就是让复数的输入输出和 int、float 等基本类型一样简单；

假设 num1、num2 是复数，那么输出形式就是：`cout << num1 << num2 << endl;`，输入形式就是：`cin >> num1 >> num2;`；

cout 是 istream 类的对象，cin 是 ostream 类的对象，要想达到这个目标，就必须以全局函数（友元函数）的形式重载`<<`和`>>`，否则就要修改标准库中的类，这显然不是我们所期望的；

<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

class complex {
public:
    complex(double real = 0.0, double imag = 0.0) : m_real(real), m_imag(imag) {}
    friend istream & operator>>(istream &in, complex &A);
    friend ostream & operator<<(ostream &out, complex &A);
private:
    double m_real;
    double m_imag;
};

istream & operator>>(istream &in, complex &A) {
    in >> A.m_real >> A.m_imag;
    return in;
}

ostream & operator<<(ostream &out, complex &A) {
    out << A.m_real << " + " << A.m_imag << "i";
    return out;
}

int main() {
    complex a, b;
    cin >> a >> b;
    cout << a << "\t" << b << endl;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [19:59:28]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [20:00:10]
$ ./a.out
1 2 3 4
1 + 2i	3 + 4i
</script></code></pre>


## 重载[]（下标运算符）
C++ 规定，`下标运算符[]`必须以**成员函数**的形式进行重载；

该重载函数在类中的声明格式：`返回值类型 & operator[](参数);`，或者`const 返回值类型 & operator[](参数) const;`

使用第一种声明方式，`[]`不仅可以访问元素，还可以修改元素；
使用第二种声明方式，`[]`只能访问而不能修改元素；

在实际开发中，我们应该同时提供以上两种形式，这样做是为了适应 const 对象，因为通过 const 对象只能调用 const 成员函数，如果不提供第二种形式，那么将无法访问 const 对象的任何元素；

下面我们通过一个具体的例子来演示如何重载`[]`；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Array {
public:
    Array(int len = 0);
    ~Array();
public:
    int length() const;
    void print() const;
public:
    int & operator[](int i);
    const int & operator[](int i) const;
private:
    int m_len;
    int *m_ptr;
};

Array::Array(int len) : m_len(len) {
    if (len == 0) {
        m_ptr = nullptr;
    } else {
        m_ptr = new int[len];
    }
}

Array::~Array() {
    delete[] m_ptr;
}

int Array::length() const {
    return m_len;
}

void Array::print() const {
    printf("Array[%d] = { ", m_len);
    for (int i=0; i<m_len; i++) {
        printf("%d, ", m_ptr[i]);
    }
    printf("\b\b }\n");
}

int & Array::operator[](int i) {
    return m_ptr[i];
}

const int & Array::operator[](int i) const {
    return m_ptr[i];
}

int main() {
    int len;
    printf("array_len: ");
    scanf("%d", &len);

    Array arr1(len);
    for (int i=0; i<arr1.length(); i++) {
        arr1[i] = i*5;
    }
    arr1.print();

    const Array arr2(len);
    printf("arr2[0] = %d\n", arr2[0]);
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:21:40] C:130
$ g++ a.cpp

# root @ arch in ~/work on git:master x [20:21:42]
$ ./a.out
array_len: 4
Array[4] = { 0, 5, 10, 15 }
arr2[0] = 0
</script></code></pre>


## 重载`++`和`--`（自增自减运算符）
`自增++`和`自减--`都是一元运算符，它的前置形式和后置形式都可以被重载：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <cstdio>

using namespace std;

class stopwatch {
public:
    stopwatch() : m_min(0), m_sec(0) {}
public:
    void setzero() { m_min = m_sec = 0; }
    stopwatch run();
    stopwatch operator++(); // ++i 前置形式
    stopwatch operator++(int); // i++ 后置形式
    friend ostream & operator<<(ostream &out, const stopwatch &s);
private:
    int m_min;
    int m_sec;
};

stopwatch stopwatch::run() {
    m_sec++;
    if (m_sec == 60) {
        m_min++;
        m_sec = 0;
    }
    return *this;
}

stopwatch stopwatch::operator++() {
    return run();
}

stopwatch stopwatch::operator++(int) {
    stopwatch s = *this;
    run();
    return s;
}

ostream & operator<<(ostream &out, const stopwatch &s) {
    out << s.m_min << ":" << s.m_sec << endl;
    return out;
}

int main() {
    stopwatch s1, s2;

    s1 = s2++;
    cout << s1 << s2;

    s1.setzero(); s2.setzero();
    s1 = ++s2;
    cout << s1 << s2;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:36:29] C:127
$ g++ a.cpp

# root @ arch in ~/work on git:master x [20:36:31]
$ ./a.out
0:0
0:1
0:1
0:1
</script></code></pre>


自减运算符的重载与上面类似，这里不再赘述；

## 重载new和delete运算符
内存管理运算符：`new`、`new[]`、`delete`和`delete[]`也可以进行重载，其重载形式既可以是类的成员函数，也可以是全局函数；

一般情况下，内建的内存管理运算符就够用了，只有在需要自己管理内存时才会重载；

**重载 new 运算符**：`void * operator new(size_t size);`
**重载 new[] 运算符**：`void * operator new[](size_t size);`

返回值是`void *`类型，并且都有一个参数，为`size_t`类型；
重载`new`或`new[]`时，无论是作为成员函数还是作为全局函数，它的第一个参数必须是`size_t`类型；
`size_t`表示的是要分配空间的大小，对于`new[]`的重载函数而言，`size_t`则表示所需要分配的所有空间的总和；

当然，重载函数也可以有其他参数，但都必须有默认值，并且第一个参数的类型必须是`size_t`；

**重载 delete 运算符**：`void operator delete(void *ptr);`
**重载 delete[] 运算符**：`void operator delete[](void *ptr);`

两种重载形式的返回值都是`void`类型，并且都必须有一个`void *`类型的指针作为参数，该指针指向需要释放的内存空间；
