---
title: C++ 类和对象
date: 2017-08-24 12:00:00
categories:
- cpp
tags:
- cpp
keywords: cpp class C++ 类和对象
---

> 
C++ 类和对象，类和对象是 C++ 的重要特性，它们使得 C++ 成为面向对象的编程语言

<!-- more -->

## 类的定义和对象的创建
类是创建对象的模板，一个类可以创建多个对象，每个对象都是类类型的一个变量；创建对象的过程也叫类的实例化。每个对象都是类的一个具体实例（Instance），拥有类的成员变量和成员函数。

有些教程将类的成员变量称为类的属性（Property），将类的成员函数称为类的方法（Method）。在面向对象的编程语言中，经常把函数（Function）称为方法（Method）。

与结构体一样，类只是一种复杂数据类型的声明，不占用内存空间。而对象是类这种数据类型的一个变量，或者说是通过类这种数据类型创建出来的一份实实在在的数据，所以占用内存空间；

**类的定义**
类是用户自定义的类型，如果程序中要用到类，必须提前说明，或者使用已存在的类（别人写好的类、标准库中的类等），C++语法本身并不提供现成的类的名称、结构和内容；

定义一个简单的Student类：
<pre><code class="language-cpp line-numbers"><script type="text/plain">class Student {
    public:
        // 成员变量
        char *name;
        int age;
        float score;

        // 成员函数
        void print() {
            printf("name: %s, age: %d, score: %g\n", name, age, score);
        }
};
</script></code></pre>


`class`是 C++ 中新增的关键字，专门用来定义类，`Student`是类的名称；类名的首字母一般大写，以和其他的标识符区分开，`{}`内部是类所包含的`成员变量`和`成员函数`，它们统称为`类的成员（Member）`；由`{}`包围起来的部分有时也称为`类体`，和函数体的概念类似；

`public`也是 C++ 的新增关键字，它只能用在类的定义中，表示类的`成员变量`或`成员函数`具有`“公开”的访问权限`

C++通过`public`、`protected`、`private`三个关键字来控制成员变量和成员函数的访问权限，它们分别表示`公有的`、`受保护的`、`私有的`，被称为`成员访问限定符`；所谓访问权限，就是你能不能使用该类中的成员；

**C++ 中的 public、private、protected 只能修饰类的成员，不能修饰类，C++中的类没有共有私有之分**

在类的内部（定义类的代码内部），无论成员被声明为 public、protected 还是 private，都是可以互相访问的，没有访问权限的限制；

在类的外部（定义类的代码之外），只能通过对象访问成员，并且通过对象只能访问 public 属性的成员，不能访问 private、protected 属性的成员；

现在我们只需要使用 public 和 private 就行了，protected 将在继承中讲解；

> 
注意在类定义的最后有一个`分号;`，它是类定义的一部分，表示类定义结束了，不能省略；

整体上讲，上面的代码创建了一个 Student 类，它包含了 3 个成员变量和 1 个成员函数。

**类只是一个模板（Template），编译后不占用内存空间，所以在定义类时不能对成员变量进行初始化，因为没有地方存储数据；只有在创建对象以后才会给成员变量分配内存，这个时候就可以赋值了**

类可以理解为一种新的数据类型，该数据类型的名称是 Student。与 char、int、float 等基本数据类型不同的是，Student 是一种复杂数据类型，可以包含基本类型，而且还有很多基本类型中没有的特性；

**创建对象**
有了 Student 类后，就可以通过它来创建对象了，例如：
`Student zhang3;`：创建对象，Student是类名，zhang3是对象名；这和使用基本类型定义变量的形式类似：

除了创建单个对象，还可以创建对象数组：
`Student stus[100];`：该语句创建了一个 stus 数组，它拥有100个元素，每个元素都是 Student 类型的对象；

**访问类的成员**
创建对象以后，可以使用`点号.`来访问`成员变量`和`成员函数`，这和通过结构体变量来访问它的成员类似：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Student {
    public:
        char *name;
        int age;
        float score;

        void print() {
            printf("name: %s, age: %d, score: %g\n", name, age, score);
        }
};

int main() {
    Student zhang3;
    zhang3.name = (char *)"ZhangSan";
    zhang3.age = 14;
    zhang3.score = 130.5f;
    zhang3.print();
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:25:12]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [12:26:01]
$ ./a.out
name: ZhangSan, age: 14, score: 130.5
</script></code></pre>


stu 是一个对象，占用内存空间，可以对它的成员变量赋值，也可以读取它的成员变量；

**类通常定义在函数外面，当然也可以定义在函数内部，不过很少这样使用；**

**使用对象指针**
C语言中经典的指针在 C++ 中仍然广泛使用，尤其是指向对象的指针，没有它就不能实现某些功能；

使用`&`取地址符获取之前创建的`zhang3`对象的地址，并将其地址赋值给指针`ptr`：
`Student *ptr = &zhang3;`

或者直接使用`new`创建一个堆上的对象，并使用一个指针指向它：
`Student *ptr = new Student;`

有了对象指针后，可以通过`箭头->`来访问对象的`成员变量`和`成员函数`，这和通过结构体指针来访问它的成员类似：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Student {
    public:
        char *name;
        int age;
        float score;

        void print() {
            printf("name: %s, age: %d, score: %g\n", name, age, score);
        }
};

int main() {
    Student zhang3;
    zhang3.name = (char *)"ZhangSan";
    zhang3.age = 14;
    zhang3.score = 130.5f;
    zhang3.print();

    Student *p_zhang3 = new Student;
    p_zhang3 -> name = (char *)"ZhangSan";
    p_zhang3 -> age = 14;
    p_zhang3 -> score = 130.5f;
    p_zhang3 -> print();
    delete p_zhang3;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:33:35]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [12:34:00]
