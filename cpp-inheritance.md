---
title: C++ 继承与派生
date: 2017-08-25 10:48:00
categories:
- cpp
tags:
- cpp
keywords: C++ 继承与派生
---

> 
C++ 继承与派生，继承性是面向对象程序设计最重要的特征，可以说，如果没有掌握继承性，就等于没有掌握类和对象的精华，就是没有掌握面向对象程序设计的真谛

<!-- more -->

## 继承的概念
**继承是类与类之间的关系**，是一个很简单很直观的概念，与现实世界中的继承类似，例如儿子继承父亲的财产；

`继承（Inheritance）`可以理解为一个类从另一个类获取成员变量和成员函数的过程；
例如类 B 继承于类 A，那么 B 就拥有 A 的成员变量和成员函数；**被继承的类称为父类或基类，继承的类称为子类或派生类**；

派生类除了拥有基类的成员，还可以定义自己的新成员，以增强类的功能；

以下是两种典型的使用继承的场景：
1) 当你创建的新类与现有的类相似，只是多出若干成员变量或成员函数时，可以使用继承，这样不但会减少代码量，而且新类会拥有基类的所有功能；

2) 当你需要创建多个类，它们拥有很多相似的成员变量或成员函数时，也可以使用继承；可以将这些类的共同成员提取出来，定义为基类，然后从基类继承，既可以节省代码，也方便后续修改成员；

下面，我们定义了一个基类 People，然后由此派生出 Student 类：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class People {
public:
    void setname(const char *name) { m_name = name; }
    void setage(int age) { m_age = age; }
    const char *getname() const { return m_name; }
    int getage() const { return m_age; }
private:
    const char *m_name;
    int m_age;
};

class Student : public People {
public:
    void setscore(float score) { m_score = score; }
    float getscore() const { return m_score; }
private:
    float m_score;
};

int main() {
    Student s;
    s.setname("Otokaze");
    s.setage(18);
    s.setscore(111);
    printf("name: %s, age: %d, score: %g\n", s.getname(), s.getage(), s.getscore());
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [11:05:22]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [11:05:47]
$ ./a.out
name: Otokaze, age: 18, score: 111
</script></code></pre>


本例中，People 是基类，Student 是派生类。Student 类继承了 People 类的成员，同时还新增了自己的成员变量 m_score 和成员函数 setscore()、getscore()；这些继承过来的成员，可以通过子类对象访问，就像自己的一样；

`class Student : public People`
class 后面的“Student”是新声明的派生类，冒号后面的“People”是已经存在的基类；在“People”之前有一关键宇 public，用来表示是`公有继承`；

继承方式包括 `public（公有的）`、`private（私有的）`和 `protected（受保护的）`，此项是可选的，如果不写，那么默认为 `private`；

## 继承权限和继承方式
继承方式限定了基类成员在派生类中的访问权限，包括 public（公有的）、private（私有的）和 protected（受保护的）；此项是可选项，如果不写，默认为 private（成员变量和成员函数默认也是 private）；

现在我们知道，public、protected、private 三个关键字除了可以修饰类的成员，还可以指定继承方式；

**public、protected、private 修饰类的成员**
类成员的访问权限由高到低依次为 public --> protected --> private，public 成员可以通过对象来访问，private 成员不能通过对象访问；

现在再来补充一下 protected；protected 成员和 private 成员类似，也不能通过对象访问；但是当存在继承关系时，protected 和 private 就不一样了：基类中的 protected 成员可以在派生类中使用，而基类中的 private 成员不能在派生类中使用；

**public、protected、private 指定继承方式**
不同的继承方式会影响基类成员在派生类中的访问权限：

**1) public继承方式**
基类中所有 public 成员在派生类中为 public 属性；
基类中所有 protected 成员在派生类中为 protected 属性；
基类中所有 private 成员在派生类中不能使用。

**2) protected继承方式**
基类中的所有 public 成员在派生类中为 protected 属性；
基类中的所有 protected 成员在派生类中为 protected 属性；
基类中的所有 private 成员在派生类中不能使用。

**3) private继承方式**
基类中的所有 public 成员在派生类中均为 private 属性；
基类中的所有 protected 成员在派生类中均为 private 属性；
基类中的所有 private 成员在派生类中不能使用。

通过上面的分析可以发现：
1) 基类成员在派生类中的访问权限不得高于继承方式中指定的权限；例如，当继承方式为 protected 时，那么基类成员在派生类中的访问权限最高也为 protected，高于 protected 的会降级为 protected，但低于 protected 不会升级；再如，当继承方式为 public 时，那么基类成员在派生类中的访问权限将保持不变；

也就是说，继承方式中的 public、protected、private 是用来指明基类成员在派生类中的最高访问权限的；

2) 不管继承方式如何，基类中的 private 成员在派生类中始终不能使用（不能在派生类的成员函数中访问或调用）；

3) 如果希望基类的成员能够被派生类继承并且毫无障碍地使用，那么这些成员只能声明为 public 或 protected；只有那些不希望在派生类中使用的成员才声明为 private；

