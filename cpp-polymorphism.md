---
title: C++ 多态与虚函数
date: 2017-08-26 07:18:00
categories:
- cpp
tags:
- cpp
keywords: C++ 多态性 虚函数 RTTI运行时类型识别
---

> 
C++ 多态与虚函数，`多态（Polymorphism）`是面向对象（Object-Oriented，OO）思想"三大特征"之一，其余两个分别是`封装（Encapsulation）`和`继承（Inheritance）`--可见多态的重要性；或者说，不懂得什么是多态就不能说懂得面向对象；
**多态是一种机制、一种能力**，而非某个关键字；它在类的继承中得以实现，在类的方法调用中得以体现；

<!-- more -->

## 多态的概念
在之前的继承与派生中，我们讲到这么个例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class People {
public:
    People(const char *name, int age);
    void print() const;
protected:
    const char *m_name;
    int m_age;
};

People::People(const char *name, int age) : m_name(name), m_age(age) {}
void People::print() const {
    printf("name: %s, age: %d\n", m_name, m_age);
}

class Student : public People {
public:
    Student(const char *name, int age, float score);
    void print() const;
private:
    float m_score;
};

Student::Student(const char *name, int age, float score) : People(name, age), m_score(score) {}
void Student::print() const {
    printf("name: %s, age: %d, score: %g\n", m_name, m_age, m_score);
}

int main() {
    People *p = nullptr;
    p = new People("ZhangSan", 25);
    p -> print();
    delete p;

    p = new Student("LiSi", 14, 87.5f);
    p -> print();
    delete p;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [8:16:25]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [8:16:44]
$ ./a.out
name: ZhangSan, age: 25
name: LiSi, age: 14
</script></code></pre>


我们直观上认为，如果指针指向了派生类对象，那么就应该使用派生类的成员变量和成员函数，这符合人们的思维习惯；
但是本例的运行结果却告诉我们，当基类指针 p 指向派生类 Student 的对象时，虽然使用了 Student 的成员变量，但是却没有使用它的成员函数，导致输出结果不伦不类，不符合我们的预期；

换句话说，**通过基类指针只能访问派生类的成员变量，但是不能访问派生类的成员函数**；

为了消除这种尴尬，让基类指针能够访问派生类的成员函数，C++ 增加了`虚函数（Virtual Function）`；使用虚函数非常简单，只需要在函数声明前面增加`virtual`关键字；

更改上面的代码，将 print() 声明为虚函数：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class People {
public:
    People(const char *name, int age);
    virtual ~People() {}
    virtual void print() const;
protected:
    const char *m_name;
    int m_age;
};

class Student : public People {
public:
    Student(const char *name, int age, float score);
    virtual ~Student() override {}
    virtual void print() const override;
private:
    float m_score;
};

People::People(const char *name, int age) : m_name(name), m_age(age) {}
void People::print() const {
    printf("name: %s, age: %d\n", m_name, m_age);
}

Student::Student(const char *name, int age, float score) : People(name, age), m_score(score) {}
void Student::print() const {
    printf("name: %s, age: %d, score: %g\n", m_name, m_age, m_score);
}

int main() {
    People *p = nullptr;

    p = new People("ZhangSan", 25);
    p -> print();
    delete p;

    p = new Student("LiSi", 14, 87.5);
    p -> print();
    delete p;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [8:47:47]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [8:48:13]
$ ./a.out
name: ZhangSan, age: 25
name: LiSi, age: 14, score: 87.5
</script></code></pre>


> 
`override`是 C++11 中新增的关键字，override确保在派生类中声明的重载函数跟基类的虚函数有相同的签名；

有了`虚函数`，基类指针指向基类对象时就使用基类的成员（包括成员函数和成员变量），指向派生类对象时就使用派生类的成员；
换句话说，基类指针可以按照基类的方式来做事，也可以按照派生类的方式来做事，它有多种形态，或者说有多种表现方式，我们将这种现象称为`多态（Polymorphism）`；

多态是面向对象编程的主要特征之一，C++中虚函数的唯一用处就是构成多态；

C++提供多态的目的是：可以通过基类指针对所有派生类（包括直接派生和间接派生）的成员变量和成员函数进行“全方位”的访问，尤其是成员函数；如果没有多态，我们只能访问成员变量；

前面我们说过，通过指针调用普通的成员函数时会根据指针的类型（通过哪个类定义的指针）来判断调用哪个类的成员函数，但是通过本节的分析可以发现，这种说法并不适用于虚函数，虚函数是根据指针的指向来调用的，指针指向哪个类的对象就调用哪个类的虚函数；

但是话又说回来，对象的内存模型是非常干净的，没有包含任何成员函数的信息，编译器究竟是根据什么找到了成员函数呢？我们将在「C++虚函数表vtable以及动态绑定」中讲解说明；

**借助引用也可以实现多态**
引用在本质上是通过指针的方式实现的，既然借助指针可以实现多态，那么我们就有理由推断：借助引用也可以实现多态；
修改上面的 main() 函数：
<pre><code class="language-cpp line-numbers"><script type="text/plain">int main() {
    People p("ZhangSan", 25);
    Student s("LiSi", 14, 87.5);

    People &rp = p;
    People &rs = s;
    rp.print();
    rs.print();

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [8:58:10]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [8:58:42]
$ ./a.out
name: ZhangSan, age: 25
name: LiSi, age: 14, score: 87.5
</script></code></pre>


不过引用不像指针灵活，指针可以随时改变指向，而引用只能指代固定的对象，在多态性方面缺乏表现力，所以以后我们再**谈及多态时一般是说指针**；本例的主要目的是让读者知道，除了指针，引用也可以实现多态；

## virtual虚函数
虚函数对于多态具有决定性的作用，有虚函数才能构成多态，这节我们来重点说一下虚函数的注意事项：
1) **只需要在虚函数的声明处加上 virtual 关键字，函数定义处不加**
2) **可以只将基类中的函数声明为虚函数，当派生类中出现参数列表相同的同名函数时，自动成为虚函数**
3) 当在基类中定义了虚函数时，如果派生类没有定义新的函数来override此函数，那么将使用基类的虚函数
4) **构造函数不能是虚函数**；对于基类的构造函数，它仅仅是在派生类构造函数中被调用，这种机制不同于继承；也就是说，派生类不继承基类的构造函数，将构造函数声明为虚函数没有什么意义
5) **析构函数有必要声明为虚函数**