$ ./a.out
name: ZhangSan, age: 14, score: 130.5
name: ZhangSan, age: 14, score: 130.5
</script></code></pre>

## 类的成员变量和成员函数
类可以看做是一种数据类型，它类似于普通的数据类型，但是又有别于普通的数据类型。类这种数据类型是一个`包含成员变量和成员函数的集合`。

类的成员变量和普通变量一样，也有数据类型和名称，占用固定长度的内存。但是，在定义类的时候不能对成员变量赋值，因为类只是一种数据类型或者说是一种模板，本身不占用内存空间，而变量的值则需要内存来存储。

类的成员函数也和普通函数一样，都有返回值和参数列表，它与一般函数的区别是：成员函数是一个类的成员，出现在类体中，它的作用范围由类来决定；而普通函数是独立的，作用范围是全局的，或位于某个命名空间内；

之前我们定义的Student类：
<pre><code class="language-cpp line-numbers"><script type="text/plain">class Student {
    public:
        char *name;
        int age;
        float score;

        void print() {
            printf("name: %s, age: %d, score: %g\n", name, age, score);
        }
};
</script></code></pre>

这段代码在类体中定义了成员函数，我们也可以在类体中声明成员函数，而在类体外定义函数：
<pre><code class="language-cpp line-numbers"><script type="text/plain">class Student {
    public:
        char *name;
        int age;
        float score;

        void print();
};

void Student::print() {
    printf("name: %s, age: %d, score: %g\n", name, age, score);
}
</script></code></pre>


在类体中直接定义函数时，不需要在函数名前面加上类名，因为函数属于哪一个类不言而喻；
但当成员函数定义在类外时，就必须在函数名前面加上类名予以限定；`::`被称为`域解析符`（也称作用域运算符或作用域限定符），用来连接类名和函数名，指明当前函数属于哪个类；

成员函数必须先在类体中作原型声明，然后在类外定义，也就是说类体的位置应在函数定义之前；

**inline成员函数**
在类体中和类体外定义成员函数是有区别的：
**在类体中定义的成员函数会自动成为内联函数，在类体外定义的不会**；

内联函数一般不是我们所期望的，它会将函数调用处用函数体替代，所以我**建议在类体内部对成员函数作声明，而在类体外部进行定义**，这是一种良好的编程习惯，实际开发中大家也是这样做的；

当然，**如果成员函数比较短小，希望定义为内联函数，那也没有什么不妥的**；

如果你既希望将函数定义在类体外部，又希望它是内联函数，那么可以在**定义函数时**加 inline 关键字；但是多此一举，没有必要！

## 类成员的访问权限
C++通过`public`、`protected`、`private`三个关键字来控制成员变量和成员函数的访问权限，它们分别表示`公有的`、`受保护的`、`私有的`，被称为`成员访问限定符`；

> 
C++ 中的 public、private、protected **只能修饰类的成员**，**不能修饰类**，C++中的类没有共有私有之分；

在类的内部（定义类的代码内部），无论成员被声明为 public、protected 还是 private，都是可以互相访问的，没有访问权限的限制；
在类的外部（定义类的代码之外），只能通过对象访问成员，并且通过对象只能访问 public 属性的成员，不能访问 private、protected 属性的成员；

> 
现在重点讲解 public 和 private，protected 将在继承中讲解；

先来看修改后的Student类：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Student {
    public:
        void setname(char *name) {
            m_name = name;
        }
        void setage(int age) {
            m_age = age;
        }
        void setscore(float score) {
            m_score = score;
        }
        char *getname() {
            return m_name;
        }
        int getage() {
            return m_age;
        }
        float getscore() {
            return m_score;
        }
    public:
        void print();
    private:
        char *m_name;
        int m_age;
        float m_score;
};

void Student::print() {
    printf("name: %s, age: %d, score: %g\n", m_name, m_age, m_score);
}

int main() {
    Student stu;
    stu.setname((char *)"ZhangSan");
    stu.setage(15);
    stu.setscore(134.5f);
    stu.print();

    Student *pstu = new Student;
    pstu -> setname((char *)"LiSi");
    pstu -> setage(14);
    pstu -> setscore(144.5);
    printf("name: %s, age: %d, score: %g\n", pstu->getname(), pstu->getage(), pstu->getscore());
    delete pstu;

    return 0;
}
</script></code></pre>


类的声明和成员函数的定义都是类定义的一部分，在实际开发中，我们通常将类的声明放在头文件中，而将成员函数的定义放在源文件中；

类中的成员变量`m_name`、`m_age`和`m_ score`被设置成`private`属性，在类的外部不能通过对象访问。也就是说，私有成员变量和成员函数只能在类内部使用，在类外都是无效的；

成员函数`setname()`、`setage()`和`setscore()`被设置为`public`属性，是公有的，可以通过对象访问；

成员变量大都以`m_开头`，这是约定成俗的写法，不是语法规定的内容。以`m_开头`既可以一眼看出这是成员变量，又可以和成员函数中的形参名字区分开；

**简单地谈类的封装**
private 关键字的作用在于更好地隐藏类的内部实现；
该向外暴露的接口（能通过对象访问的成员）都声明为 public；
不希望外部知道、或者只在类内部使用的、或者对外部没有影响的成员，都建议声明为 private；