4) 如果希望基类的成员既不向外暴露（不能通过对象访问），还能在派生类中使用，那么只能声明为 protected；

注意，我们这里说的是基类的 private 成员不能在派生类中使用，并没有说基类的 private 成员不能被继承；
实际上，基类的 private 成员是能够被继承的，并且（成员变量）会占用派生类对象的内存，它只是在派生类中不可见，导致无法使用罢了；
private 成员的这种特性，能够很好的对派生类隐藏基类的实现，以体现面向对象的`封装性`；

> 
由于 private 和 protected 继承方式会改变基类成员在派生类中的访问权限，导致继承关系复杂，所以实际开发中我们一般使用 public；

在派生类中访问基类 private 成员的唯一方法就是借助基类的非 private 成员函数，如果基类没有非 private 成员函数，那么该成员在派生类中将无法访问（除非使用下面讲到的 using 关键字）；

**改变访问权限**
使用 using 关键字可以改变基类成员在派生类中的访问权限，例如将 public 改为 private、将 protected 改为 public 等：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
protected:
    int m_a;
    char m_b;
    float m_c;
private:
    char *m_d;
};

class B : public A {
public:
    using A::m_a;
protected:
    using A::m_b;
//    using A::m_d; // 基类的private成员在派生类中不可见
private:
    using A::m_c;
};

int main() {
    B b;
    b.m_a = 1;
    return 0;
}
</script></code></pre>


> 
注意：在派生类中只能访问基类的非 private 成员，使用 using 时，也必须遵循这个规则；
对于父类的 public/protected 成员：using 可以将其改为 public、protected、private 属性；
对于父类的 private 成员：由于在派生类中不可见，所以也就不能使用 using 改变属性；

## 继承时的名字屏蔽
如果派生类中的成员（包括成员变量和成员函数）和基类中的成员重名，那么就会遮蔽从基类继承过来的成员；
所谓遮蔽，就是在派生类中使用该成员（包括在定义派生类时使用，也包括通过派生类对象访问该成员）时，实际上使用的是派生类新增的成员，而不是从基类继承来的；

比如，在父类 A 和 派生类 B 中都有函数 func()，在调用的时候就会发生名字屏蔽：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    void func() {
        printf("class A: func()\n");
    }
};

class B : public A {
public:
    void func() {
        printf("class B: func()\n");
    }
};

int main() {
    B b;
    b.func();
    b.A::func();
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [11:53:52]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [11:57:54]
$ ./a.out
class B: func()
class A: func()
</script></code></pre>


在基类 A 和 派生类 B 中都存在一个名字一样的函数 func()，名字一样，造成屏蔽；
由于派生类中的 func() 优先级更高，如不明确指定使用父类的 func()，默认调用派生类的 func() 函数；

**基类成员函数和派生类成员函数不构成重载**
基类成员和派生类成员的**名字一样**时会造成遮蔽，这句话对于成员变量很好理解，对于成员函数要引起注意，**不管函数的参数如何，只要名字一样就会造成遮蔽**；

换句话说，**基类成员函数和派生类成员函数不会构成重载，如果派生类有同名函数，那么就会遮蔽基类中的所有同名函数，不管它们的参数是否一样**；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    void func() {
        printf("class A: func()\n");
    }
    void func(int) {
        printf("class A: func(int)\n");
    }
};

class B : public A {
public:
    void func(char) {
        printf("class B: func(char)\n");
    }
    void func(float) {
        printf("class B: func(float)\n");
    }
};

int main() {
    B b;
    b.func('A'); // 正确
    b.func(1.1f); // 正确
//    b.func(); // 错误
//    b.func(10); // 错误
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:09:38]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [12:09:40]
$ ./a.out
class B: func(char)
class B: func(float)
</script></code></pre>


如果说有重载关系，那么也是 A 类的两个 func 构成重载，而 B 类的两个 func 构成另外的重载；

## 类继承时的作用域嵌套
类其实也是一种作用域，每个类都会定义它自己的作用域，在这个作用域内我们再定义类的成员；
当存在继承关系时，派生类的作用域嵌套在基类的作用域之内，如果一个名字在派生类的作用域内无法找到，编译器会继续到外层的基类作用域中查找该名字的定义；

换句话说，作用域能够彼此包含，被包含（或者说被嵌套）的作用域称为内层作用域（inner scope），包含着别的作用域的作用域称为外层作用域（outer scope）；
一旦在外层作用域中声明（或者定义）了某个名字，那么它所嵌套着的所有内层作用域中都能访问这个名字；同时，允许在内层作用域中重新定义外层作用域中已有的名字；

举个栗子：C 继承于 B，B 继承于 A，即 A --> B --> C；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    void func() {
        printf("class_A: func()\n");
    }
    void print() {
        printf("class_A: print()\n");
    }
};

class B : public A {
public:
    void func() {
        printf("class_B: func()\n");
    }
};

class C : public B {
public:
    void print() {
        printf("class_C: print()\n");
    }
};