**overload、override区别**
**函数重载（overload）**
同一作用域中，函数有同样的名称，但是参数列表不相同的情形，这样的同名不同参数的函数之间，互相称之为`重载函数`；

基本条件：
- `函数名`必须`相同`；
- `函数参数`必须`不相同`，可以是参数类型或者参数个数不同；
- `函数返回值`可以`相同`，也可以`不相同`；

注意：
- **只能通过`不同的参数列表`进行重载**，例如：`不同的参数类型`，`不同的参数个数`，`不同的参数顺序`；
- **不能通过`访问权限`、`返回类型`、`抛出的异常`进行重载**；
- 重载的函数应该在**相同的作用域**下；

**函数重写（override）**
子类重新定义父类中有`相同名称`和`参数`的`虚函数`，主要在**继承关系**中出现；

基本条件：
- 必须`存在继承关系`；
- 重写的函数和被重写的函数必须为`virtual`函数，分别位于`基类`和`派生类中`；
- 重写的函数和被重写的函数`函数名`和`函数参数`必须`一致`；
- 重写的函数和被重写的函数`返回值相同`，或者都返回`指针/引用`，重写函数返回的指针/引用类型必须和被重写函数返回的`类型一致`，或者为`其子类型`；

注意：
- 重写的函数所**抛出的异常**必须和被重写的函数所抛出的异常**一致**，或者是**其子类**；
- 重写的函数的**访问修饰符可以不同于被重写的函数**，如基类的virtual函数的修饰符为private，派生类改为public或protected也是可以的；
- `static静态方法`不能被重写，也就是**static和virtual不能同时使用**，**构造函数也不能为virtual函数**；
- 重写的函数可以带virtual关键字，也可以不带；

