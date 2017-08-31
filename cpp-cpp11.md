---
title: C++11 新特性
date: 2017-08-31 14:20:00
categories:
- cpp
tags:
- cpp
keywords: C++11 C++ 新特性
---

> 
C++11 新特性，auto/decltype自动类型推导、lambda表达式、移动语义、右值引用、智能指针、delete/default指示符等

<!-- more -->

## auto/decltype自动类型推导
### auto
auto 这个关键字 C++ 原先就有，用来指定存储器；但是这明显是多余的，没有必要；
因为很少有人去用这个东西，所以在 C++11 中就把原有的 auto 功能给废弃掉了，而变成了现在的类型推导关键字；

简单说一下 auto 的用法：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <typeinfo>

using namespace std;

const char * func(const char *name) {
    cout << name << endl;
    return name;
}

int main() {
    auto n = 10;    // int
    auto f = 3.14f; // float

    cout << typeid(n).name() << ", " << typeid(f).name() << endl;

    const char * (*f1)(const char *) = func;    // 函数指针 f1
    f1("original style");

    auto f2 = func; // 自动推导
    f2("c++11 style");

    cout << typeid(f1).name() << ", " << typeid(f2).name() << endl;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:38:58]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [14:39:32]
$ ./a.out
i, f
original style
c++11 style
PFPKcS0_E, PFPKcS0_E
</script></code></pre>


一般的，对于基本类型、结构体、类，并不推荐使用 auto 进行类型推导，因为这样的代码可读性不强；
但是对于诸如 函数指针、STL中某些复杂的类型、lambda表达式等，可以使用 auto，让代码更简洁；

从效率上来说，auto 不会对运行时效率产生影响，它是在编译的时候推导类型的；

auto和其他变量类型有明显的区别：
1. auto 声明的变量必须要初始化，否则编译器不能判断变量的类型；
2. auto 不能被声明为返回值，auto 不能作为形参，auto 不能被修饰为模板参数；


### decltype
decltype 和 auto 是相互对应的，它们经常在一些场所配合使用；
decltype 可以在编译的时候判断出一个变量或者表达式的类型，例如：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

const char * func(const char *name) {
    cout << name << endl;
    return name;
}

int main() {
    auto f1 = func;
    decltype(f1) f2 = func;
    f1("f1");
    f2("f2");
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:48:17]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [14:48:29]
$ ./a.out
f1
f2
</script></code></pre>


auto 和 decltype 还有一种经典的使用场合，看下面例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

template <typename T1, typename T2>
auto Add(T1 a, T2 b) -> decltype(a + b) {
    return a + b;
}

int main() {
    cout << Add(100, 200) << endl;
    cout << Add(1024l, 1.2f) << endl;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:52:48]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [14:52:58]
$ ./a.out
300
1025.2
</script></code></pre>


函数模板 Add 用来计算两个变量之和，如果两个类型 T1 和 T2 不一样的话，我们就无法事先知道返回值的类型，这时候 auto 和 decltype 就派上出场了；

让`a + b`的类型由编译器来决定，这样使用 decltype 就可以拿到返回的类型；
但是如果把`decltype(a + b)`放到函数命名的前面作为返回值的话，按照编译器的解析顺序，当解析到返回值`decltype(a + b)`的时候 a 和 b 还没有被定义，所以要重新声明 a 和 b 的类型，这样就会变的很复杂很难读懂；

于是 C++11 就改变了语法规则，把`decltype(a + b)`放到函数的后方，并用 auto 作为返回值来告诉编译器，真正的返回值在函数声明之后；简单的说 auto 可以作为返回值占位符来使返回值后置；

## functional标准库
C++11 引入了函数对象标准库`<functional>`，里面包含各种内建的**函数对象**以及相关的操作函数，非常方便；

**`std::function`**
`std::function`类模板是一种`通用的函数包装器`，它可以容纳所有可以调用的对象（Callable），包括函数、函数指针、Lambda表达式、bind表达式、成员函数及成员变量或者其他函数对象；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <functional>

using namespace std;

// 普通函数
int func(int a) {
    return a;
}

// lambda表达式
auto lambda = [](int a) -> int {
    return a;
};

// 函数对象
class Functor {
public:
    int operator()(int a) {
        return a;
    }
};

class TestClass {
public:
    // 成员函数
    int member_func(int a) {
        return a;
    }
    // 静态成员函数
    static int static_func(int a) {
        return a;
    }
};