int main() {
    B b;
    b.func();   // 调用 B 的 func();
    b.print();  // 调用 A 的 print();

    C c;
    c.func();   // 调用 B 的 func();
    c.print();  // 调用 C 的 print();
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:58:55]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [12:59:16]
$ ./a.out
class_B: func()
class_A: print()
class_B: func()
class_C: print()
</script></code></pre>


如果当前类中没有找到该成员，就逐个的往外层作用域中查找，直到找到为止，否则就报错；
这个过程叫做`名字查找（name lookup）`，也就是在作用域链中寻找与所用名字最匹配的声明（或定义）的过程；

对于成员变量这个过程很好理解，对于成员函数要引起注意，编译器**仅仅是根据函数的名字来查找的，不会理会函数的参数**；
换句话说，一旦内层作用域有同名的函数，不管有几个，编译器都不会再到外层作用域中查找，编译器仅把内层作用域中的这些同名函数作为一组候选函数；这组候选函数就是一组重载函数；

说白了，只有一个作用域内的同名函数才具有重载关系，不同作用域内的同名函数是会造成遮蔽，使得外层函数无效。派生类和基类拥有不同的作用域，所以它们的同名函数不具有重载关系；

## 继承时的对象内存模型
**没有继承关系时**：成员变量和成员函数会分开存储，成员变量存储在栈、堆、全局数据区，而成员函数存储在代码区；
**有继承关系时**：派生类的内存模型可以看成是基类成员变量和新增成员变量的总和，而所有成员函数仍然存储在代码区，由所有对象共享；

**基类的成员变量排在前面，派生类的排在后面**；

在派生类的对象模型中，会包含所有基类的成员变量（包括被屏蔽的基类成员变量等）；这种设计方案的优点是访问效率高，能够在派生类对象中直接访问基类变量，无需经过好几层间接计算；

我们来看一下这个例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    A() : m_a(1) {}
protected:
    int m_a;
};

class B : public A {
public:
    B() : A(), m_a(2) {}
protected:
    int m_a;
};

class C : public B {
public:
    C() : B(), m_b(11) {}
protected:
    int m_b;
};

class D : public C {
public:
    D() : C(), m_b(22) {}
protected:
    int m_b;
};

int main() {
    D d;
    printf("sizeof(d) = %lu\n", sizeof(d));
    printf("A::m_a = %d, B::m_a = %d, C::m_b = %d, D::m_b = %d\n",
            *(int *)&d,
            *(int *)((long)&d + 4),
            *(int *)((long)&d + 8),
            *(int *)((long)&d + 12));
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:20:53]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [13:21:08]
$ ./a.out
sizeof(d) = 16
A::m_a = 1, B::m_a = 2, C::m_b = 11, D::m_b = 22
</script></code></pre>


即使造成名字屏蔽，父类的成员变量还是排在前面，并不会有什么影响；

## 派生类的构造函数
前面我们说基类的成员函数可以被继承，可以通过派生类的对象访问，但这仅仅指的是普通的成员函数，**类的构造函数不能被继承**；
构造函数不能被继承是有道理的，因为即使继承了，它的名字和派生类的名字也不一样，不能成为派生类的构造函数，当然更不能成为普通的成员函数；

在设计派生类时，对继承过来的成员变量的初始化工作也要由派生类的构造函数完成，但是大部分基类都有 private 属性的成员变量，它们在派生类中无法访问，更不能使用派生类的构造函数来初始化；

这种矛盾在C++继承中是普遍存在的，解决这个问题的思路是：**在派生类的构造函数中调用基类的构造函数**；
还有一点要注意，**派生类构造函数中只能调用直接基类的构造函数，不能调用间接基类的**；

**事实上，通过派生类创建对象时必须要调用基类的构造函数，这是语法规定**；
换句话说，定义派生类构造函数时最好指明基类构造函数；如果不指明，就调用基类的默认构造函数（不带参数的构造函数）；如果没有默认构造函数，那么编译失败；

派生类构造函数总是先调用基类构造函数再执行其他代码（包括参数初始化表以及函数体中的代码）；
并且**只能将基类构造函数的调用放在函数头部，不能放在函数体中**；

**构造函数调用顺序**
基类构造函数总是被优先调用，这说明创建派生类对象时，会先调用基类构造函数，再调用派生类构造函数，如果继承关系有好几层的话，例如：A --> B --> C
那么创建 C 类对象时构造函数的执行顺序为：A类构造函数 --> B类构造函数 --> C类构造函数

**构造函数的调用顺序是按照继承的层次自顶向下、从基类再到派生类的**

举例说明：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    A() {
        printf("class_A A()\n");
    }
    A(int) {
        printf("class_A A(int)\n");
    }
};

class B : public A {
public:
    B() {
        printf("class_B B()\n");
    }
    B(char) {
        printf("class_B B(char)\n");
    }
};

class C : public B {
public:
    C() : B('A') {
        printf("class_C C()\n");
    }
};

int main() {
    C c;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:47:50]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [13:48:09]
$ ./a.out
class_A A()
class_B B(char)
class_C C()
</script></code></pre>

