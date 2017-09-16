---
title: C++ 异常
date: 2017-08-29 07:51:00
categories:
- cpp
tags:
- cpp
keywords: C++ 异常 exception
---

> 
C++ 异常

<!-- more -->

## 异常处理入门（try和catch）
程序的错误大致可以分为三种，分别是`语法错误`、`逻辑错误`和`运行时错误`：
1) 语法错误在编译和链接阶段就能发现，只有 100% 符合语法规则的代码才能生成可执行程序；
语法错误是最容易发现、最容易定位、最容易排除的错误，程序员最不需要担心的就是这种错误；

2) 逻辑错误是说我们编写的代码思路有问题，不能够达到最终的目标，这种错误可以通过调试来解决；

3) 运行时错误是指程序在运行期间发生的错误，例如除数为 0、内存分配失败、数组越界、文件不存在等；
C++ `异常（Exception）`机制就是为解决运行时错误而引入的；

运行时错误如果放任不管，系统就会执行默认的操作，`终止程序运行`，也就是我们常说的`程序崩溃（Crash）`；
C++ 提供了`异常（Exception）机制`，让我们能够捕获运行时错误，给程序一次“起死回生”的机会，或者至少告诉用户发生了什么再终止程序；

一个发生运行时错误的例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <string>

using namespace std;