根据C++软件设计规范，实际项目开发中的成员变量以及只在类内部使用的成员函数（只被成员函数调用的成员函数）都建议声明为 private，而只将允许通过对象调用的成员函数声明为 public；
另外还有一个关键字 protected，声明为 protected 的成员在类外也不能通过对象访问，但是在它的派生类内部可以访问；

为了访问类的 private 成员变量，需要额外定义两个 public 属性的成员函数，一个用来设置成员变量的值，一个用来修改成员变量的值；
setname()、setage()、setscore() 函数用来设置成员变量的值；
getname()、getage()、getscore() 函数用来获取成员变量的值；

成员变量的赋值函数通常称为`set函数`，它的名字通常`以set开头`，后跟成员变量的名字；
成员变量的读取函数通常称为`get函数`，它的名字通常`以get开头`，后跟成员变量的名字；

除了set函数和get函数，在创建对象时还可以调用构造函数来初始化各个成员变量，不过构造函数只能给成员变量赋值一次，以后再修改还得借助set函数；

这种将成员变量声明为 private、将部分成员函数声明为 public 的做法体现了**类的封装性**；
所谓封装，是指尽量隐藏类的内部实现，只向用户提供有用的成员函数；

**public 和 private**
声明为 private 的成员和声明为 public 的成员的次序任意，既可以先出现 private 部分，也可以先出现 public 部分。如果既不写 private 也不写 public，就默认为 private。

在一个类体中，private 和 public 可以分别出现多次。每个部分的有效范围到出现另一个访问限定符或类体结束时（最后一个右花括号）为止。但是为了使程序清晰，应该养成这样的习惯，使每一种成员访问限定符在类定义体中只出现一次；

## 对象的内存模型
类是创建对象的模板，本身并不占用存储空间，只有通过类创建出来的对象才会占用实实在在的空间（栈、堆上）；

不同对象的成员变量的值可能不同，需要单独分配内存来存储，但是不同对象的成员函数的代码是一样的；
所以编译器实际上会将成员变量和成员函数分开存储，分别为每个对象的成员变量分配内存，但是所有对象都共享同一段函数代码；

**成员变量在堆区或栈区分配内存，成员函数在代码区分配内存**

如：通过 sizeof 获取对象的内存占用大小；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

struct S {
    char *name;
    int age;
    float score;
};

class C1 {
    char *name;
    int age;
    float score;
};

class C2 {
    char *name;
    int age;
    float score;
    void func1() {}
    void func2() {}
    void func3() {}
};

int main() {
    S s;
    C1 c1;
    C2 c2;
    printf("sizeof S: %lu, sizeof s: %lu\n", sizeof(S), sizeof(s));
    printf("sizeof C1: %lu, sizeof c1: %lu\n", sizeof(C1), sizeof(c1));
    printf("sizeof C2: %lu, sizeof c2: %lu\n", sizeof(C2), sizeof(c2));
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:19:41]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [14:19:57]
$ ./a.out
sizeof S: 16, sizeof s: 16
sizeof C1: 16, sizeof c1: 16
sizeof C2: 16, sizeof c2: 16
</script></code></pre>


三个聚合类型`S`、`C1`、`C2`，成员变量都是`char *`、`int`、`float`，占用的内存分别为8、4、4；总和为16字节；
上面的例子中，可以发现，不管有没有成员函数，对象占用的内存大小总是和成员变量有关，与成员函数无关；

**在对象的内存分布中，只存储有成员变量，并且和结构体一样，遵循内存对齐规则**；

类中的成员变量按照声明的顺序排列，和结构体一样，也说明了类其实是在结构体的基础上改进的；

## 成员函数的实现
从上节的分析中可以看出，对象的内存中只保留了成员变量，除此之外没有任何其他信息，程序运行时不知道 stu 的类型为 Student，也不知道它还有几个成员函数 get函数、set函数、print()函数，C++ 究竟是如何通过对象调用成员函数的呢？

C++和C语言的编译方式不同：
C语言中的函数在编译时名字不变，或者只是简单的加一个下划线_（不同的编译器有不同的实现），例如，`func()` 编译后为 `func()` 或 `_func()`；
而C++中的函数在编译时会根据它所在的`命名空间`、它所属的`类`、以及它的`参数列表`（也叫参数签名）等信息进行`重新命名`，形成一个新的函数名；这个新的函数名只有编译器知道，对用户是不可见的；对函数重命名的过程叫做`名字编码（Name Mangling）`，是通过一种特殊的算法来实现的；

Name Mangling的算法是可逆的，既可以通过现有函数名计算出新函数名，也可以通过新函数名逆向推演出原有函数名；
Name Mangling可以确保新函数名的唯一性，只要函数所在的命名空间、所属的类、包含的参数列表等有一个不同，最后产生的新函数名也不同；

**this指针**
**不管是普通函数还是成员函数，最终都会被编译成一个与对象无关的全局函数**；

C++规定，**编译成员函数时要额外添加一个参数，把当前对象的指针传递进去，通过指针来访问成员变量**，这个指针就是`this`指针；

这个指针总是指向调用成员函数的对象所在的内存，不同的对象调用成员函数，this指针的值也会不同；
注意：这个指针是编译器自动传递的，隐式完成的，对程序员是透明的，不需要我们进行干涉；

这样，我们再来分析一下成员函数的调用：
对于成员函数：`void Student::func();`，编译后变成：`void new_function_name(Student *const this);`
在调用的时候：`stu.func();`，会被转换为：`new_function_name(&stu);` 类似的形式；

## 构造函数（Constructor）
在C++中，有一种特殊的成员函数，**它的名字和类名相同，没有返回值，不需要用户显式调用（用户也不能调用），而是在创建对象时自动执行**；这种特殊的成员函数就是`构造函数（Constructor）`；