## 派生类的析构函数
**和构造函数类似，析构函数也不能被继承**；
与构造函数不同的是，在派生类的析构函数中不用显式地调用基类的析构函数，因为每个类只有一个析构函数，编译器知道如何选择，无需程序员干涉；

另外**析构函数的执行顺序和构造函数的执行顺序也刚好相反**：
- 创建派生类对象时，构造函数的执行顺序和继承顺序相同，即先执行基类构造函数，再执行派生类构造函数；
- 而销毁派生类对象时，析构函数的执行顺序和继承顺序相反，即先执行派生类析构函数，再执行基类析构函数；

<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    A() {
        printf("A()\n");
    }
    ~A() {
        printf("~A()\n");
    }
};

class B : public A {
public:
    B() {
        printf("B()\n");
    }
    ~B() {
        printf("~B()\n");
    }
};

class C : public B {
public:
    C() {
        printf("C()\n");
    }
    ~C() {
        printf("~C()\n");
    }
};

int main() {
    C c;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:55:19]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [13:55:23]
$ ./a.out
A()
B()
C()
~C()
~B()
~A()
</script></code></pre>

## 类的多继承
在前面的例子中，派生类都只有一个基类，称为`单继承（Single Inheritance）`；
除此之外，C++也支持`多继承（Multiple Inheritance）`，即一个派生类可以有两个或多个基类；

> 
多继承容易让代码逻辑复杂、思路混乱，一直备受争议，中小型项目中较少使用，后来的 Java、C#、PHP 等干脆取消了多继承；

C++ 多继承的语法也很简单，将多个基类用逗号隔开即可：`class C : public A, public B { ... }`，表示 C 继承 A 和 B；

**多继承下的构造函数**
多继承形式下的构造函数和单继承形式基本相同，只是要在派生类的构造函数中调用多个基类的构造函数；
比如对于上面的 C 类的构造函数：`C(形参列表) : A(实参列表), B(实参列表) {函数体}`

**基类构造函数的调用顺序和和它们在派生类构造函数中出现的顺序无关，而是和声明派生类时基类出现的顺序相同**

**多继承形式下析构函数的执行顺序和构造函数的执行顺序相反**，和单继承中的析构函数一样；

**命名冲突**
当两个或多个基类中有同名的成员时，如果直接访问该成员，就会产生命名冲突，编译器不知道使用哪个基类的成员；
这个时候需要在成员名字前面加上**类名**和**域解析符**`::`，以显式地指明到底使用哪个类的成员，消除二义性；

<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    A() {
        printf("A()\n");
    }
    ~A() {
        printf("~A()\n");
    }
protected:
    void print() {
        printf("A::print()\n");
    }
};

class B {
public:
    B() {
        printf("B()\n");
    }
    ~B() {
        printf("~B()\n");
    }
protected:
    void print() {
        printf("B::print()\n");
    }
};

class C : public A, public B {
public:
    C() : A(), B() {
        printf("C()\n");
    }
    ~C() {
        printf("~C()\n");
    }
public:
    void print() {
        A::print();
        B::print();
    }
};

int main() {
    C c;
    c.print();
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:12:30]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [14:12:50]
$ ./a.out
A()
B()
C()
A::print()
B::print()
~C()
~B()
~A()
</script></code></pre>

## 多继承时的对象内存模型
前面讲解了单继承时对象的内存模型，这节我们来分析一下多继承时对象的内存模型；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    int m_a;
};

class B {
public:
    int m_b;
};

class C : public A, public B {
public:
    int m_c;
};

class D: public B, public A {
public:
    int m_d;
};

int main() {
    C c;
    printf("&c.m_a: %p, &c.m_b: %p, &c.m_c: %p\n", &c.m_a, &c.m_b, &c.m_c);
    D d;
    printf("&d.m_a: %p, &d.m_b: %p, &d.m_d: %p\n", &d.m_a, &d.m_b, &d.m_d);
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:32:56]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [14:33:19]
$ ./a.out
&c.m_a: 0x7ffd506d1cb0, &c.m_b: 0x7ffd506d1cb4, &c.m_c: 0x7ffd506d1cb8
&d.m_a: 0x7ffd506d1cc0, &d.m_b: 0x7ffd506d1cbc, &d.m_d: 0x7ffd506d1cc4
</script></code></pre>


结论：
**基类对象的排列顺序和继承时声明的顺序相同**

## 借助指针突破访问权限的限制
我们都知道，C++ 不允许通过对象来访问 private、protected 属性的成员变量；
不过 C++ 的这种限制仅仅是语法层面的，通过某种“蹩脚”的方法，我们能够突破访问权限的限制，访问到 private、protected 属性的成员变量，赋予我们这种“特异功能”的，正是强大而又灵活的指针（Pointer）；

**使用偏移**
在对象的内存模型中，成员变量和对象的开头位置会有一定的距离；和struct结构体中的数据成员偏移量（offset）是一个概念；

一旦知道了对象的起始地址，再加上偏移就能够求得成员变量的地址，知道了成员变量的地址和类型，也就能够轻而易举地知道它的值；