int main() {
    string str = "www.zfl9.com";

    char c1 = str[20];  // 下标越界，c1为垃圾值
    cout << c1 << endl;

    char c2 = str.at(20);   // 下标越界，抛出异常std::out_of_range
    cout << c2 << endl;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [8:45:25] C:134
$ g++ a.cpp

# root @ arch in ~/work on git:master x [8:45:31]
$ ./a.out
þ
terminate called after throwing an instance of 'std::out_of_range'
  what():  basic_string::at: __n (which is 20) >= this->size() (which is 12)
[1]    12312 abort (core dumped)  ./a.out
</script></code></pre>


at() 是 string 类的一个成员函数，它会根据下标来返回字符串的一个字符；
与 [] 不同，at() 会检查下标是否越界，如果越界就抛出一个异常；而 [] 不做检查，不管下标是多少都会照常访问；

上面的代码中，下标 20 显然超出了字符串 str 的长度；
由于第 9 行代码不会检查下标越界，虽然有逻辑错误，但是程序能够正常运行；
而第 12 行代码则不同，at() 函数检测到下标越界会抛出一个异常，这个异常可以由程序员处理，但是我们在代码中并没有处理，所以系统只能**执行默认的操作**，也即**终止程序执行**；

**捕获异常**
我们可以借助 C++ 异常机制来捕获上面的异常，避免程序崩溃；捕获异常的语法为：
<pre><code class="language-cpp line-numbers"><script type="text/plain">try {
    // 可能抛出异常的语句
} catch(exceptionType variable) {
    // 处理异常的语句
}
</script></code></pre>


`try`和`catch`都是 C++ 中的关键字，后跟语句块，不能省略`{}`；
try 中包含可能会抛出异常的语句，一旦有异常抛出就会被后面的 catch 捕获；

从 try 的意思可以看出，它只是“检测”语句块有没有异常，如果没有发生异常，它就“检测”不到；
catch 是“抓住”的意思，用来捕获并处理 try 检测到的异常；如果 try 语句块没有检测到异常（没有异常抛出），那么就不会执行 catch 中的语句；

catch 关键字后面的`exceptionType variable`指明了当前 catch 可以处理的异常类型，以及具体的出错信息；
我们稍后再对异常类型展开讲解，当务之急是演示一下`try-catch`的用法，先让读者有一个整体上的认识；

修改上面的代码，加入捕获异常的语句：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <string>
#include <exception>

using namespace std;

int main() {
    string str = "www.zfl9.com";

    try {
        char c1 = str[20];  // 下标越界，c1为垃圾值
        cout << c1 << endl;
    } catch(exception &e) {
        cout << e.what() << endl;
    }

    try {
        char c2 = str.at(20);   // 下标越界，抛出异常std::out_of_range
        cout << c2 << endl;
    } catch(exception &e) {
        cout << e.what() << endl;
    }

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [8:59:32]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [8:59:42]
$ ./a.out

basic_string::at: __n (which is 20) >= this->size() (which is 12)
</script></code></pre>


可以看出，第一个 try 没有捕获到异常，输出了一个没有意义的字符（垃圾值）；
因为 [] 不会检查下标越界，不会抛出异常，所以即使有错误，try 也检测不到；
换句话说，**发生异常时必须将异常明确地抛出，try 才能检测到；如果不抛出来，即使有异常 try 也检测不到**；
所谓抛出异常，就是明确地告诉程序发生了什么错误；

第二个 try 检测到了异常，并交给 catch 处理，执行 catch 中的语句；
需要说明的是，异常一旦抛出，会立刻被 try 检测到，并且不会再执行异常点（异常发生位置）后面的语句；
本例中抛出异常的位置是第 18 行的 at() 函数，它后面的 cout 语句就不会再被执行，所以看不到它的输出；

说得直接一点，检测到异常后程序的执行流会发生跳转，从异常点跳转到 catch 所在的位置，位于异常点之后的、并且在当前 try 块内的语句就都不会再执行了；
即使 catch 语句成功地处理了错误，程序的执行流也不会再回退到异常点，所以这些语句永远都没有执行的机会了；本例中，第 19 行代码就是被跳过的代码；

执行完 catch 块所包含的代码后，程序会继续执行 catch 块后面的代码，就恢复了正常的执行流；

为了演示「不明确地抛出异常就检测不到异常」，大家不妨将第 11 行代码改为`char c1 = str[100000000];`；
访问第 20 个字符可能不会发生异常，但是访问第 1 亿个字符肯定会发生异常了，这个异常就是**内存访问错误**；
运行更改后的程序，会发现第 11 行代码产生了异常，导致程序崩溃了，这说明 try-catch 并没有捕获到这个异常；

关于「如何抛出异常」，我们将在下节讲解，这里重点是让大家明白异常的处理流程：
`抛出（Throw）` --> `检测（Try）` --> `捕获（Catch）`

**发生异常的位置**
异常可以发生在`当前的 try 块中`，也可以发生在`try 块所调用的某个函数中`，或者是`所调用的函数又调用了另外的一个函数，这个另外的函数中发生了异常`；这些异常，都可以被 try 检测到；

1) 下面的例子演示了 try 块中直接发生的异常：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

int main() {
    try {
        throw "Unknown Exception";
        cout << "after throw" << endl;
    } catch(const char * &e) {
        cout << e << endl;
    }

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:11:26]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [9:11:28]
$ ./a.out
Unknown Exception
</script></code></pre>


2) 下面的例子演示了 try 块中调用的某个函数中发生了异常：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

void func() {
    throw "Unknown Exception (call func())";
    cout << "after throw" << endl;
}

int main() {
    try {
        func();
        cout << "after func()" << endl;
    } catch(const char * &e) {
        cout << e << endl;
    }

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:14:42]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [9:14:51]
$ ./a.out
Unknown Exception (call func())
</script></code></pre>


3) try 块中调用了某个函数，该函数又调用了另外的一个函数，这个另外的函数抛出了异常：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

void func_inner() {
    throw "Unknown Exception (call func_inner())";
    cout << "func_inner()" << endl;
}

void func_outer() {
    func_inner();
    cout << "func_outer()" << endl;
}

int main() {
    try {
        func_outer();
        cout << "after func_outer()" << endl;
    } catch(const char * &e) {
        cout << e << endl;
    }

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:17:44]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [9:17:45]
$ ./a.out
Unknown Exception (call func_inner())
</script></code></pre>


## 异常类型以及多级catch
首先来回顾一下上节讲到的 try-catch 的用法：
<pre><code class="language-cpp line-numbers"><script type="text/plain">try {
    // 可能抛出异常的语句
} catch(exceptionType variable) {
    // 处理异常的语句
}
</script></code></pre>


我们还遗留下一个问题，就是 catch 关键字后边的`exceptionType variable`，这节就来详细分析一下：