在前面，我们通过成员函数 setname()、setage()、setscore() 分别为成员变量 name、age、score 赋值，这样做虽然有效，但显得有点麻烦。有了构造函数，我们就可以简化这项工作，在创建对象的同时为成员变量赋值：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Student {
    public:
        Student(char *name, int age, float score);
        void print();
    private:
        char *m_name;
        int m_age;
        float m_score;
};

Student::Student(char *name, int age, float score) {
    m_name = name;
    m_age = age;
    m_score = score;
}

void Student::print() {
    printf("name: %s, age: %d, score: %g\n", m_name, m_age, m_score);
}

int main() {
    // 在栈上创建一个具名对象stu
    Student stu((char *)"Otokaze", 18, 111.5f);
    stu.print();

    // 在堆上创建一个匿名对象，使用指针ptr指向它
    Student *ptr = new Student((char *)"Otokaze", 18, 111.5f);
    ptr -> print();
    delete ptr;

    // 在栈上创建一个匿名对象
    Student((char *)"Otokaze", 18, 111.5f).print();

    // 在堆上创建一个匿名对象
    (new Student((char *)"Otokaze", 18, 111.5f)) -> print();

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:05:12] C:130
$ g++ a.cpp

# root @ arch in ~/work on git:master x [15:05:45]
$ ./a.out
name: Otokaze, age: 18, score: 111.5
name: Otokaze, age: 18, score: 111.5
name: Otokaze, age: 18, score: 111.5
name: Otokaze, age: 18, score: 111.5
</script></code></pre>


**构造函数一般为 public 属性，否则创建对象时无法调用**；
某些情况下也会设为 private、protected 属性，如不希望创建当前类的对象，而是希望创建当前类的子类的对象时，就会将 构造函数/析构函数 设置为 protected 属性；

构造函数没有返回值，因为没有变量来接收返回值，即使有也毫无用处，这意味着：
- 不管是声明还是定义，函数名前面都不能出现返回值类型，即使是 void 也不允许；
- 函数体中不能有 return 语句

**构造函数的重载**
和普通成员函数一样，构造函数是允许重载的。一个类可以有多个重载的构造函数，创建对象时根据传递的实参来判断调用哪一个构造函数。

构造函数的调用是强制性的，一旦在类中定义了构造函数，那么创建对象时就一定要调用，不调用是错误的。如果有多个重载的构造函数，那么创建对象时提供的实参必须和其中的一个构造函数匹配；反过来说，创建对象时只有一个构造函数会被调用。

对上面的代码，如果写作`Student stu`或者`new Student`就是错误的，因为类中包含了构造函数，而创建对象时却没有调用；

如果我们再添加一个没有参数的构造函数`Student()`，那么这两个构造函数就构成了重载，上面的语句就不会报错了；

**默认构造函数**
如果用户自己没有定义构造函数，那么编译器会自动生成一个默认的构造函数，只是这个构造函数的函数体是空的，也没有形参，也不执行任何操作；

比如上面的 Student 类，默认生成的构造函数：`Student() {}`

**一个类必须有构造函数，要么用户自己定义，要么编译器自动生成**

一旦用户自己定义了构造函数，不管有几个，也不管形参如何，编译器都不再自动生成；

实际上编译器只有在必要的时候才会生成默认构造函数，而且它的函数体一般不为空；
默认构造函数的目的是帮助编译器做初始化工作，而不是帮助程序员；这是C++的内部实现机制，这里不再深究；

最后需要注意的一点是，调用没有参数的构造函数也可以省略括号；
在栈上创建对象可以写作`Student stu()`或`Student stu`，在堆上创建对象可以写作`Student *pstu = new Student()`或`Student *pstu = new Student`，它们都会调用构造函数`Student()`；

以前我们就是这样做的，创建对象时都没有写括号，其实是调用了默认的构造函数；

## 构造函数的参数初始化表
构造函数的一项重要功能是对成员变量进行初始化，为了达到这个目的，可以在构造函数的函数体中对成员变量一一赋值，还可以采用`参数初始化表`；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Student {
    public:
        Student(char *name, int age, float score);
        void print();
    private:
        char *m_name;
        int m_age;
        float m_score;
};

// Student::Student(char *name, int age, float score) {
//     m_name = name;
//     m_age = age;
//     m_score = score;
// }

Student::Student(char *name, int age, float score) : m_name(name), m_age(age), m_score(score) {
    // TODO
}

void Student::print() {
    printf("name: %s, age: %d, score: %g\n", m_name, m_age, m_score);
}

int main() {
    // 在栈上创建一个具名对象stu
    Student stu((char *)"Otokaze", 18, 111.5f);
    stu.print();

    // 在堆上创建一个匿名对象，使用指针ptr指向它
    Student *ptr = new Student((char *)"Otokaze", 18, 111.5f);
    ptr -> print();
    delete ptr;

    // 在栈上创建一个匿名对象
    Student((char *)"Otokaze", 18, 111.5f).print();

    // 在堆上创建一个匿名对象
    (new Student((char *)"Otokaze", 18, 111.5f)) -> print();

    return 0;
}
</script></code></pre>


`冒号:`后面紧跟`m_name(name), m_age(age), m_score(score)`语句，这个语句的意思相当于函数体内部的`m_name = name; m_age = age; m_score = score;`语句，也是赋值的意思；

> 
使用参数初始化表并没有效率上的优势，仅仅是书写方便，尤其是成员变量较多时，这种写法非常简明明了，并且某些情况下必须使用参数初始化列表来初始化某些变量（如const成员变量）；