当通过对象指针访问成员变量时，编译器实际上也是使用这种方式来取得它的值；

在前面讲解对象的内存模型的时候，已经多次的演示了如何通过 指针 + 计算偏移量 来访问对象的 protected、private 成员；这里就不再重复演示了；

我们说 C++ 的成员访问权限仅仅是语法层面上的，是指访问权限仅对取成员运算符`.`和`->`起作用，而无法防止直接通过指针来访问；
本节的目的不是为了访问到 private、protected 属性的成员变量，这种“花拳绣腿”没有什么现实的意义，本节主要是让大家明白编译器内部的工作原理，以及指针的灵活运用；

## 虚继承和虚基类
`多继承（Multiple Inheritance）`是指从多个直接基类中产生派生类的能力，多继承的派生类继承了所有父类的成员；尽管概念上非常简单，但是多个基类的相互交织可能会带来错综复杂的设计问题，命名冲突就是不可回避的一个；

多继承时很容易产生命名冲突，即使我们很小心地将所有类中的成员变量和成员函数都命名为不同的名字，命名冲突依然有可能发生，比如典型的是`菱形继承`，如下图所示：
![C++ 菱形继承](/images/cpp-diamond.jpg)

类 A 派生出类 B 和类 C，类 D 继承自类 B 和类 C，这个时候类 A 中的成员变量和成员函数继承到类 D 中变成了两份，一份来自 A-->B-->D 这条路径，另一份来自 A-->C-->D 这条路径；

在一个派生类中保留间接基类的多份同名成员，虽然可以在不同的成员变量中分别存放不同的数据，但大多数情况下这是多余的：因为保留多份成员变量不仅占用较多的存储空间，还容易产生命名冲突；假如类 A 有一个成员变量 a，那么在类 D 中直接访问 a 就会产生歧义，编译器不知道它究竟来自 A -->B-->D 这条路径，还是来自 A-->C-->D 这条路径；

下面就是菱形继承的体现：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
protected:
    int m_a;
};

class B : public A {
protected:
    int m_b;
};

class C : public A {
protected:
    int m_c;
};

class D: public B, public C {
public:
//  void seta(int a) { m_a = a; } // 命名冲突
    void seta(int a) { B::m_a = a; } // 正确
//  void seta(int a) { C::m_a = a; } // 正确
    void setb(int b) { m_b = b; }
    void setc(int c) { m_c = c; }
    void setd(int d) { m_d = d; }
private:
    int m_d;
};

int main() {
    D d;
    return 0;
}
</script></code></pre>


**虚继承（Virtual Inheritance）**
为了解决多继承时的命名冲突和冗余数据问题，C++ 提出了虚继承，使得在派生类中只保留一份间接基类的成员；

在继承方式前面加上`virtual`关键字就是虚继承：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
protected:
    int m_a;
};

class B : virtual public A {
protected:
    int m_b;
};

class C : virtual public A {
protected:
    int m_c;
};

class D: public B, public C {
public:
    void seta(int a) { m_a = a; }
//  void seta(int a) { B::m_a = a; } // 正确
//  void seta(int a) { C::m_a = a; } // 正确
    void setb(int b) { m_b = b; }
    void setc(int c) { m_c = c; }
    void setd(int d) { m_d = d; }
private:
    int m_d;
};

int main() {
    D d;
    return 0;
}
</script></code></pre>


这段代码使用虚继承重新实现了上图所示的菱形继承，这样在派生类 D 中就只保留了一份成员变量 m_a，直接访问就不会再有歧义了；

**虚继承的目的是让某个类做出声明，承诺愿意共享它的基类**；其中，这个**被共享的基类**就称为`虚基类（Virtual Base Class）`；
本例中的 A 就是一个虚基类。在这种机制下，不论虚基类在继承体系中出现了多少次，在派生类中都只包含一份虚基类的成员；

**虚派生只影响从指定了虚基类的派生类中进一步派生出来的类，它不会影响派生类本身**

C++标准库中的 iostream 类就是一个虚继承的实际应用案例；iostream 从 istream 和 ostream 直接继承而来，而 istream 和 ostream 又都继承自一个共同的名为 base_ios 的类，是典型的菱形继承；此时 istream 和 ostream 必须采用虚继承，否则将导致 iostream 类中保留两份 base_ios 类的成员；

**虚继承的二义性问题**
因为在虚继承的最终派生类中只保留了一份虚基类的成员，所以该成员可以被直接访问，不会产生二义性；
但是这仅仅解决了虚基类中的成员的命名冲突，如果虚基类的派生类中有同名成员，那么还是无法避免命名冲突；

以上图的菱形继承为例，假设 B 定义了一个名为 x 的成员变量，当我们在 D 中直接访问 x 时，会有三种可能性：
- 如果 A 和 C 中都没有 x 的定义，那么 x 将被解析为 B 的成员，此时不存在二义性；
- 如果 A 中定义了 x，也不会有二义性，派生类的 x 比虚基类的 x 优先级更高；
- 如果 C 中定义了 x，那么直接访问 x 将产生二义性问题；需要指明使用 B 中的 x 还是 C 中的 x；