exceptionType 是异常类型，它指明了当前的 catch 可以处理什么类型的异常；
variable 是一个变量，用来接收异常信息；当程序抛出异常时，会创建一份数据，这份数据包含了错误信息，程序员可以根据这些信息来判断到底出了什么问题，接下来怎么处理；

异常既然是一份数据，那么就应该有数据类型；
C++ 规定，异常类型可以是 int、char、float、bool 等基本类型，也可以是指针、数组、字符串、结构体、类等聚合类型；
C++ 语言本身以及标准库中的函数抛出的异常，都是`exception`类或其子类的异常；也就是说，抛出异常时，会创建一个 exception 类或其子类的对象；

`exceptionType variable`和`函数的形参`非常类似，当异常发生后，会将异常数据传递给 variable 这个变量，这和函数传参的过程类似；
当然，只有跟 exceptionType 类型匹配的异常数据才会被传递给 variable，否则 catch 不会接收这份异常数据，也不会执行 catch 块中的语句；换句话说，catch 不会处理当前的异常；

**我们可以将 catch 看做一个没有返回值的函数，当异常发生后 catch 会被调用，并且会接收实参（异常数据）**

但是 catch 和真正的函数调用又有区别：
1) 真正的函数调用，形参和实参的类型必须要匹配，或者可以自动转换，否则在编译阶段就报错了；
2) 而对于 catch，异常是在运行阶段产生的，它可以是任何类型，没法提前预测，所以不能在编译阶段判断类型是否正确，只能等到程序运行后，真的抛出异常了，再将异常类型和 catch 能处理的类型进行匹配，匹配成功的话就“调用”当前的 catch，否则就忽略当前的 catch；

总起来说，catch 和真正的函数调用相比，多了一个「在运行阶段将实参和形参匹配」的过程；

另外需要注意的是，如果不希望 catch 处理异常数据，也可以将 variable 省略掉，也即写作：
<pre><code class="language-cpp line-numbers"><script type="text/plain">try {
    // 可能抛出异常的语句
} catch(exceptionType) {
    // 处理异常的语句
}
</script></code></pre>


这样只会将异常类型和 catch 所能处理的类型进行匹配，不会传递异常数据了；

如果想匹配任何类型的异常，那么可以这样写：
<pre><code class="language-cpp line-numbers"><script type="text/plain">try {
    // 可能抛出异常的语句
} catch (...) {
    // 处理异常的语句
}
</script></code></pre>


**多级 catch**
前面的例子中，一个 try 对应一个 catch，这只是最简单的形式；其实，一个 try 后面可以跟多个 catch：
<pre><code class="language-cpp line-numbers"><script type="text/plain">try {
    // 可能抛出异常的语句
} catch(exception_type_1 e) {
    // 处理异常的语句
} catch(exception_type_2 e) {
    // 处理异常的语句
}
</script></code></pre>


当异常发生时，程序会按照**从上到下**的顺序，将异常类型和 catch 所能接收的类型逐个匹配；
一旦找到类型匹配的 catch 就停止检索，并将异常交给当前的 catch 处理（其他的 catch 不会被执行）；
如果最终也没有找到匹配的 catch，就只能交给系统处理，终止程序的运行；

下面的例子演示了多级 catch 的使用：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

class Base {};
class Derived : public Base {};