参数初始化表可以用于全部成员变量，也可以只用于部分成员变量；

**参数初始化顺序与初始化表列出的变量的顺序无关，它只与成员变量在类中声明的顺序有关**

**参数初始化表还有一个很重要的作用，那就是初始化 const 成员变量**；
**初始化 const 成员变量的唯一方法就是使用参数初始化表**；

## 析构函数（Destructor）
创建对象时系统会自动调用构造函数进行初始化工作，同样，销毁对象时系统也会自动调用一个函数来进行清理工作，例如释放分配的内存、关闭打开的文件等，这个函数就是`析构函数`；

**析构函数（Destructor）也是一种特殊的成员函数，没有返回值，不需要程序员显式调用（程序员也没法显式调用），而是在销毁对象时自动执行**。构造函数的名字和类名相同，而析构函数的名字是在类名前面加一个`~符号`；

**析构函数没有参数，不能被重载，因此一个类只能有一个析构函数**；如果用户没有定义，编译器会自动生成一个默认的析构函数；

一般情况下，并不需要自己定义一个析构函数，默认的就可以了，但是如果在对象中使用了动态分配的内存、打开的文件、socket等，就需要自己定义一个析构函数，释放对应的内存，打开的文件、socket等资源；

和构造函数一样，析构函数一般也为 public 属性；

**析构函数的执行时机**
析构函数在对象被销毁时调用，而对象的销毁时机与它所在的内存区域有关：
- 在所有函数之外创建的对象是全局对象，它和全局变量类似，位于内存分区中的全局数据区，程序在结束执行时会调用这些对象的析构函数；
- 在函数内部创建的对象是局部对象，它和局部变量类似，位于栈区，函数执行结束时会调用这些对象的析构函数；
- new 创建的对象位于堆区，通过 delete 删除时才会调用析构函数；如果没有 delete，析构函数就不会被执行；

下面的例子演示了析构函数的执行：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
    public:
        A(const char *flag) : m_flag(flag) {
            printf("[%s] call Constructor_Function\n", m_flag);
        }
        ~A() {
            printf("[%s] call Destructor_Function\n", m_flag);
        }
    private:
        const char *m_flag;
};

// 全局数据区 data
A g_a("global");

int main() {
    // 栈区 stack
    A main_a("main_stack");

    // 堆区 heap
    A *heap_a = new A("heap");
    delete heap_a;

    // 堆区 heap, 不调用 delete
    A *heap_b = new A("heap_no_delete");
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:10:15]
$ g++ a.cpp
a.cpp: In function ‘int main()’:
a.cpp:29:8: warning: unused variable ‘heap_b’ [-Wunused-variable]
     A *heap_b = new A("heap_no_delete");
        ^~~~~~

# root @ arch in ~/work on git:master x [16:10:37]
$ ./a.out
[global] call Constructor_Function
[main_stack] call Constructor_Function
[heap] call Constructor_Function
[heap] call Destructor_Function
[heap_no_delete] call Constructor_Function
[main_stack] call Destructor_Function
[global] call Destructor_Function
</script></code></pre>

## this指针
`this`是 C++ 中的一个关键字，也是一个`const指针`，它指向当前对象，通过它可以访问当前对象的所有成员；

所谓当前对象，是指正在使用的对象；例如对于`stu.print();`，stu就是当前对象，this就指向stu；
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Student {
    public:
        Student(char *name, int age, float score);
        void setname(char *name);
        void setage(int age);
        void setscore(float score);
        void print();
    private:
        char *name;
        int age;
        float score;
};

Student::Student(char *name, int age, float score) : name(name), age(age), score(score) {}

void Student::setname(char *name) {
    this->name = name;
}

void Student::setage(int age) {
    this->age = age;
}

void Student::setscore(float score) {
    this->score = score;
}

void Student::print() {
    printf("name: %s, age: %d, score: %g\n", name, age, score);
}

int main() {
    Student stu((char *)"A", 1, 1.1);
    stu.print();
    stu.setname((char *)"B");
    stu.setage(2);
    stu.setscore(2.2);
    stu.print();
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:28:56]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [16:29:29]
$ ./a.out
name: A, age: 1, score: 1.1
name: B, age: 2, score: 2.2
</script></code></pre>


对于this指针，有几点要注意：
- this 是 const 指针，指针的值是不能被修改的，一切企图修改该指针的操作，如赋值、递增、递减等都是不允许的；
- this 只能在成员函数内部使用，用在其他地方没有意义，也是非法的；
- 只有当对象被创建后 this 才有意义，因此不能在 static 成员函数中使用（后续会讲到 static 成员）；

## static静态成员变量
对象的内存中包含了成员变量，不同的对象占用不同的内存，这使得不同对象的成员变量相互独立，它们的值不受其他对象的影响。例如有两个相同类型的对象 a、b，它们都有一个成员变量 m_name，那么修改 a.m_name 的值不会影响 b.m_name 的值；

可是有时候我们希望在多个对象之间共享数据，对象 a 改变了某份数据后对象 b 可以检测到；
共享数据的典型使用场景是计数，以前面的 Student 类为例，如果我们想知道班级中共有多少名学生，就可以设置一份共享的变量，每次创建对象时让该变量加 1；

在C++中，我们可以使用**静态成员变量**来实现多个对象共享数据的目标。`静态成员变量是一种特殊的成员变量，它被关键字static修饰`；

例如：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Student {
    public:
        Student(char *name, int age, float score);
        void print();
    private:
        char *m_name;
        int m_age;
        float m_score;
        static int m_total; // 声明静态成员，此时并未分配内存空间给m_total
};