**什么情况下声明虚函数**
首先看成员函数所在的类是否会作为基类；然后看成员函数在类的继承后有无可能被更改功能；
如果希望更改其功能的，一般应该将它声明为虚函数；如果成员函数在类被继承后功能不需修改，或派生类用不到该函数，则不要把它声明为虚函数；

我们来看这个例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
    public:
        virtual void func() { printf("A::func()\n"); }
        virtual void func(int) { printf("A::func(int)\n"); }
        virtual void func(char) { printf("A::func(char)\n"); }
};

class B : public A {
    public:
        virtual void func() { printf("B::func()\n"); }
        virtual void func(int) { printf("B::func(int)\n"); }
        virtual void func(float) { printf("B::func(float)\n"); }
};

class C : public B {
    public:
        virtual void func() { printf("C::func()\n"); }
        virtual void func(double) { printf("C::func(double)\n"); }
        virtual void func(float) { printf("C::func(float)\n"); }
};

int main() {
    A *pa = nullptr;
    B *pb = nullptr;

    pa = new B;
    pa -> func(); // [override] A::func() >> B::func()
    pa -> func(1); // [override] A::func(int) >> B::func(int)
//  pa -> func(1.1f); // 编译报错
    pa -> func('A'); // A::func(char)

    pa = new C;
    pa -> func(); // [override] A::func() >> B::func() >> C::func()
    pa -> func(1); // [override] A::func(int) >> B::func(int)
    pa -> func('A'); // A::func(char)

    pb = new C;
    pb -> func(); // [override] B::func() >> C::func()
    pb -> func(1.1f); // [override] B::func(float) >> C::func(float)

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:11:34]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [10:12:02]
$ ./a.out
B::func()
B::func(int)
A::func(char)
C::func()
B::func(int)
A::func(char)
C::func()
C::func(float)
</script></code></pre>


## 虚析构函数的必要性
上节我们讲到，构造函数不能是虚函数，因为派生类不能继承基类的构造函数，将构造函数声明为虚函数没有意义；
这是原因之一，另外还有一个原因：C++中的构造函数用于在创建对象时进行初始化工作，在执行构造函数之前对象尚未创建完成，虚函数表尚不存在，也没有指向虚函数表的指针，所以此时无法查询虚函数表，也就不知道要调用哪一个构造函数；

析构函数用于在销毁对象时进行清理工作，可以声明为虚函数，而且有时候必须要声明为虚函数；

为了说明虚析构函数的必要性，我们来看这个例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    A() { printf("A()\n"); }
    ~A() { printf("~A()\n"); }
};

class B : public A {
public:
    B() { printf("B()\n"); }
    ~B() { printf("~B()\n"); }
};

int main() {
    A *p = nullptr;
    p = new B;
    delete p;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:17:33]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [10:17:51]
$ ./a.out
A()
B()
~A()
</script></code></pre>


我们先来看构造函数：
在 19 行中，`new B;`将执行类 B 的构造函数，在类 B 的构造函数中我们没有显示调用 A 的构造函数，但是编译器会默认帮我们调用 A 的无参构造函数，进而也就调用了 A 的构造函数，没什么问题；

再看看析构函数：
在 20 行中，`delete p;`将执行类 A 的析构函数，因为 p 的指针类型为`A *`，也就调用 A 的析构函数，并不会调用B的析构函数；

在本例中，这并无什么大碍，因为没有申请的动态内存，打开的文件、socket等；并不需要在析构函数中清理资源；
但是如果在派生类中申请了动态内存、打开了文件、socket，那么这些资源将会一直占用，直到程序结束系统才会回收；

所以，很有必要将基类的析构函数声明为虚函数，不管其派生类有没有需要关闭、释放的资源，这是一个很好的编程习惯；

**将基类的析构函数声明为虚函数后，派生类的析构函数也会自动成为虚函数**