可以看到，使用多继承经常会出现二义性问题，必须十分小心；
上面的例子是简单的，如果继承的层次再多一些，关系更复杂一些，程序员就很容易陷人迷魂阵，程序的编写、调试和维护工作都会变得更加困难，因此我不提倡在程序中使用多继承，只有在比较简单和不易出现二义性的情况或实在必要时才使用多继承，能用单一继承解决的问题就不要使用多继承；也正是由于这个原因，C++ 之后的很多面向对象的编程语言，例如 Java、C#、PHP 等，都不支持多继承；

## 虚继承时的构造函数
**在虚继承中，虚基类是由最终的派生类初始化的**，换句话说，最终派生类的构造函数必须要调用虚基类的构造函数；
对最终的派生类来说，虚基类是间接基类，而不是直接基类；这跟普通继承不同，在普通继承中，派生类构造函数中只能调用直接基类的构造函数，不能调用间接基类的构造函数；

看下面的例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    A(int a);
protected:
    int m_a;
};

A::A(int a) : m_a(a) {}

class B : virtual public A {
public:
    B(int a, int b);
    void print() const;
protected:
    int m_b;
};

B::B(int a, int b) : A(a), m_b(b) {}
void B::print() const {
    printf("m_a: %d, m_b: %d\n", m_a, m_b);
}

class C : virtual public A {
public:
    C(int a, int c);
    void print() const;
protected:
    int m_c;
};

C::C(int a, int c) : A(a), m_c(c) {}
void C::print() const {
    printf("m_a: %d, m_c: %d\n", m_a, m_c);
}

class D : public B, public C {
public:
    D(int a, int b, int c, int d);
    void print() const;
private:
    int m_d;
};

D::D(int a, int b, int c, int d) : A(a), B(0, b), C(0, c), m_d(d) {}
void D::print() const {
    printf("m_a: %d, m_b: %d, m_c: %d, m_d: %d\n", m_a, m_b, m_c, m_d);
}

int main() {
    B(1, 2).print();

    C(1, 3).print();

    D(1, 2, 3, 4).print();
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:50:08]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [15:50:34]
$ ./a.out
m_a: 1, m_b: 2
m_a: 1, m_c: 3
m_a: 1, m_b: 2, m_c: 3, m_d: 4
</script></code></pre>


注意第 48 行的代码，虽然我们都给 B 和 C 的构造函数的第一个参数传递了 0，但是编译器会默认忽略，以`A(a)`为准；

另外需要关注的是**构造函数的执行顺序**，虚继承时构造函数的执行顺序与普通继承时不同：
在最终派生类的构造函数调用列表中，不管各个构造函数出现的顺序如何，编译器总是**先调用虚基类的构造函数**，**再按照最终派生类的继承的顺序调用其他的构造函数**；

## 虚继承下的内存模型
简单的面向对象，只有单继承或多继承的情况下，内存模型很好理解，编译器实现起来也容易，C++ 的效率和 C 的效率不相上下；一旦和 virtual 关键字扯上关系，使用到虚继承或虚函数，内存模型就变得混乱起来，各种编译器的实现也不一致，让人抓狂；

这是因为 C++ 标准仅对 C++ 的实现做了框架性的概述，并没有规定细节如何实现，所以不同厂商的编译器在具体实现方案上会有所差异；

本节我们只关注虚继承时的内存模式，有关虚函数的内容将在虚函数一节中讲解；
对于普通继承，基类子对象始终位于派生类对象的前面（也即基类成员变量始终在派生类成员变量的前面），而且不管继承层次有多深，它相对于派生类对象顶部的偏移量是固定的；

前面我们说过，编译器在知道对象首地址的情况下，通过计算偏移来存取成员变量；对于普通继承，基类成员变量的偏移是固定的，不会随着继承层级的增加而改变，存取起来非常方便；
而对于虚继承，恰恰和普通继承相反，大部分编译器会把基类成员变量放在派生类成员变量的后面，这样随着继承层级的增加，基类成员变量的偏移就会改变，就得通过其他方案来计算偏移量；

虚继承时的派生类对象被分成了两部分：
- 不带阴影的一部分偏移量固定，不会随着继承层次的增加而改变，称为固定部分；
- 带有阴影的一部分是虚基类的子对象，偏移量会随着继承层次的增加而改变，称为共享部分；

当要访问对象的成员变量时，需要知道对象的首地址和变量的偏移，对象的首地址很好获得，关键是变量的偏移。对于固定部分，偏移是不变的，很好计算；而对于共享部分，偏移会随着继承层次的增加而改变，这就需要设计一种方案，在偏移不断变化的过程中准确地计算偏移。各个编译器正是在设计这一方案时出现了分歧，不同的编译器设计了不同的方案来计算共享部分的偏移。

对于虚继承，将派生类分为固定部分和共享部分，并把共享部分放在最后，几乎所有的编译器都在这一点上达成了共识。主要的分歧就是如何计算共享部分的偏移，可谓是百花齐放，没有统一标准；