int main() {
    try {
        throw Derived();    // 创建一个 Derived 匿名对象，并抛出
    } catch (int) {
        cout << "Exception Type: int" << endl;
    } catch (char *) {
        cout << "Exception Type: char *" << endl;
    } catch (Base) {    // 向上转型，匹配成功
        cout << "Exception Type: class Base" << endl;
    } catch (Derived) {
        cout << "Exception Type: class Derived" << endl;
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:46:34]
$ g++ a.cpp
a.cpp: In function ‘int main()’:
a.cpp:17:7: warning: exception of type ‘Derived’ will be caught
     } catch (Derived) {
       ^~~~~
a.cpp:15:7: warning:    by earlier handler for ‘Base’
     } catch (Base) {    // 向上转型，匹配成功
       ^~~~~

# root @ arch in ~/work on git:master x [9:46:36]
$ ./a.out
Exception Type: class Base
</script></code></pre>


**catch 在匹配过程中的类型转换**
C/C++ 中存在多种多样的类型转换，以`普通函数（非模板函数）`为例，发生函数调用时，如果实参和形参的类型不是严格匹配，那么会将实参的类型进行适当的转换，以适应形参的类型，这些转换包括：
- 算数转换：例如 int 转换为 float，char 转换为 int，double 转换为 int 等；
- 向上转型：也就是派生类向基类的转换；
- const 转换：也即将非 const 类型转换为 const 类型；
- 数组或函数指针转换：如果函数形参不是引用类型，那么数组名会转换为数组指针，函数名也会转换为函数指针；
- 用户自定的类型转换；

catch 在匹配异常类型的过程中，也会进行类型转换，但是这种转换受到了更多的限制，仅能进行「`向上转型`」、「`const 转换`」和「`数组或函数指针转换`」，其他的都不能应用于 catch；

`向上转型`在上面的例子中已经发生了，下面的例子演示了`const 转换`以及`数组和指针`的转换：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

int main() {
    try {
        int arr[] = {1, 2, 3, 4, 5};
        throw arr;
    } catch (const int *) {
        cout << "Exception Type: const int *" << endl;
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:52:18]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [9:52:20]
$ ./a.out
Exception Type: const int *
</script></code></pre>


arr 的类型为`int [5]`，由于没有匹配的类型，所以先转换为`int *`，还是没有匹配的，最后降为`const int *`，匹配成功

## throw关键字（抛出异常+异常规范）
C++ 异常处理的流程，具体为：`抛出（Throw）` --> `检测（Try）` --> `捕获（Catch）`
**异常必须显式地抛出，才能被检测和捕获到；如果没有显式的抛出，即使有异常也检测不到**

在 C++ 中，我们使用 throw 关键字来显式地抛出异常，它的用法为：`throw exceptionData;`
`exceptionData`是异常数据，它可以包含任意的信息，完全有程序员决定；
`exceptionData`可以是 int、float、bool 等基本类型，也可以是指针、数组、字符串、结构体、类等聚合类型；

**向上层抛出异常数据**
如果当前`catch`捕获到了异常，但是并不想处理，可以将其继续往外层抛出，被外层的`catch`再次捕获，让他们处理：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

void func() {
    try {
        throw "exception data";
    } catch (...) {
        throw;  // 不作处理，直接往上层抛出
    }
}

int main() {
    try {
        func();
    } catch (const char * &e) {
        cout << e << endl;
    }
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:33:14]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [13:33:38]
$ ./a.out
exception data
</script></code></pre>


**throw 用作异常规范**
> 
异常规范是 C++98 新增的一项功能，但是后来的 C++11 已经将它抛弃了，不再建议使用；

throw 关键字除了可以用在函数体中抛出异常，还可以用在函数头和函数体之间，指明当前函数能够抛出的异常类型，这称为`异常规范（Exception specification）`，有些教程也称为`异常指示符`或`异常列表`；

请看下面的例子：
`double func(char param) throw(int);`
这条语句声明了一个名为 func 的函数，它的返回值类型为 double，有一个 char 类型的参数，并且只能抛出 int 类型的异常；如果抛出其他类型的异常，try 将无法捕获，只能终止程序；

如果函数会抛出多种类型的异常，那么可以用逗号隔开：
`double func(char param) throw(int, char, exception);`

如果函数不会抛出任何异常，那么`()`中什么也不写：
`double func(char param) throw();`
如此，func() 函数就不能抛出任何类型的异常了，即使抛出了，try 也检测不到；

**请抛弃异常规范，不要再使用它**
异常规范的初衷是好的，它希望让程序员看到函数的定义或声明后，立马就知道该函数会抛出什么类型的异常，这样程序员就可以使用 try-catch 来捕获了；如果没有异常规范，程序员必须阅读函数源码才能知道函数会抛出什么异常；

不过这有时候也不容易做到：
例如，func_outer() 函数可能不会引发异常，但它调用了另外一个函数 func_inner()，这个函数可能会引发异常；
再如，您编写的函数调用了老式的库函数，此时不会引发异常，但是库更新以后这个函数却引发了异常；
总之，异常规范的初衷实现起来有点困难，所以大家达成的一致意见是，最好不要使用异常规范；

## exception类
C++语言本身或者标准库抛出的异常都是`exception`的子类，称为`标准异常（Standard Exception）`；你可以通过下面的语句来捕获所有的标准异常：
<pre><code class="language-cpp line-numbers"><script type="text/plain">try {
    // 可能抛出异常的语句
} catch (exception &e) {
    // 处理异常的语句
}
</script></code></pre>


之所以使用引用，是为了提高效率；如果不使用引用，就要经历一次对象拷贝（要调用拷贝构造函数）的过程；

exception 类位于`<exception>`头文件中，它被声明为：
<pre><code class="language-cpp line-numbers"><script type="text/plain">class exception {
public:
    exception() throw();  // 构造函数
    exception(const exception &) throw();  // 拷贝构造函数
    exception & operator=(const exception &) throw();  // 运算符重载
    virtual ~exception() throw();  // 虚析构函数
    virtual const char * what() const throw();  // 虚函数
}
</script></code></pre>


这里需要说明的是`what()`函数；
what() 函数返回一个能识别异常的字符串，正如它的名字“what”一样，可以粗略地告诉你这是什么异常；
不过C++标准并没有规定这个字符串的格式，各个编译器的实现也不同，所以 what() 的返回值仅供参考；

**exception 类的继承层次**
![exception 继承层次](/images/cpp-exception.jpg)

**exception 类的直接派生类**：

异常名称|说  明
:---:|:---:
logic_error|逻辑错误
runtime_error|运行时错误
bad_alloc|使用 new 或 new[] 分配内存失败时抛出的异常
bad_typeid|使用 typeid 操作一个 NULL 指针，而且该指针是带有虚函数的类，这时抛出 bad_typeid 异常
bad_cast|使用 dynamic_cast 转换失败时抛出的异常
ios_base::failure|I/O 过程中出现的异常
bad_exception|这是个特殊的异常，如果函数的异常列表里声明了 bad_exception 异常，当函数内部抛出了异常列表中没有的异常时，如果调用的 unexpected() 函数中抛出了异常，不论什么类型，都会被替换为 bad_exception 类型

**logic_error 的派生类**：

异常名称|说  明
:---:|:---:
length_error|试图生成一个超出该类型最大长度的对象时抛出该异常，例如 vector 的 resize 操作
domain_error|参数的值域错误，主要用在数学函数中，例如使用一个负值调用只能操作非负数的函数
out_of_range|超出有效范围
invalid_argument|参数不合适；在标准库中，当利用 string 对象构造 bitset 时，而 string 中的字符不是 0 或 1 的时候，抛出该异常；

**runtime_error 的派生类**：

异常名称|说  明
:---:|:---:
range_error|计算结果超出了有意义的值域范围
overflow_error|算术计算上溢
underflow_error|算术计算下溢


## RAII和异常安全
**RAII是什么**？
RAII 是 Resource Acquisition Is Initialization 的简称，是 C++ 语言的一种管理资源、避免泄漏的惯用法；利用的就是 C++ 构造的对象最终会被销毁的原则；RAII 的做法是使用一个对象，在其构造时获取对应的资源，在对象生命期内控制对资源的访问，使之始终保持有效，最后在对象析构的时候，释放构造时获取的资源；

**为什么要使用RAII**？
上面说到 RAII 是用来管理资源、避免资源泄漏的方法；那么资源是如何定义的？在计算机系统中，资源是数量有限且对系统正常运行具有一定作用的元素；比如：网络套接字、互斥锁、文件句柄和内存等等，它们属于系统资源；由于系统的资源是有限的，就好比自然界的石油，铁矿一样，不是取之不尽，用之不竭的，所以，我们在编程使用系统资源时，都必须遵循一个步骤：
1) 申请资源；
2) 使用资源；
3) 释放资源；