现在我们将基类的析构函数改为virtual虚函数：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
public:
    A() { printf("A()\n"); }
    virtual ~A() { printf("~A()\n"); }
};

class B : public A {
public:
    B() { printf("B()\n"); }
    ~B() { printf("~B()\n"); }
};

int main() {
    A *p = nullptr;
    p = new B;
    delete p;
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:29:51]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [10:29:54]
$ ./a.out
A()
B()
~B()
~A()
</script></code></pre>


## 纯虚函数和抽象类
在C++中，可以将虚函数声明为`纯虚函数`，语法格式为：`virtual 返回值类型 函数名(函数参数) = 0;`
**纯虚函数没有函数体，只有函数声明**，在虚函数声明的结尾加上`=0`，表明此函数为纯虚函数；

最后的`=0`并不表示函数返回值为0，它只起形式上的作用，告诉编译系统“这是纯虚函数”；

**包含纯虚函数的类称为`抽象类（Abstract Class）`**；
之所以说它抽象，是因为它**无法实例化**，也就是无法创建对象；
原因很明显，纯虚函数没有函数体，不是完整的函数，无法调用，也无法为其分配内存空间；

**抽象类通常是作为基类，让派生类去实现纯虚函数；派生类必须实现纯虚函数才能被实例化**

**关于纯虚函数的几点说明**：
1) 一个纯虚函数就可以使类成为抽象基类，但是抽象基类中除了包含纯虚函数外，还可以包含其它的成员函数（虚函数或普通函数）和成员变量；

2) 只有类中的虚函数才能被声明为纯虚函数，普通成员函数和顶层函数均不能声明为纯虚函数；

例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Line {
public:
    Line(float len);
    virtual ~Line() {}
    virtual float area() const = 0;
    virtual float volume() const = 0;
protected:
    float m_len;
};

Line::Line(float len) : m_len(len) {}

class Rectangle : public Line {
public:
    Rectangle(float len, float width);
    virtual float area() const override;
protected:
    float m_width;
};

Rectangle::Rectangle(float len, float width) : Line(len), m_width(width) {}
float Rectangle::area() const { return m_len * m_width; }

class Cuboid : public Rectangle {
public:
    Cuboid(float len, float width, float height);
    virtual float area() const override;
    virtual float volume() const override;
protected:
    float m_height;
};

Cuboid::Cuboid(float len, float width, float height) : Rectangle(len, width), m_height(height) {}
float Cuboid::area() const { return 2 * (m_len*m_width + m_len*m_height + m_width*m_height); }
float Cuboid::volume() const { return m_len * m_width * m_height; }

class Cube : public Cuboid {
public:
    Cube(float len);
    virtual float area() const override;
    virtual float volume() const override;
};

Cube::Cube(float len) : Cuboid(len, len, len) {}
float Cube::area() const { return 6 * m_len*m_len; }
float Cube::volume() const { return m_len * m_len * m_len; }