**cfront解决方案**
早期的 cfront 编译器会在派生类对象中安插一些指针，每个指针指向一个虚基类的子对象，要存取继承来的成员变量，可以使用指针间接完成；

这种方案的一个缺点是，随着虚继承层次的增加，访问顶层基类需要的间接转换会越来越多，效率越来越低；
这种方案另外的一个缺点是：当有多个虚基类时，派生类要为每个虚基类都安插一个指针，会增加对象的体积；

**VC解决方案**
cfront 的后来者 VC 尝试对上面的方案进行了改进，一定程度上弥补了它的不足：
VC引入了**虚基类表**，如果某个派生类有一个或多个虚基类，编译器就会在派生类对象中安插一个指针，指向虚基类表；虚基类表其实就是一个数组，数组中的元素存放的是各个虚基类的偏移；

虚继承表中保存的是所有虚基类（包括直接继承和间接继承到的）相对于当前对象的偏移，这样通过派生类指针访问虚基类的成员变量时，不管继承层次都多深，只需要一次间接转换就可以；
另外，这种方案还可以避免有多个虚基类时让派生类对象额外背负过多的指针；

## 向上转型（将派生类赋值给基类）
在 C/C++ 中经常会发生数据类型的转换，例如将 int 类型的数据赋值给 float 类型的变量时，编译器会先把 int 类型的数据转换为 float 类型再赋值；反过来，float 类型的数据在经过类型转换后也可以赋值给 int 类型的变量；

数据类型转换的前提是，编译器知道如何对数据进行取舍；

类其实也是一种数据类型，也可以发生数据类型转换，不过这种转换只有在基类和派生类之间才有意义，并且只能将派生类赋值给基类，包括将派生类对象赋值给基类对象、将派生类指针赋值给基类指针、将派生类引用赋值给基类引用，这在 C++ 中称为`向上转型（Upcasting）`。相应地，将基类赋值给派生类称为`向下转型（Downcasting）`；

向上转型非常安全，可以由编译器自动完成；向下转型有风险，需要程序员手动干预。本节只介绍向上转型，向下转型将在后续章节介绍；

**将派生类对象赋值给基类对象**
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    A(int a);
    void print() const;
protected:
    int m_a;
};

A::A(int a) : m_a(a) {}
void A::print() const {
    printf("class A: m_a = %d\n", m_a);
}

class B : public A {
public:
    B(int a, int b);
    void print() const;
private:
    int m_b;
};

B::B(int a, int b) : A(a), m_b(b) {}
void B::print() const {
    printf("class B: m_a = %d, m_b = %d\n", m_a, m_b);
}

int main() {
    A a(1);
    B b(11, 22);
    a.print();
    b.print();

    printf("----------------\n");

    a = b;
    a.print();
    b.print();

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:32:08] C:130
$ g++ a.cpp

# root @ arch in ~/work on git:master x [16:32:10]
$ ./a.out
class A: m_a = 1
class B: m_a = 11, m_b = 22
----------------
class A: m_a = 11
class B: m_a = 11, m_b = 22
</script></code></pre>


赋值的本质是将现有的数据写入已分配好的内存中，对象的内存只包含了成员变量，所以对象之间的赋值是成员变量的赋值，成员函数不存在赋值问题。运行结果也有力地证明了这一点，虽然有`a = b;`这样的赋值过程，但是 a.print() 始终调用的都是 A 类的 print() 函数。换句话说，对象之间的赋值不会影响成员函数，也不会影响 this 指针；

将派生类对象赋值给基类对象时，会舍弃派生类新增的成员，也就是“大材小用”；

这种转换关系是不可逆的，只能用派生类对象给基类对象赋值，而不能用基类对象给派生类对象赋值。理由很简单，基类不包含派生类的成员变量，无法对派生类的成员变量赋值。同理，同一基类的不同派生类对象之间也不能赋值。

要理解这个问题，还得从赋值的本质入手。赋值实际上是向内存填充数据，当数据较多时很好处理，舍弃即可；本例中将 b 赋值给 a 时（执行`a = b;`语句），成员 m_b 是多余的，会被直接丢掉，所以不会发生赋值错误。但当数据较少时，问题就很棘手，编译器不知道如何填充剩下的内存；如果本例中有`b = a;`这样的语句，编译器就不知道该如何给变量 m_b 赋值，所以会发生错误；

**将派生类指针赋值给基类指针**
除了可以将派生类对象赋值给基类对象（对象变量之间的赋值），还可以将派生类指针赋值给基类指针（对象指针之间的赋值）；

我们先来看一个多继承的例子：基类 A 拥有成员 m_a；中间派生类 B 继承于 A，并添加了新成员 m_b；基类 C 拥有成员 m_c；最终派生类 D 继承 B 和 C；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    A(int a);
    void print() const;
protected:
    int m_a;
};

A::A(int a) : m_a(a) {}
void A::print() const {
    printf("class A: m_a = %d\n", m_a);
}

class B : public A {
public:
    B(int a, int b);
    void print() const;
protected:
    int m_b;
};