int main() {
    function<int (int)> f;

    f = func;
    cout << f(100) << endl;

    f = lambda;
    cout << f(200) << endl;

    f = Functor();
    cout << f(300) << endl;

    TestClass testObj;
    f = bind(&TestClass::member_func, &testObj, placeholders::_1);
    cout << f(400) << endl;

    f = TestClass::static_func;
    cout << f(500) << endl;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:46:54]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [15:47:19]
$ ./a.out
100
200
300
400
500
</script></code></pre>


**`std::bind`**
顾名思义，`std::bind`函数用来绑定函数的某些参数并生成一个新的`function`对象；
bind用于实现`偏函数（Partial Function）`，相当于实现了函数式编程中的`Currying（柯里化）`；

所谓偏函数，就是给一个原有的函数固定几个参数，形成一个新的函数，因此可以进行个性化定制；

看这个例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <functional>

using namespace std;

void func1(int a, int b) {
    cout << "a = " << a << ", b = " << b << endl;
}

void func2(int a, int b, int c, int d, int e) {
    cout << "a = " << a
        << ", b = " << b
        << ", c = " << c
        << ", d = " << d
        << ", e = " << e << endl;
}

int main() {
    auto f1 = bind(func1, placeholders::_2, placeholders::_1);
    func1(1, 2);
    f1(1, 2);

    cout << "------------------------" << endl;

    auto f2 = bind(func2, placeholders::_1, 2, 3, 4, 5);
    func2(1, 2, 3, 4, 5);
    f2(1);
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:00:22]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [16:00:33]
$ ./a.out
a = 1, b = 2
a = 2, b = 1
------------------------
a = 1, b = 2, c = 3, d = 4, e = 5
a = 1, b = 2, c = 3, d = 4, e = 5
</script></code></pre>


## lambda表达式
Lambda 表达式来源于函数式编程，说白就了就是在使用的地方定义函数，有的语言叫`闭包`；

lambda 表达式语法：`[capture](parameters) mutable -> return-type { ... };`
- `[capture]`：捕捉列表；捕捉列表总是出现在Lambda函数的开始处；
实际上，[]是Lambda引出符，编译器根据该引出符判断接下来的代码是否是Lambda函数；
捕捉列表能够捕捉上下文中的变量以供Lambda函数使用；
- `(parameters)`：参数列表；与普通函数的参数列表一致；如果不需要参数传递，则可以连同括号“()”一起省略；
- `mutable`：mutable修饰符；默认情况下，Lambda函数为一个const成员函数，mutable可以取消其常量性；
在使用该修饰符时，参数列表不可省略（即使参数为空）；
- `-> return-type`：返回类型；用追踪返回类型形式声明函数的返回类型；
如果没有返回值（例如 void），其返回类型可以完全省略；
在返回类型明确的情况下，也可以省略该部分，让编译器对返回类型进行推导；
- `{ ... }`：函数体；内容与普通函数一样，不过除了可以使用参数之外，还可以使用所有捕获的变量；


与普通函数最大的区别是，除了可以使用参数以外，Lambda函数还可以通过捕获列表访问一些上下文中的数据；
捕捉列表描述了上下文中哪些数据可以被Lambda使用，以及使用方式（以值传递的方式或引用传递的方式）；
语法上，在“[]”包括起来的是捕捉列表，捕捉列表由多个捕捉项组成，并以逗号分隔；捕捉列表有以下几种形式：
- `[=]`：值传递方式捕捉所有父作用域的变量（包括 this）；
- `[&]`：引用传递方式捕捉所有父作用域的变量（包括 this）；
- `[var]`：值传递方式捕捉变量 var；
- `[&var]`：引用传递方式捕捉变量 var；
- `[this]`：值传递方式捕捉当前的 this 指针；
- `[&this]`：引用传递方式捕捉当前的 this 指针；
- 引用方式：因为 const 修饰对引用无效，所以通过引用捕获的变量可以在 lambda 表达式中进行读取、写入操作；
- 值传递方式：因为默认添加了 const 进行修饰，所以只能读取捕获到的变量，无法进行写入操作；
除非使用 mutable 修饰 lambda 表达式，这样就能够进行写入操作了，但是注意，因为是值传递方式，所以在 lambda 内部修改数据并不会影响外部变量的值；


上面提到了**父作用域**，也就是包含 Lambda 函数的语句块，说通俗点就是包含 Lambda 的“{}”代码块；

上面的捕捉列表还可以进行组合，例如：
- `[=, &a, &b]`：以引用传递的方式捕捉变量 a 和 b，以值传递方式捕捉其它所有变量；
- `[&,a,this]`：以值传递的方式捕捉变量 a 和 this，引用传递方式捕捉其它所有变量；