int Student::m_total = 0; // 定义并初始化静态成员m_total，此时分配内存空间

Student::Student(char *name, int age, float score) : m_name(name), m_age(age), m_score(score) {
    m_total++;
}

void Student::print() {
    printf("[%d] name: %s, age: %d, score: %g\n", m_total, m_name, m_age, m_score);
}

int main() {
    Student s1((char *)"A", 1, 11);
    s1.print();
    Student s2((char *)"B", 2, 22);
    s2.print();
    Student s3((char *)"C", 3, 33);
    s3.print();
    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:45:59]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [16:46:00]
$ ./a.out
[1] name: A, age: 1, score: 11
[2] name: B, age: 2, score: 22
[3] name: C, age: 3, score: 33
</script></code></pre>


**static 成员变量属于类，不属于某个具体的对象**，即使创建多个对象，也只为 m_total 分配一份内存，所有对象使用的都是这份内存中的数据。当某个对象修改了 m_total，也会影响到其他对象；

static成员变量和函数中的static变量很相似，两者都是存储在`全局数据区`（`data`或`bss`）；
虽然它们存储在全局数据区，但是它们的作用域被限制了，对于函数的static变量：限制于该函数之内；对于类的static成员变量：限制于类的作用域之内；

静态成员变量在初始化时不能再加 static，但必须要有数据类型；
被 private、protected、public 修饰的静态成员变量都可以用这种方式初始化；

static 成员变量既可以通过对象来访问，也可以通过类来访问，都是访问的同一份数据

> 
注意：static成员变量依旧遵循类的访问控制符（public、protected、private）的限制：
对于static变量，基类和派生类访问的都是同一块内存，都是共享的，并不会创建一份新的static变量；

例如：对于public属性的static变量，可以通过类访问`Student::m_total = 10;`，也可以通过对象访问`stu.m_total = 20;`
对于protected、private属性的static变量，只能在类的内部进行访问，不能通过类、对象进行访问；

## static静态成员函数
在类中，static除了可以声明静态成员变量，还可以声明静态成员函数；
普通成员函数可以访问所有成员（包括成员变量和成员函数），静态成员函数只能访问静态成员；

编译器在编译一个普通成员函数时，会隐式地增加一个形参this，并把当前对象的地址赋值给this，所以普通成员函数只能在创建对象后通过对象来调用，因为它需要当前对象的地址；
而静态成员函数可以通过类来直接调用，编译器不会为它增加形参this，它不需要当前对象的地址，所以不管有没有创建对象，都可以调用静态成员函数；

> 
静态成员函数与普通成员函数的根本区别在于：普通成员函数有 this 指针，可以访问类中的任意成员；而静态成员函数没有 this 指针，只能访问静态成员（包括静态成员变量和静态成员函数）；
static成员函数和static成员变量一样，遵循public、protected、private控制符的限制；
并且，static函数没有“虚函数”一说；因为static函数实际上是“加上了访问控制的全局函数”；

例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Student {
    public:
        Student(char *name, int age, float score);
        void print();
    public:
        static int get_cnt();
        static float get_sum();
    private:
        static int m_cnt;
        static float m_sum;
    private:
        char *m_name;
        int m_age;
        float m_score;
};

int Student::m_cnt = 0;
float Student::m_sum = 0.0f;

Student::Student(char *name, int age, float score) : m_name(name), m_age(age), m_score(score) {
    m_cnt++;
    m_sum += score;
}

void Student::print() {
    printf("name: %s, age: %d, score: %g\n", m_name, m_age, m_score);
}

int Student::get_cnt() {
    return m_cnt;
}

float Student::get_sum() {
    return m_sum;
}

int main() {
    Student s1((char *)"A", 1, 11);
    s1.print();

    Student s2((char *)"B", 2, 22);
    s2.print();

    Student s3((char *)"C", 3, 33);
    s3.print();

    printf("student_total: %d, score_total: %g\n", Student::get_cnt(), Student::get_sum());

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:58:09]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [17:58:11]
$ ./a.out
name: A, age: 1, score: 11
name: B, age: 2, score: 22
name: C, age: 3, score: 33
student_total: 3, score_total: 66
</script></code></pre>


**在C++中，静态成员函数的主要目的是访问静态成员**

和静态成员变量类似，静态成员函数在声明时要加 static，在定义时不能加 static；
静态成员函数可以通过类来调用（一般都是这样做），也可以通过对象来调用；

## 类与const关键字
在类中，如果你不希望某些数据被修改，可以使用const关键字加以限定；const 可以用来修饰`成员变量`、`成员函数`以及`对象`；

**const成员变量**
const 成员变量的用法和普通 const 变量的用法相似，只需要在声明时加上 const 关键字；
初始化 const 成员变量只有一种方法，就是通过参数初始化表；

**const成员函数**
const 成员函数可以使用类中的所有成员变量，但是不能修改它们的值，这种措施主要还是为了保护数据而设置的；const 成员函数也称为**常成员函数**；

常成员函数需要在**声明**和**定义**的时候在函数头部的**结尾**加上 const 关键字：

加了const限定的函数和没加const的函数是两个不同的函数，如：`void func();`和`void func() const;`是不一样的；

**const对象**
const 也可以用来修饰对象，称为`常对象`；一旦将对象定义为常对象之后，就只能调用类的 const 成员了；

一旦将对象定义为常对象之后，不管是哪种形式，该对象就只能访问被 const 修饰的成员了（包括 const 成员变量和 const 成员函数），因为非 const 成员可能会修改对象的数据（编译器也会这样假设），C++禁止这样做；