第一步和第二步缺一不可，因为资源必须要申请才能使用的，使用完成以后，必须要释放，如果不释放的话，就会造成资源泄漏；

一个最简单的例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
using namespace std;

int main() {
    int *p = new int[10]; // 申请内存资源
    // TODO 使用内存资源
    delete[] p; // 释放内存资源
    p = nullptr;
    return 0;
}
</script></code></pre>



**RAII的作用**
RAII 的主要作用是在不失代码简洁性的同时，可以很好地保证代码的异常安全性；

当一个函数需要通过多个局部变量来管理资源时，RAII 就显得非常好用；因为只有被构造成功(构造函数没有抛出异常)的对象才会在返回时调用析构函数，同时析构函数的调用顺序恰好是它们构造顺序的反序，这样既可以保证多个资源(对象)的正确释放，又能满足多个资源之间的依赖关系；

由于 RAII 可以极大地简化资源管理，并有效地保证程序的正确和代码的简洁，所以通常会强烈建议在 C++ 中使用它；

**RAII对比finally**
虽然 RAII 和 finally 都能保证资源管理时的异常安全，但相对来说，使用 RAII 的代码相对更加简洁；
正如比雅尼·斯特劳斯特鲁普所说，“在真实环境中，调用资源释放代码的次数远多于资源类型的个数，所以相对于使用用 finally 来说，使用 RAII 能减少代码量”