不过值得注意的是，捕捉列表不允许变量重复传递；下面一些例子就是典型的重复，会导致编译时期的错误；例如：
- `[=, a]`：已经以值传递方式捕捉了所有变量，但是重复捕捉了 a，错误；
- `[&, &this]`：已经以引用传递方式捕捉了所有变量，再捕捉 this 也是一种重复；


lambda 表达式的例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

void func(void (*f)(int, int), int i, int j) {
    f(i, j);
}

int main() {
    // lambda表达式作为函数参数传递
    func([](int a, int b){ cout << "a = " << a << ", b = " << b << endl; }, 10, 20);

    // 将lambda表达式保存至一个变量f
    auto f = [] (int a, int b) {
        cout << "a = " << a << ", b = " << b << endl;
    };
    func(f, 99, 999);

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:44:05]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [16:44:15]
$ ./a.out
a = 10, b = 20
a = 99, b = 999
</script></code></pre>


按值传递、引用传递的区别：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

int main() {
    int a = 10, b = 20;
    [a, b] () mutable -> void { a++; b--; cout << a << ", " << b << endl; } ();
    cout << a << ", " << b << endl;
    cout << "-------------------" << endl;

    a = 10; b = 20;
    [&a, &b] () mutable -> void { a++; b--; cout << a << ", " << b << endl; } ();
    cout << a << ", " << b << endl;
    cout << "-------------------" << endl;

    a = 10; b = 20;
    [&a, &b] () -> void { a++; b--; cout << a << ", " << b << endl; } ();
    cout << a << ", " << b << endl;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:51:26]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [16:51:40]
$ ./a.out
11, 19
10, 20
-------------------
11, 19
11, 19
-------------------
11, 19
11, 19
</script></code></pre>


如果你搞不懂上面的代码为什么会产生这样的结果，那么请跟随我一起来探究一下 lambda 表达式的实现原理；

**函数对象/仿函数**
类的对象跟括号`()`结合，表现出函数一般的行为，这个对象可以称作是函数对象：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

class Functor {
public:
    int operator()(int a) {
        return a;
    }
};

int func(int a) {
    return a;
}

int main() {
    cout << func(100) << endl;  // 普通函数的调用，func(100);

    Functor functor;
    cout << functor(200) << endl;   // 函数对象的调用，实际上是调用 functor.operator()(200);

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:01:45]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [17:01:54]
$ ./a.out
100
200
</script></code></pre>


这个示例说明函数对象的本质是重载了函数调用运算符；
当一个类重载了函数调用运算符`()`后，它的对象就成了`函数对象`；这是理解 lambda 表达式内部实现的基础；

**lambda表达式原理**
原理：编译器会把一个lambda表达式生成一个匿名类的匿名对象，并在类中重载函数调用运算符；

1) 无捕获列表、无参数列表：`[] () -> void { cout << "www.zfl9.com"; };`
假设生成的匿名类的类名为`TMP`，编译器会将上面的 lambda 表达式改写为下面的形式：
<pre><code class="language-cpp line-numbers"><script type="text/plain">class TMP {
public:
    void operator()() const {
        cout << "www.zfl9.com" << endl;
    }
};

TMP()();
</script></code></pre>


2) 无捕获列表、有参数列表：`[] (int a, int b) { return a + b; };`
<pre><code class="language-cpp line-numbers"><script type="text/plain">class TMP {
public:
    auto operator()(int a, int b) const -> decltype(a + b) {
        return a + b;
    }
};

auto ret = TMP()(a, b);
</script></code></pre>


3) 值捕获：`[name, age] () { cout << name << ", " << age << endl; };`
<pre><code class="language-cpp line-numbers"><script type="text/plain">const char *name = "zhang3";
int age = 18;

class TMP {
public:
    TMP(const char *name, int age) : name(name), age(age) {}
public:
    void operator()() const {
        cout << name << ", " << age << endl;
    }
private:
    const char *name;
    int age;
};
</script></code></pre>


4) 引用捕获：`[&name, &age] () { age++; cout << name << ", " << age << endl; };`
<pre><code class="language-cpp line-numbers"><script type="text/plain">const char *name = "zhang3";
int age = 18;

class TMP {
public:
    TMP(const char * &name, int &age) : name(name), age(age) {}
public:
    void operator()() const {
        age++;
        cout << name << ", " << age << endl;
    }
private:
    const char * &name;
    int &age;
};
</script></code></pre>


5) mutable修饰的值捕获：`[name, age] () mutable { age++; cout << name << ", " << age << endl; };`
<pre><code class="language-cpp line-numbers"><script type="text/plain">const char *name = "zhang3";
int age = 18;