举例说明：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Student {
    public:
        Student(const char *name, int age, float score);
    public:
        const char * getname() const;
        int getage() const;
        float getscore() const;
        void setname(const char *name);
        void setage(int age);
        void setscore(float score);
        void print() const;
    private:
        const char *m_name;
        int m_age;
        float m_score;
};

Student::Student(const char *name, int age, float score) : m_name(name), m_age(age), m_score(score) {}

const char * Student::getname() const { return m_name; }
int Student::getage() const { return m_age; }
float Student::getscore() const { return m_score; }
void Student::setname(const char *name) { m_name = name; }
void Student::setage(int age) { m_age = age; }
void Student::setscore(float score) { m_score = score; }

void Student::print() const {
    printf("name: %s, age: %d, score: %g\n", m_name, m_age, m_score);
}

int main() {
    Student stu1("A", 1, 11);
    stu1.print();

    const Student stu2("B", 2, 22);
    stu2.print();

    const Student *pstu = new Student("C", 3, 33);
    pstu -> print();
    delete pstu;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [19:10:16]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [19:10:47]
$ ./a.out
name: A, age: 1, score: 11
name: B, age: 2, score: 22
name: C, age: 3, score: 33
</script></code></pre>

## friend友元函数和友元类
一个类中可以有 public、protected、private 三种属性的成员，通过对象可以访问 public 成员，只有本类中的函数可以访问本类的 private 成员；
现在，我们来介绍一种例外情况——`友元（friend）`；借助友元，可以使得其他类中的成员函数以及全局范围内的函数访问当前类的 private 成员；

**友元函数**
在当前类以外定义的、不属于当前类的函数也可以在类中声明，但要在前面加`friend`关键字，这样就构成了`友元函数`；
友元函数可以是不属于任何类的非成员函数，也可以是其他类的成员函数；

友元函数可以访问当前类中的所有成员，包括 public、protected、private 属性的；

**将非成员函数声明为友元函数**
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Student {
    public:
        Student(const char *name, int age, float score);
    public:
        const char * getname() const;
        int getage() const;
        float getscore() const;
        void setname(const char *name);
        void setage(int age);
        void setscore(float score);
        friend void print(Student *ptr);
    private:
        const char *m_name;
        int m_age;
        float m_score;
};

Student::Student(const char *name, int age, float score) : m_name(name), m_age(age), m_score(score) {}

const char * Student::getname() const { return m_name; }
int Student::getage() const { return m_age; }
float Student::getscore() const { return m_score; }
void Student::setname(const char *name) { m_name = name; }
void Student::setage(int age) { m_age = age; }
void Student::setscore(float score) { m_score = score; }

void print(Student *ptr) {
    printf("name: %s, age: %d, score: %g\n", ptr->m_name, ptr->m_age, ptr->m_score);
}

int main() {
    Student stu1("A", 1, 11);
    print(&stu1);

    const Student stu2("B", 2, 22);
    print((Student *)&stu2);

    const Student *pstu = new Student("C", 3, 33);
    print((Student *)pstu);
    delete pstu;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [19:25:19]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [19:25:21]
$ ./a.out
name: A, age: 1, score: 11
name: B, age: 2, score: 22
name: C, age: 3, score: 33
</script></code></pre>


**将其他类的成员函数声明为友元函数**
friend 函数不仅可以是全局函数（非成员函数），还可以是另外一个类的成员函数：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class Address; // 提前声明 Address 类

// 声明 Student 类
class Student {
    public:
        Student(const char *name, int age, float score);
    public:
        const char * getname() const;
        int getage() const;
        float getscore() const;
        void setname(const char *name);
        void setage(int age);
        void setscore(float score);
        void print(Address *addr) const;
    private:
        const char *m_name;
        int m_age;
        float m_score;
};

// 声明 Address 类
class Address {
    public:
        Address(const char *province, const char *city, char const *district);
        friend void Student::print(Address *addr) const;
    private:
        const char *m_province;
        const char *m_city;
        const char *m_district;
};

// 实现 Student 类
Student::Student(const char *name, int age, float score) : m_name(name), m_age(age), m_score(score) {}

const char * Student::getname() const { return m_name; }
int Student::getage() const { return m_age; }
float Student::getscore() const { return m_score; }
void Student::setname(const char *name) { m_name = name; }
void Student::setage(int age) { m_age = age; }
void Student::setscore(float score) { m_score = score; }

void Student::print(Address *addr) const {
    printf("name: %s, age: %d, score: %g | province: %s, city: %s, district: %s\n", m_name, m_age, m_score, addr->m_province, addr->m_city, addr->m_district);
}

// 实现 Address 类
Address::Address(const char *province, const char *city, const char *district) : m_province(province), m_city(city), m_district(district) {}

int main() {
    Student stu1("A", 1, 11);
    Address addr1("广东省", "广州市", "天河区");
    stu1.print(&addr1);

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [19:43:02] C:1
$ g++ a.cpp

# root @ arch in ~/work on git:master x [19:43:05]
$ ./a.out
name: A, age: 1, score: 11 | province: 广东省, city: 广州市, district: 天河区
</script></code></pre>


一个函数可以被多个类声明为友元函数，这样就可以访问多个类中的 private 成员

**友元类**
不仅可以将一个函数声明为一个类的“朋友”，还可以将整个类声明为另一个类的“朋友”，这就是友元类；友元类中的所有成员函数都是另外一个类的友元函数；

例如将类 B 声明为类 A 的友元类，那么类 B 中的所有成员函数都是类 A 的友元函数，可以访问类 A 的所有成员，包括 public、protected、private 属性的；

将上面的代码中的29行改为`friend class Student;`，Student就成了Address的友元类；


关于友元，有两点需要说明：
- **友元的关系是单向的而不是双向的**；如果声明了类 B 是类 A 的友元类，不等于类 A 是类 B 的友元类，类 A 中的成员函数不能访问类 B 中的 private 成员；
- **友元的关系不能传递**；如果类 B 是类 A 的友元类，类 C 是类 B 的友元类，不等于类 C 是类 A 的友元类；

> 
除非有必要，一般不建议把整个类声明为友元类，而只将某些成员函数声明为友元函数，这样更安全一些

## 类其实也是一种作用域
**类其实也是一种作用域，每个类都会定义它自己的作用域**；

在类的作用域之外，普通的成员只能通过对象（可以是对象本身，也可以是对象指针或对象引用）来访问，静态成员既可以通过对象访问，又可以通过类访问，而 typedef 定义的类型只能通过类来访问；

<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
    public:
        typedef char *Str;
        static void show();
        void work();
};

void A::show() {
    printf("show()\n");
}

void A::work() {
    printf("work()\n");
}

int main() {
    A::Str str = (char *)"www.zfl9.com";
    printf("str: %s\n", str);

    A::show();

    A a;
    a.work();

    return 0;
}
</script></code></pre>


一个类就是一个作用域的事实能够很好的解释为什么我们在类的外部定义成员函数时必须同时提供类名和函数名；在类的外部，类内部成员的名字是不可见的；

一旦遇到类名，定义的剩余部分就在类的作用域之内了，这里的剩余部分包括参数列表和函数体；结果就是，我们可以直接使用类的其他成员而无需再次授权了；请看下面的例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <cstdio>

using namespace std;

class A {
    public:
        typedef char *Str;
        Str show(Str str);
};

A::Str A::show(Str str) {
    printf("str: %s\n", str);
    return str;
}

int main() {
    A a;
    A::Str str = (char *)"www.zfl9.com";
    a.show(str);

    return 0;
}
</script></code></pre>


我们在定义 show() 函数时用到了类 A 中定义的一种类型 Str，因为前面已经指明了当前正位于 A 类的作用域中，所以不用再使用A::Str这样的冗余形式；

另一方面，函数的返回值类型出现在函数名之前，当成员函数定义在类的外部时，返回值类型中使用的名字都位于类的作用域之外，此时必须指明该名字是哪个类的成员；如 show() 的返回值就必须指定作用域为类 A，不然就会报错；

## C++中class和struct的区别
C++ 中保留了C语言的 struct 关键字，并且加以扩充；
在C语言中，struct 只能包含成员变量，不能包含成员函数；
而在C++中，struct 类似于 class，既可以包含成员变量，又可以包含成员函数；

C++中的 struct 和 class 基本是通用的，唯有几个细节不同：
- 使用 class 时，类中的成员默认都是 private 属性的；而使用 struct 时，结构体中的成员默认都是 public 属性的；
- class 继承默认是 private 继承，而 struct 继承默认是 public 继承；
- class 可以使用模板，而 struct 不能；

**在编写C++代码时，强烈建议使用 class 来定义类，而使用 struct 来定义结构体，这样做语义更加明确**；

## C++ string字符串类
C++ 大大增强了对字符串的支持，除了可以使用C风格的字符串，还可以使用内置的 string 类；string 类处理起字符串来会方便很多，完全可以代替C语言中的字符数组或字符串指针；

使用 string 类需要包含头文件`<string>`，下面的例子介绍了几种定义 string 变量（对象）的方法：
`string s1;`：只有定义，没有初始化，编译器默认赋值一个空字符串`""`给s1；
`string s2 = "www.zfl9.com";`：定义的同时被初始化为`"www.zfl9.com"`，与C风格的字符串不同，string没有结尾字符`'\0'`；
`string s3 = s2;`：定义的同时用s2进行初始化，内容同s2；
`string s4(5, 's');`：s4被初始化为`"sssss"`；

在string内部，对很多运算符进行了重载，使得可以直接将`const char *`类型的字符串`"www.zfl9.com"`直接赋值给string类的对象；

与C风格的字符串不同，当我们需要知道字符串长度时，可以调用 string 类提供的 `length()` 函数；
由于 string 的末尾没有`'\0'`字符，所以 length() 返回的是字符串的真实长度；

**转换为`const char *`类型的字符串**
虽然 C++ 提供了 string 类来替代C语言中的字符串，但是在实际编程中，有时候还是需要使用C风格的字符串（`char *`类型）；
为此，string 类为我们提供了一个转换函数 `c_str()`，该函数能够将 string 字符串转换为C风格的字符串，并返回该字符串（`const char *`）；

**string字符串的输入输出**
string 类重载了输入输出运算符，可以像对待普通变量那样对待 string 变量，也就是用`>>`进行输入，用`<<`进行输出；

**访问字符串中的字符**
string 字符串也可以像C风格的字符串一样按照下标来访问其中的每一个字符；string 字符串的起始下标仍是从 0 开始；
如：`string s = "www.zfl9.com";`
访问第 3 个元素：`char c = s[3];`，修改第 0 个元素的值：`s[0] = 'W';`

**字符串的拼接**
有了 string 类，我们可以使用`+`或`+=`运算符来直接拼接字符串，非常方便；

用`+`来拼接字符串时，运算符的两边可以都是 string 字符串，也可以是一个 string 字符串和一个C风格的字符串，还可以是一个 string 字符串和一个字符数组，或者是一个 string 字符串和一个单独的字符；