**构造函数、析构函数可以抛出异常吗**？
构造函数：在必要的情况下，可以抛出异常，表示对象构造失败，这时候是不会调用析构函数的，因为构造的并不是一个完整的对象；
析构函数：一定不能抛出异常，即使会发生异常也要在析构函数内部把异常吞掉，否则在异常转递的`堆栈辗转开解（stack-unwinding）`的过程中，terminate 函数被调用，而 terminate 通常调用 abort() 结束程序！

总结起来说：
如果在构造函数中申请了系统资源，那么在析构函数中需要有相应的资源释放操作，并且析构函数不能抛出任何异常！
对于动态分配的内存，请尽量不要直接使用 new/delete 操作符，尽量使用 C++11 中的智能指针来进行内存的申请和释放！


**异常安全**
异常安全的代码是指，满足两个条件：
- 异常中立性
 - 指当你的代码（包括你调用的代码）引发异常时，这个异常能保持原样传递到外层调用代码；
- 异常安全性
 - 抛出异常后，资源不泄露；
 - 抛出异常后，不会使原有数据恶化（例如正常指针变野指针）；
 - 少些 try catch，因为大量的 try catch 会影响代码逻辑；导致代码丑陋混乱不优雅；



一段代码要具有异常安全性，必须同时具有异常中立性和一定等级的异常安全性保证；

C++ 中"异常安全函数"提供了三种安全等级：
1) `基本承诺`：如果异常被抛出，对象内的任何成员仍然能保持有效状态，没有数据的破坏及资源泄漏；但对象的现实状态是不可估计的，即不一定是调用前的状态，但至少保证符合对象正常的要求；
2) `强烈保证`：如果异常被抛出，对象的状态保持不变；即如果调用成功，则完全成功；如果调用失败，则对象依然是调用前的状态；
3) `不抛异常保证`：函数承诺不会抛出任何异常；一般内置类型的所有操作都有不抛异常的保证；

如果一个函数不能提供上述保证之一，则不具备异常安全性；

最后，需要提醒的是：
1) 不要滥用异常，也不要摒弃异常，只有在必要的情况下去考虑 try...catch；
2) 千万不要使用 try...catch 进行程序的逻辑控制，不要拿它们当 if...else 用；
3) 为了让代码具有更好的异常安全性，首先是”用对象来管理资源“（RAII），以避免资源的泄漏；其次，在异常安全性等级上，应该尽可能地往更高的等级上来限制；
4) “在恰当的场合使用恰当的特性”对每个称职的 C++ 程序员来说都是一个基本标准；