class TMP {
public:
    TMP(const char * name, int age) : name(name), age(age) {}
public:
    void operator()() const {
        age++;
        cout << name << ", " << age << endl;
    }
private:
    mutable const char * name;
    mutable int age;
};
</script></code></pre>


## 相关的关键字
**nullptr 空指针常量**
`nullptr`是 C++11 标准用来表示空指针的常量值；
在 C 语言中，空指针的值表示为`#define NULL ((void *)0)`；
在 C++ 中，由于对语法的类型检查更为严格，因而空指针的值就不能表示为`(void *)0`；
所以至少自 C++98 开始`#define NULL 0`；但这会在函数重载时遇到新的困难，所以加入了 nullptr 来表示空指针；

**override、final**
在成员函数声明或定义中，`override`确保该函数为虚并重写来自基类的虚函数；若此非真则程序发生编译错误；
标记了`final`的虚函数不能被派生类重写，因此会将其从类的虚表中删除；而标记为`final`的类，编译器则根本不会生成虚表；并且不能被继承，为最终类；这样的代码显然更有效率；

**mutable、const**
`mutable`是可变的意思，mutable 用来修饰类的`非静态`和`非常量`数据成员；
而被 mutable 修饰的数据成员，可以在 const 成员函数中修改；

在之前的 lambda 表达式原理解析中，使用了 mutable，不清楚的可以去参考一下；

**delete、default**
C++ 的编译器在你没有定义某些成员函数的时候会给你的类自动生成这些函数，比如：构造函数，拷贝构造，析构函数，赋值函数；
有些时候，我们不想要这些函数，比如，构造函数，因为我们想做实现单例模式；传统的做法是将其声明成 private 类型；

在 C++11 中引入了两个指示符，`delete`告诉编译器不自动产生这个函数，`default`告诉编译器产生一个默认的函数；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

class A {
public:
    A() = default;  // 生成默认的构造函数
    ~A() = default; // 生成默认的析构函数
public:
    void * operator new(size_t) = delete; // 无法调用 new 运算符创建类 A 的实例
    void * operator new[](size_t) = delete;
    void operator delete(void *) = delete;
    void operator delete[](void *) = delete;
};

int main() {
    A a;    // 正确
    A *p = new A;   // 报错
    return 0;
}
</script></code></pre>


为什么我们需要default？我什么都不写不就是default吗？
不全然是，比如构造函数，因为只要你定义了一个构造函数，编译器就不会给你生成一个默认的了；所以，为了要让默认的和自定义的共存，才引入这个参数；

对于 delete 还有一个有用的地方，阻止函数的相关形参类型的调用：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

void func(int) {}
void func(double) = delete;

int main() {
    func(1);    // 正确
    func('A');  // 正确
    func(1.1);  // 错误
    return 0;
}
</script></code></pre>


**noexcept**
如果一个函数不能抛出异常，或者一个程序并没有接获某个函数所抛出的异常并进行处理，那么这个函数可以用新的`noexcept`关键字对其进行修饰，表示这个函数不会抛出异常或者抛出的异常不会被接获并处理；

例如：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

void func() noexcept {
    throw "exception data";
}

int main() {
    try {
        func();
    } catch (...) {
        cout << "catch exception" << endl;
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [18:18:12] C:1
$ g++ a.cpp
a.cpp: In function ‘void func()’:
a.cpp:6:11: warning: throw will always call terminate() [-Wterminate]
     throw "exception data";
           ^~~~~~~~~~~~~~~~

# root @ arch in ~/work on git:master x [18:18:13]
$ ./a.out
terminate called after throwing an instance of 'char const*'
[1]    32267 abort (core dumped)  ./a.out
</script></code></pre>


如果一个经过 noexcept 修饰的函数抛出异常，程序会通过调用 terminate() 来结束执行；
通过 terminate() 的调用来结束程序的执行会带来很多问题，例如，无法保证对象的析构函数的正常调用，无法保证栈的自动释放，同时也无法在没有遇到任何问题的情况下重新启动程序；所以，它是不可靠的；

这和 C++11 之前的 throw() 异常规范一样的作用，但是比它却要高效得多；

同时，我们还可以让一个函数根据不同的条件实现 noexcept 修饰或者是无 noexcept 修饰；
声明的通常形式是`noexcept(expression)`，并且单独的一个“noexcept”关键字实际上就是的一个`noexcept(true)`的简化；
`expression`是一个bool表达式，如果结果为 true，那么表示该函数不抛出异常，反之则可能抛出异常；

**explicit**
C++ 提供了关键字 `explicit`，可以阻止不应该允许的经过转换构造函数进行的隐式转换的发生；声明为`explicit`的构造函数不能在隐式转换中使用：

没有使用 explicit 关键字的构造函数（一个参数）：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

class A {
public:
    A(int) {}
};

int main() {
    A a(10);    // 显示调用，正确
    A b = 10;   // 隐式调用，转换构造函数，也正确
    return 0;
}
</script></code></pre>


使用 explicit 声明的构造函数：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>

using namespace std;

class A {
public:
    explicit A(int) {}
};

int main() {
    A a(10);    // 显示调用，正确
    A b = 10;   // 隐式调用，错误
    return 0;
}
</script></code></pre>


## 委托构造
所谓委托构造就是：在一个构造函数中调用另外一个构造函数，这就是委托的意味；
不同的构造函数自己负责处理自己的不同情况，把最基本的构造工作委托给某个基础构造函数完成，实现分工协作；

比如：
<pre><code class="language-cpp line-numbers"><script type="text/plain">class A {
public:
    A(int) {}
    A() : A(0) {}   // 调用构造函数 A(int)
    A(float) { A(); }   // 调用构造函数 A()
};
</script></code></pre>


## 统一的初始化语法
C++ 之前的初始化语法很乱，有四种初始化方式，而且每种之前甚至不能相互转换；让人有种剪不断，理还乱的感觉；

1) 小括号初始化方法：`int a = int(5);`

2) 赋值初始化方法：`int a = 3;`