int main() {
    Line *p = nullptr;

    p = new Cuboid(3, 4, 5);
    printf("Cuboid[3, 4, 5]: area = %g, volume = %g\n", p->area(), p->volume());
    delete p;

    p = new Cube(5);
    printf("Cube[5]: area = %g, volume = %g\n", p->area(), p->volume());
    delete p;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [11:05:43]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [11:06:02]
$ ./a.out
Cuboid[3, 4, 5]: area = 94, volume = 60
Cube[5]: area = 150, volume = 125
</script></code></pre>


本例中定义了四个类，它们的继承关系为：Line --> Rectangle --> Cuboid --> Cube；

Line 是一个抽象类，也是最顶层的基类，在 Line 类中定义了两个纯虚函数 area() 和 volume()；

在 Rectangle 类中，实现了 area() 函数；所谓实现，就是定义了纯虚函数的函数体；但这时 Rectangle 仍不能被实例化，因为它没有实现继承来的 volume() 函数，volume() 仍然是纯虚函数，所以 Rectangle 也仍然是抽象类；

直到 Cuboid 类，才实现了 volume() 函数，才是一个完整的类，才可以被实例化；

可以发现，Line 类表示“线”，没有面积和体积，但它仍然定义了 area() 和 volume() 两个纯虚函数；这样的用意很明显：Line 类不需要被实例化，但是它为派生类提供了“约束条件”，派生类必须要实现这两个函数，完成计算面积和体积的功能，否则就不能实例化；

在实际开发中，你可以定义一个抽象基类，只完成部分功能，未完成的功能交给派生类去实现（谁派生谁实现）；这部分未完成的功能，往往是基类不需要的，或者在基类中无法实现的；虽然抽象基类没有完成，但是却强制要求派生类完成，这就是抽象基类的“霸王条款”；

抽象基类除了约束派生类的功能，还可以实现`多态`；指针 p 的类型是 Line，但是它却可以访问派生类中的 area() 和 volume() 函数，正是由于在 Line 类中将这两个函数定义为纯虚函数；如果不这样做，后面的代码都是错误的；我想，这或许才是C++提供纯虚函数的主要目的；

## 虚函数表，多态的实现机制
前面我们一再强调，当通过指针访问类的成员函数时：
- 如果该函数是`非虚函数`，那么编译器会根据指针的类型找到该函数；也就是说，指针是哪个类的类型就调用哪个类的函数；
- 如果该函数是`虚函数`，并且派生类有同名的函数`override`它，那么编译器会根据指针的指向找到该函数；也就是说，指针指向的对象属于哪个类就调用哪个类的函数，这就是多态；

编译器之所以能通过指针指向的对象找到虚函数，是因为在**创建对象时**额外地增加了`虚函数表`；
**如果一个类包含了虚函数，那么在创建该类的对象时就会额外地增加一个数组，数组中的每一个元素都是虚函数的入口地址**；
不过数组和对象是分开存储的，为了将对象和数组关联起来，编译器还要在对象中安插一个指针，指向数组的起始位置；
这里的数组就是`虚函数表（Virtual function table）`，简写为`vtable`；

在对象的**开头位置**有一个指针`vfptr`，指向`虚函数表`，并且这个指针**始终位于对象的开头位置**；

仔细观察虚函数表，可以发现基类的虚函数在`vtable`中的索引（下标）是固定的，不会随着继承层次的增加而改变，派生类新增的虚函数放在`vtable`的最后；
如果派生类有同名的虚函数override了基类的虚函数，那么将使用派生类的虚函数替换基类的虚函数，这样具有override关系的虚函数在`vtable`中只会出现一次；

当通过指针调用虚函数时，先根据指针找到`vfptr`，再根据`vfptr`找到虚函数的入口地址；对于不同的虚函数，仅仅改变索引（下标）即可；

以上是针对单继承的说明；当存在多继承时，虚函数表的结构就会变得复杂，尤其是有虚继承时，还会增加虚基类表，更加让人抓狂；

## typeid运算符
`typeid`运算符用来获取一个表达式的`类型信息`；类型信息对于编程语言非常重要，它描述了数据的各种属性：
- 对于`基本类型`（int、float 等C++内置类型）的数据，类型信息所包含的内容比较简单，主要是指`数据的类型`；
- 对于`类类型`的数据（也就是对象），类型信息是指对象`所属的类`、`所包含的成员`、`所在的继承关系`等；

类型信息是创建数据的模板，数据占用多大内存、能进行什么样的操作、该如何操作等，这些都由它的类型信息决定；

使用`typeid`需要引入头文件`<typeinfo>`；`typeid`的操作对象既可以是`表达式`，也可以是`数据类型`；

typeid 会把获取到的类型信息保存到一个`type_info`类型的`对象`里面，并返回该对象的`常引用`；当需要具体的类型信息时，可以通过成员函数来提取；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>
#include <typeinfo>

using namespace std;

struct S {};
union U {};
class C {};

int main() {
    int n = 10;
    const type_info &n_info = typeid(n);
    printf("name: %s, hash_code: %lu\n", n_info.name(), n_info.hash_code());

    const type_info &d_info = typeid(3.14);
    printf("name: %s, hash_code: %lu\n", d_info.name(), d_info.hash_code());

    const type_info &s_info = typeid("www.zfl9.com");
    printf("name: %s, hash_code: %lu\n", s_info.name(), s_info.hash_code());

    S s;
    const type_info &S_info = typeid(s);
    printf("name: %s, hash_code: %lu\n", S_info.name(), S_info.hash_code());

    U u;
    const type_info &U_info = typeid(u);
    printf("name: %s, hash_code: %lu\n", U_info.name(), U_info.hash_code());

    C c;
    const type_info &C_info = typeid(c);
    printf("name: %s, hash_code: %lu\n", C_info.name(), C_info.hash_code());
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [11:32:03]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [11:32:20]
$ ./a.out
name: i, hash_code: 6253375586064260614
name: d, hash_code: 14494284460613645429
name: A13_c, hash_code: 12263922133773362029
name: 1S, hash_code: 8719536160452096907
name: 1U, hash_code: 9261050694815014181
name: 1C, hash_code: 17357974150197383590
</script></code></pre>


本例中还用到了 type_info 类的几个成员函数，下面是对它们的介绍：
- `name()`用来返回类型的名称；
- `hash_code()`用来返回当前类型对应的 hash 值；在不同的编译器下可能会有不同的整数，但它们都能唯一地标识某个类型；

遗憾的是，C++ 标准只对 type_info 类做了很有限的规定，不仅成员函数少，功能弱，而且各个平台的实现不一致；
C++ 标准规定，type_info 类至少要有如下所示的 4 个 public 属性的成员函数，其他的扩展函数编译器开发者可以自由发挥，不做限制：

1) 原型：`const char *name() const;`
返回一个能表示类型名称的字符串；但是C++标准并没有规定这个字符串是什么形式

2) 原型：`bool before(const type_info &rhs) const;`
判断一个类型是否位于另一个类型的前面，rhs 参数是一个 type_info 对象的引用；但是C++标准并没有规定类型的排列顺序，不同的编译器有不同的排列规则，程序员也可以自定义；要特别注意的是，这个排列顺序和继承顺序没有关系，基类并不一定位于派生类的前面；