B::B(int a, int b) : A(a), m_b(b) {}
void B::print() const {
    printf("class B: m_a = %d, m_b = %d\n", m_a, m_b);
}

class C {
public:
    C(int c);
    void print() const;
protected:
    int m_c;
};

C::C(int c) : m_c(c) {}
void C::print() const {
    printf("class C: m_c = %d\n", m_c);
}

class D : public B, public C {
public:
    D(int a, int b, int c, int d);
    void print() const;
private:
    int m_d;
};

D::D(int a, int b, int c, int d) : B(a, b), C(c), m_d(d) {}
void D::print() const {
    printf("class D: m_a = %d, m_b = %d, m_c = %d, m_d = %d\n", m_a, m_b, m_c, m_d);
}

int main() {
    A *pa = new A(1);
    pa -> print();

    B *pb = new B(11, 22);
    pb -> print();

    C *pc = new C(3);
    pc -> print();

    D *pd = new D(110, 114, 119, 120);
    pd -> print();

    printf("--------------------------\n");

    pa = pd;
    pa -> print();

    pb = pd;
    pb -> print();

    pc = pd;
    pc -> print();

    printf("---------------------------\n");

    printf("pa = %p\npb = %p\npc = %p\npd = %p\n", pa, pb, pc, pd);

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:55:12]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [16:55:25]
$ ./a.out
class A: m_a = 1
class B: m_a = 11, m_b = 22
class C: m_c = 3
class D: m_a = 110, m_b = 114, m_c = 119, m_d = 120
--------------------------
class A: m_a = 110
class B: m_a = 110, m_b = 114
class C: m_c = 119
---------------------------
pa = 0xb61aee4090
pb = 0xb61aee4090
pc = 0xb61aee4098
pd = 0xb61aee4090
</script></code></pre>


本例中定义了多个对象指针，并尝试将派生类指针赋值给基类指针。与对象变量之间的赋值不同的是，对象指针之间的赋值并没有拷贝对象的成员，也没有修改对象本身的数据，仅仅是改变了指针的指向；

1) **通过基类指针访问派生类的成员**
将派生类指针 pd 赋值给了基类指针 pa，从运行结果可以看出，调用 display() 函数时虽然使用了派生类的成员变量，但是 display() 函数本身却是基类的；也就是说，将派生类指针赋值给基类指针时，通过基类指针只能使用派生类的成员变量，但不能使用派生类的成员函数，pb、pc 也是一样的情况；

这是因为，调用哪个类的函数不是由指针指向的数据为判断标准，而是决定于指针的类型，如果指针的类型是`A *`，那么就调用类 A 的函数，如果是`D *`，就调用类 D 的函数；

概括起来说就是：编译器通过指针来访问成员变量，指针指向哪个对象就使用哪个对象的数据；编译器通过指针的类型来访问成员函数，指针属于哪个类的类型就使用哪个类的函数。

2) **赋值后值不一致的情况**
本例中我们将最终派生类的指针 pd 分别赋值给了基类指针 pa、pb、pc，按理说它们的值应该相等，都指向同一块内存，但是运行结果却有力地反驳了这种推论，只有 pa、pb、pd 三个指针的值相等，pc 的值比它们都大。也就是说，执行`pc = pd;`语句后，pc 和 pd 的值并不相等；

这是因为在赋值前，编译器会进行某些处理，就如同将 float 类型的 3.14 赋值给 int 类型的变量时，编译器会直接舍弃小数点后的值，最终变为 3；对象指针之间的赋值也是这个道理；

**将派生类引用赋值给基类引用**
引用在本质上是通过指针的方式实现的，既然基类的指针可以指向派生类的对象，那么我们就有理由推断：基类的引用也可以指向派生类的对象，并且它的表现和指针是类似的；

> 
最后需要注意的是，向上转型后通过基类的对象、指针、引用只能访问从基类继承过去的成员（包括成员变量和成员函数），不能访问派生类新增的成员；

## 将派生类指针赋值给基类指针时到底发生了什么
通过上节最后一个例子我们发现，将派生类的指针赋值给基类的指针后，它们的值有可能相等，也有可能不相等；

我们通常认为，赋值就是将一个变量的值交给另外一个变量，这种想法虽然没错，但是有一点要注意，就是赋值以前编译器可能会对现有的值进行处理；

例如将 double 类型的值赋给 int 类型的变量，编译器会直接抹掉小数部分，导致赋值运算符两边变量的值不相等；

将派生类的指针赋值给基类的指针时也是类似的道理，编译器也可能会在赋值前进行处理；

首先要明确的一点是，**对象的指针必须要指向对象的起始位置**；
对于 A 类和 B 类来说，它们的子对象的起始地址和 D 类对象一样，所以将 pd 赋值给 pa、pb 时不需要做任何调整，直接传递现有的值即可；
而 C 类子对象距离 D 类对象的开头有一定的偏移，将 pd 赋值给 pa 时要加上这个偏移，这样 pc 才能指向 C 类子对象的起始位置；
也就是说，执行`pc = pd;`语句时编译器对 pd 的值进行了调整，才导致 pc、pd 的值不同；