3) POD 聚合，也就是经常使用的大括号初始化方法：`int arr[2] = {0, 1};`

4) 构造函数初始化：
<pre><code class="language-cpp line-numbers"><script type="text/plain">class test {
public:
    test(int var1, int var2) : a(var1), b(var2) {}
private:
    int a;
    int b;
};
</script></code></pre>


以上就是 C++03 之前的四种初始化方法，当然不是每种类型都有四种初始化方式；看到这里，估计你和我的感受差不多了，感觉这么多种初始化方式没必要呀，为毛不去统一一下，降低学习 C++ 的难度呢？

这不，C++ 的开发者们急我们之所急，在 N 年后的 C++11 中推出了统一初始化方法的新特性：统一使用花括号`{}`进行初始化
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <string>

using namespace std;

class test {
public:
    test(int a, int b) : m_a{a}, m_b{b}, m_arr{1, 2, 3, 4, 5} {}
private:
    int m_a;
    int m_b;
    int m_arr[5];
};

int main() {
    int n0 = 50;
    int n1 = {100};
    int n2{200};
    cout << n0 << ", " << n1 << ", " << n2 << endl;

    string str0("www.zfl9.com");
    string str1 = {"www.zfl9.com"};
    string str2{"www.zfl9.com"};
    cout << str0 << ", " << str1 << ", " << str2 << endl;

    test t0(1, 2);
    test t1 = {1, 2};
    test t2{1, 2};
    return 0;
}
</script></code></pre>


但是，我觉得这个看起来更别扭，还不如之前的 C/C++ 风格的初始化方法。所以尽量不要这样使用！


## 匿名命名空间
当定义一个命名空间时，可以忽略这个命名空间的名称：
<pre><code class="language-cpp line-numbers"><script type="text/plain">namespace {
    char c;
    int i;
    double d;
}
</script></code></pre>


编译器在内部会为这个命名空间生成一个唯一的名字，而且还会为这个匿名的命名空间生成一条 using 指令；所以上面的代码在效果上等同于：
<pre><code class="language-cpp line-numbers"><script type="text/plain">namespace __UNIQUE_NAME_ {
    char c;
    int i;
    double d;
}
using namespace __UNIQUE_NAME_;
</script></code></pre>


在匿名命名空间中声明的名称也将被编译器转换，与编译器为这个匿名命名空间生成的唯一内部名称(即这里的`__UNIQUE_NAME_`)绑定在一起；

还有一点很重要，就是这些名称具有`internal链接属性`，这和声明为`static`的全局名称的链接属性是相同的；
即**名称的作用域被限制在当前文件中**，无法通过在另外的文件中使用`extern`声明来进行链接；

注意：命名空间都是具有 external 链接属性的，只是匿名的命名空间产生的`__UNIQUE_NAME_`在别的文件中无法得到，这个唯一的名字是不可见的；

C++ 新的标准中提倡使用匿名命名空间，而不推荐使用 static，因为 static 用在不同的地方，含义不同容易造成混淆；

比如：带 static 的类成员为类共享，而变量前的 static 又表示内部链接、存储范围；
另外，static 不能修饰 class 定义，那样就可以将类定义放在匿名命名空间中达到同样的效果；