3) 原型：`bool operator==(const type_info &rhs) const;`
重载运算符“==”，判断两个类型是否相同，rhs 参数是一个 type_info 对象的引用；

4) 原型：`bool operator!=(const type_info &rhs) const;`
重载运算符“!=”，判断两个类型是否不同，rhs 参数是一个 type_info 对象的引用；

C++能获取到的类型信息非常有限，也没有统一的标准，如同“鸡肋”一般，大部分情况下我们只是使用重载过的“==”运算符来判断两个类型是否相同；

typeid 返回 type_info 对象的引用，一个类型不管使用了多少次，编译器都只为它创建一个对象，所有 typeid 都返回这个对象的引用；

需要提醒的是，为了减小编译后文件的体积，编译器不会为所有的类型创建 type_info 对象，只会为使用了 typeid 运算符的类型创建；不过有一种特殊情况，就是带虚函数的类（包括继承来的），不管有没有使用 typeid 运算符，编译器都会为带虚函数的类创建 type_info 对象，我们将在C++ RTTI机制（运行时类型识别）中展开讲解；

**普通类、存在virtual虚函数的类之间的区别**
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>
#include <typeinfo>

using namespace std;

class A {
public:
    virtual ~A() {}
};

class B : public A {};

class C {};
class D : public C {};

int main() {
    A *pa = nullptr;
    B *pb = nullptr;
    C *pc = nullptr;
    D *pd = nullptr;
    pa = new B;
    pb = new B;
    pc = new D;
    pd = new D;
    bool flg1 = typeid(pa) == typeid(pb);
    bool flg2 = typeid(*pa) == typeid(*pb);
    bool flg3 = typeid(pc) == typeid(pd);
    bool flg4 = typeid(*pc) == typeid(*pd);
    printf("flg1: %d, flg2: %d, flg3: %d, flg4: %d\n", flg1, flg2, flg3, flg4);
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:49:44] C:130
$ g++ a.cpp

# root @ arch in ~/work on git:master x [12:49:45]
$ ./a.out
flg1: 0, flg2: 1, flg3: 0, flg4: 0
</script></code></pre>


## RTTI机制（运行时类型识别）
一般情况下，在编译期间就能确定一个表达式的类型，但是当存在多态时，有些表达式的类型在编译期间就无法确定了，必须等到程序运行后根据实际的环境来确定；下面的例子演示了这种情况：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>
#include <typeinfo>

using namespace std;

class A {
public:
    virtual ~A() {}
};

class B : public A {};

int main() {
    int n;
    A *p = nullptr;
    scanf("%d", &n);
    if (n > 0) {
        p = new A;
    } else {
        p = new B;
    }
    printf("typeid(p): %s, typeid(*p): %s\n", typeid(p).name(), typeid(*p).name());
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:45:14]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [13:46:11]
$ ./a.out
1
typeid(p): P1A, typeid(*p): 1A

# root @ arch in ~/work on git:master x [13:46:12]
$ ./a.out
-1
typeid(p): P1A, typeid(*p): 1B
</script></code></pre>


因为基类的析构函数被声明为virtual虚函数，构成了多态；p 是基类的指针，可以指向基类对象，也可以指向派生类对象；`*p`表示 p 指向的对象；

从代码中可以看出，用户输入的数字不同，`*p`表示的对象就不同，typeid 获取到的类型也就不同，编译器在编译期间无法预估用户的输入，所以无法确定`*p`的类型，必须等到程序真的运行了、用户输入完毕了才能确定`*p`的类型；

根据前面讲过的知识，C++ 的对象内存模型主要包含了以下几个方面的内容：
- 如果`没有虚函数也没有虚继承`，那么对象内存模型中只有成员变量；
- 如果类包含了`虚继承`，那么会额外添加一个`虚基类表`，并在对象内存中插入一个指针，指向这个虚基类表；
- 如果类包含了`虚函数`，那么会额外添加一个`虚函数表`，并在对象内存中插入一个指针，指向这个虚函数表；

现在我们要补充的一点是，如果类包含了虚函数，那么该类的对象内存中还会额外增加`类型信息`，也即 `type_info` 对象；

编译器会在虚函数表`vftable`的开头插入一个指针，指向当前类对应的`type_info`对象；当程序在运行阶段获取类型信息时，可以通过对象指针 p 找到虚函数表指针`vfptr`，再通过`vfptr`找到`type_info`对象的指针，进而取得类型信息；

转换过程：`**(p->vfptr - 1)`
程序运行后，不管 p 指向 A 类对象还是指向 B 类对象，只要执行这条语句就可以取得 type_info 对象；

编译器在编译阶段无法确定 p 指向哪个对象，也就无法获取`*p`的类型信息，但是编译器可以在编译阶段做好各种准备，这样程序在运行后可以借助这些准备好的数据来获取类型信息；这些准备包括：
- 创建 type_info 对象，并在 vftable 的开头插入一个指针，指向 type_info 对象；
- 将获取类型信息的操作转换成类似`**(p->vfptr - 1)`这样的语句；

这样做虽然会占用更多的内存，效率也降低了，但这是没办法的事情，编译器实在是无能为力了；

这种在程序运行后确定对象的类型信息的机制称为**运行时类型识别（Run-Time Type Identification，RTTI）**；
在 C++ 中，只有类中**包含了虚函数时才会启用 RTTI 机制**，其他所有情况都可以在编译阶段确定类型信息；

`多态（Polymorphism）`是面向对象编程的一个重要特征，它极大地增加了程序的灵活性，C++、C#、Java 等“正统的”面向对象编程语言都支持多态；
但是支持多态的代价也是很大的，有些信息在编译阶段无法确定下来，必须提前做好充足的准备，让程序运行后再执行一段代码获取，这会消耗更多的内存和 CPU 资源；
