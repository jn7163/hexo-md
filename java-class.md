---
title: Java 类与对象
date: 2017-09-08 08:49:00
categories:
- java
tags:
- java
keywords: Java 类与对象
---

> 
Java 类与对象，类的定义及实例化、访问修饰符、变量作用域、this指针、函数重载与函数重写、基本类型装箱拆箱

<!-- more -->

## 类的定义及实例化
类必须先定义才能使用；类是创建对象的模板，创建对象也叫类的实例化；
每个类在编译之后都会生成一个以类名命名的`*.class`字节码文件；

简单的 Student 类定义：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Student {
    // 构造函数
    public Student(String name, int age, float score) {
        m_name = name;
        m_age = age;
        m_score = score;
    }

    // 成员函数 Getter/Setter
    public String getName() { return m_name; }
    public int getAge() { return m_age; }
    public float getScore() { return m_score; }
    public void setName(String name) { m_name = name; }
    public void setAge(int age) { m_age = age; }
    public void setScore(float score)  { m_score = score; }

    // 成员函数 print
    public void print() {
        out.printf("name: %s, age: %d, score: %.1f\n", m_name, m_age, m_score);
    }

    // 成员变量 private
    private String m_name;
    private int m_age;
    private float m_score;
}
</script></code></pre>


这里有一个地方与 C++ 不同，在 C++ 中，类 class 是没有所谓的访问控制符的，类都是公开访问权限的；
但是在 Java 中，类有两种访问权限：`public`、`[default]`（什么都不写就是`[default]`），区别：
- `public`：表示该类具有公开的访问权限，对任何类都可见；
- `[default]`：表示该类具有默认的访问权限，在同一包中可见；
- 区别：一个类文件中，最多只能有一个 public 属性的类，并且需要和文件名同名；而 [default] 属性的类没有此限制；


> 
Java 中的包 package，实际就是文件系统中的一个文件夹；
也就是说，[default] 类对于同一文件夹下的类都是可见的，但是子文件夹除外，因为子文件夹是另一个包 package 了；

一个类可以包含以下类型变量：
- `局部变量`：在方法或者语句块中定义的变量被称为局部变量；变量声明和初始化都是在方法中，方法结束后，变量就会自动销毁；
- `成员变量`：成员变量是定义在类中、方法体之外的变量；这种变量在创建对象的时候实例化（分配内存）；成员变量可以被类中的方法和特定类的语句访问；
- `类变量`：类变量也声明在类中，方法体之外，但必须声明为 static 类型，static 变量具有 static 生存期；


**构造函数**
在类实例化的过程中自动执行的方法叫做构造方法，它不需要你手动调用，也不能显示调用，构造方法可以在类实例化的过程中做一些初始化的工作；

构造方法的名称必须与类的名称相同，并且没有返回值，构造方法一般都是 public 访问权限，不过某些特殊的类除外；

每个类都有构造方法；如果没有显式地为类定义构造方法，Java 编译器将会为该类提供一个默认的构造方法（无参构造函数）；
但是如果显式定义了一个构造方法，那么 Java 编译器就不会再生成一个默认的构造方法了，这点和 C++ 是一样的；

> 
还有一点与 C++ 不同的是，Java 中的类没有`析构函数`，因为 Java 是一种垃圾回收语言，您无法预测何时对象将被销毁；

**finalize()**
Java 允许定义这样的方法，它在对象被垃圾收集器析构(回收)之前调用，这个方法叫做 finalize()，它用来清除回收对象；
finalize() 一般格式为：`protected void finalize() { statement }`；

例如，你可以使用 finalize() 来确保一个对象打开的文件被关闭了；在 finalize() 方法里，你必须指定在对象销毁时候要执行的操作；

但是请注意，finalize() 方法不同于 C++ 中的析构函数，即使声明了 finalize() 函数，也不一定会被 JVM 调用；
并且使用 finalize() 可能会带来某些性能问题，所以请尽量避免使用 finalize() 函数！

**创建对象**
对象是类的一个实例，创建对象的过程也叫类的实例化；对象是以类为模板来创建的；

在 Java 中，使用 new 关键字来创建对象，一般有以下三个步骤：
- 声明：声明一个对象，包括对象名称和对象类型，例如`Student stu;`，此时分配一个指针变量的空间；
- 实例化：使用关键字 new 来创建一个对象，在堆 heap 上创建对象；
- 初始化：使用 new 创建对象时，会调用构造方法初始化对象；


**访问类的成员**
创建对象之后，就可以通过`.`来访问类的成员函数和成员变量了；
因为 Java 中没有指针，也就没有 C/C++ 中的`->`操作符了；

下面是一个完整示例：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Student {
    // void main() 入口函数
    public static void main(String args[]) {
        Student stu;    // 分配一个指针变量的内存 - 64bits
        stu = new Student("Otokaze", 14, 111.5f);    // 在堆上创建对象，并调用构造函数进行初始化
        out.printf("name: %s, age: %d, score: %.1f\n", stu.getName(), stu.getAge(), stu.getScore());
        out.printf("name: %s, age: %d, score: %.1f\n", stu.m_name, stu.m_age, stu.m_score);
    }

    // 构造函数
    public Student(String name, int age, float score) {
        m_name = name;
        m_age = age;
        m_score = score;
    }

    // 成员函数 Getter/Setter
    public String getName() { return m_name; }
    public int getAge() { return m_age; }
    public float getScore() { return m_score; }
    public void setName(String name) { m_name = name; }
    public void setAge(int age) { m_age = age; }
    public void setScore(float score)  { m_score = score; }

    // 成员函数 print
    public void print() {
        out.printf("name: %s, age: %d, score: %.1f\n", m_name, m_age, m_score);
    }

    // 成员变量 private
    private String m_name;
    private int m_age;
    private float m_score;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:46:31]
$ javac Student.java

# root @ arch in ~/work on git:master x [9:46:33]
$ java Student
name: Otokaze, age: 14, score: 111.5
name: Otokaze, age: 14, score: 111.5
</script></code></pre>


## 访问控制符
Java 通过修饰符来控制类、属性和方法的访问权限和其他功能，通常放在语句的最前端；
Java 的修饰符很多，分为`访问修饰符`和`非访问修饰符`；本节仅介绍访问修饰符，非访问修饰符会在后续介绍；

访问修饰符也叫访问控制符，是指能够控制类、成员变量、成员函数的使用权限的关键字：
- `public`：公开的访问权限，对`所有类`都可见；可修饰`类`、`构造函数`、`成员函数`、`成员变量`、`接口`；
- `protected`：受保护的访问权限，对`同一包中`的类及其所有`子类`可见；可修饰`构造函数`、`成员函数`、`成员变量`；
- `private`：私有的访问权限，仅在`本类中`可见；可修饰`构造函数`、`成员函数`、`成员变量`；
- `[default]`：默认的访问权限，对`同一包中`的类可见；什么都不写就是`[default]`访问权限；


**访问控制和继承**
- 父类中声明为 public 的方法在子类中也必须为 public；
- 父类中声明为 protected 的方法在子类中要么声明为 protected，要么声明为 public；不能声明为 private；
- 父类中默认修饰符声明的方法，能够在子类中声明为 private；
- 父类中声明为 private 的方法，不能够被继承；声明为 private 的属性，能够被继承，但是在子类中不可见；


**一般原则**：
- 将不必要向外界暴露的成员变量都声明为 private，并且提供必要的 Getter/Setter 成员函数；
- 如果某些成员变量不能被外界直接访问，但是能够被子类访问，那么需要声明为 protected；
- 构造函数一般声明为 public，只在类中使用的成员函数都应该声明为 private 或 protected；


## 语句块、构造函数
在一个 Java 类中，可以有：`静态语句块`、`语句块`、`构造函数`三种不同的初始化方式；

无继承关系时的执行顺序：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Main {
    public static void main(String args[]) {
        out.println("+++++++ No.1 +++++++");
        new Test();

        out.println("\n+++++++ No.2 +++++++");
        new Test();

        out.println("\n+++++++ No.3 +++++++");
        new Test();
    }
}

class Test {
    // static 成员变量
    private static int cnt = 0;

    // static 语句块
    static {
        out.printf("static state_block\n");
    }

    // 语句块(初始化块)
    {
        cnt++;
        out.printf("init state_block <%d>\n", cnt);
    }

    // 构造函数
    public Test() {
        out.printf("constructor <%d>\n", cnt);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:35:54]
$ javac Main.java

# root @ arch in ~/work on git:master x [10:36:36]
$ java Main
+++++++ No.1 +++++++
static state_block
init state_block <1>
constructor <1>

+++++++ No.2 +++++++
init state_block <2>
constructor <2>

+++++++ No.3 +++++++
init state_block <3>
constructor <3>
</script></code></pre>


存在继承关系时的执行顺序：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Main {
    public static void main(String args[]) {
        out.println("+++++++ No.1 +++++++");
        new Derived();

        out.println("\n+++++++ No.2 +++++++");
        new Derived();

        out.println("\n+++++++ No.3 +++++++");
        new Derived();
    }
}

class Base {
    // static 语句块
    static {
        out.println("[class Base] static state_block");
    }

    // 初始化块
    {
        out.println("[class Base] init state_block");
    }

    // 构造函数
    public Base() {
        out.println("[class Base] constructor");
    }
}

class Derived extends Base {
    // static 成员变量
    private static int cnt = 0;

    // static 语句块
    static {
        out.println("[class Derived] static state_block");
    }

    // 初始化块
    {
        cnt++;
        out.printf("[class Derived] init state_block <%d>\n", cnt);
    }

    // 构造函数
    public Derived() {
        out.printf("[class Derived] constructor <%d>\n", cnt);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:48:15]
$ javac Main.java

# root @ arch in ~/work on git:master x [10:48:19]
$ java Main
+++++++ No.1 +++++++
[class Base] static state_block
[class Derived] static state_block
[class Base] init state_block
[class Base] constructor
[class Derived] init state_block <1>
[class Derived] constructor <1>

+++++++ No.2 +++++++
[class Base] init state_block
[class Base] constructor
[class Derived] init state_block <2>
[class Derived] constructor <2>

+++++++ No.3 +++++++
[class Base] init state_block
[class Base] constructor
[class Derived] init state_block <3>
[class Derived] constructor <3>
</script></code></pre>


总结：
- `static初始化块`的执行时机是在类装载的时候执行的，因此只执行一次；并且按照层级关系从基类到派生类依次执行；静态语句块总是最先执行的；
- 对于`初始化块`和`构造函数`，应该将它们看作一个整体，它们总是一起执行的，并且初始化块先于构造函数执行；总体顺序也是从基类到派生类依次执行；


## this关键字
在 C++ 中，this 是编译器隐式传给成员函数的一个对象指针，this 总是指向当前对象；
在 Java 中，this 的含义并没有发生变化，依旧表示当前对象的指针；只不过有些细节不同；

1) `this`可用于`委托构造`，语法为`this(param_list);`；
所谓的委托构造，就是在构造函数中调用本类中的其他构造函数来完成对象的初始化工作；
- 委托语句必须位于构造函数的首行；
- 只能在构造函数中使用委托构造；
- 一个构造函数中最多只能有一个委托构造语句；


2) `this`可用于调用被覆盖的同名成员变量；
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        Main m = new Main();
        m.print();
        m = new Main("Otokaze", 18, 144.5f);
        m.print();
    }

    public Main() {
        this("Unnamed", 18, 150.0f);
    }
    public Main(String name, int age, float score) {
        this.name = name;
        this.age = age;
        this.score = score;
    }

    public void print() {
        out.printf("name: %s, age: %d, score: %.1f\n", name, age, score);
    }

    private String name;
    private int age;
    private float score;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:59:14]
$ javac Main.java

# root @ arch in ~/work on git:master x [13:59:16]
$ java Main
name: Unnamed, age: 18, score: 150.0
name: Otokaze, age: 18, score: 144.5
</script></code></pre>


## 函数重载、重写
`函数重载（overload）`
同一作用域中，函数有`同样的名称`，但是`参数列表不相同`的情形，这样的同名不同参数的函数之间，互相称之为重载函数；

基本条件：
- 函数名必须相同；
- 函数参数必须不相同；
- 函数返回值可以相同，可以不相同；
- 函数的访问性可以相同、可以不相同；
- 函数的检查异常可以相同、可以不相同；
- 函数能够在同一个类中或者在一个子类中被重载（这点与 C++ 不同）；


`函数重写（override）`
子类重新定义父类中有`相同名称`和`参数`的`虚函数`，主要在`继承关系`中出现；
> 
除`构造函数`、`static成员函数`、`final成员函数`外，成员函数默认为 virtual 虚函数；

基本条件：
- 必须存在继承关系，且被重写函数为虚函数；
- 参数列表必须完全与被重写方法的相同；
- 返回类型与被重写方法相同，或者为其子类；
- 重写函数的访问性不能比被重写函数差；
- 重写函数和被重写函数的检查异常一致，或者为其子类；


重载和重写的本质：
- 重载是一个`编译期概念`、重写是一个`运行期概念`；
- 重载遵循所谓“编译期绑定”，即在编译时根据参数变量的类型判断应该调用哪个函数；
- 重写遵循所谓“运行期绑定”，即在运行的时候，根据引用变量所指向的实际对象的类型来调用函数；
- 重载是`编译期多态`，即`静态多态`；重写是`运行期多态`，即`动态多态`；


`@Override`是伪代码，表示重写（当然不写也可以），不过写上有如下好处：
- 可以当注释用，方便阅读；
- 编译器可以给你验证`@Override`下面的方法名是否是你父类中所有的，如果没有则报错；


## 变量初始值
`静态成员变量`的自动初始化赋值发生在类被`类加载器（classLoader）`加载的时候；系统会对没有显式初始化的静态成员变量在`静态存储区`赋予 0 值；
`普通成员变量`的自动初始化赋值发生在 new 分配内存的时候，自动调用`memset()`函数将每个字节用 0 来填充；
`局部变量`必须由程序员`显式初始化`，如果使用一个未初始化的局部变量，编译器会报错；

自动初始化赋值，即用 0 填充变量的每个字节；
- 整型（byte、short、int、long）：`0`；
- 单精度浮点型 float：`0.0f`；
- 双精度浮点型 double：`0.0d`；
- 字符型 char：`'\u0000'`；
- 布尔型 boolean：`false`；
- 引用型 refer：`null`；


## 构造函数细节问题
在 C++ 中，有这么一个细节被强调：
> 
类只是一个创建对象的模板，编译后不占用内存空间，所以在定义类时不能对成员变量进行初始化，因为没有地方存储数据；只有在创建对象以后才会给成员变量分配内存，这个时候就可以赋值了；

对于 Java，这条规则依旧适用，成员变量只有在对象分配了内存之后才能进行赋值；

不过有一个细节需要注意，先来看这个例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        new Main();
    }

    // 构造函数
    public Main() {
        out.printf("name: %s, age: %d, score: %.1f\n", m_name, m_age, m_score);
    }

    // 成员变量
    private String m_name = "Unnamed";
    private int m_age = 16;
    private float m_score = 100.0f;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:32:49]
$ javac Main.java

# root @ arch in ~/work on git:master x [12:33:04]
$ java Main
name: Unnamed, age: 16, score: 100.0
</script></code></pre>


对于习惯了 C++ 的朋友，是不是觉得上面的代码有点不对劲；
这三个成员变量 m_name、m_age、m_score，既不是静态生命周期变量，也不是函数、代码块中的局部变量；它们哪里来的内存空间来存储这些默认值？

别慌，我们来看一下它的正常形式：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        new Main();
    }

    // 构造函数
    public Main() {
        m_name = "Unnamed";
        m_age = 16;
        m_score = 100.0f;
        out.printf("name: %s, age: %d, score: %.1f\n", m_name, m_age, m_score);
    }

    // 成员变量
    private String m_name;
    private int m_age;
    private float m_score;
}
</script></code></pre>


是不是发现了什么？没错，其实前者的初始化方式是一种`语法糖`，在编译期间会被自动的转换为后者这种形式；


## 包装类、装箱与拆箱
基本类型（值类型）：`byte`、`short`、`int`、`long`、`float`、`double`、`boolean`、`char`
包装类（引用类型）：`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`、`Boolean`、`Character`

**值类型** -> **引用类型**，称为`装箱`；
**引用类型** -> **值类型**，称为`拆箱`。

在 jdk1.5 之前，装箱、拆箱需要我们手动干预，称为**手动装箱**、**手动拆箱**；
在 jdk1.5 之后，装箱、拆箱可以由编译器自动完成，称为**自动装箱**、**自动拆箱**。

除了 Boolean、Character 类型直接继承 Object 类，Byte、Short、Integer、Long、Float、Double 都是继承自 Number 类。

包装类对象一经创建，所封装的基本类型的值不会再改变；这一点和 String 是一样的。

**常量池**
常量池有两种：
1) `class文件常量池`：或称为"静态常量池"，用于存放编译器生成的各种`字面量`、`符号引用`，在类加载之后会放到方法区的**运行时常量池**中；
2) `运行时常量池`：或称为"动态常量池"，与静态常量池不同的是，它具有**动态性**，即可以在运行期间动态的将新的常量放入池中。

常量池的运用可以有效地减少相同常量的多次存储，减少不必要的存储空间浪费。

而动态常量池在开发中运用的最多的就是 String 的 intern() 成员方法；
并且基本类型包装类也存在运行时常量池，它们的作用和 String 是相似的。

八大基本类型的包装类中，除了`浮点型`的包装类（**Float**、**Double**）外，其他所有的包装类都存在常量池机制。

当一个 String 实例调用 intern() 方法时，首先会去查找 String 运行时常量池中是否有相同的字符串常量；
如果有，则返回常量池中该字符串的引用；如果没有，将当前对象的加入到常量池中，并返回其在常量池中的引用。

**String 的两种创建方式**
1) `String s = "www.zfl9.com";`
第一步，将字面量"www.zfl9.com"存放在 Class 文件的常量池中；
第二步，执行`String s`，新建一个 String 引用变量 s（String 类型的指针）；
第三步，将字面量"www.zfl9.com"的地址赋给引用变量 s。

在这种方式中，只创建了一个对象，即"www.zfl9.com"常量；

2) `String s = new String("www.zfl9.com");`
第一步，将字面量"www.zfl9.com"存放在 Class 文件的常量池中；
第二步，执行`new String()`，在堆中创建一个 String 对象，并使用常量"www.zfl9.com"进行初始化（拷贝构造）；
第三步，执行`String s`，新建一个 String 引用变量 s（String 类型的指针）；
第四步，将刚刚在堆中创建的匿名对象的指针赋给引用变量 s。

在这种方式中，创建了两个对象，一个在常量池中，一个在堆中；

因此，不建议使用第二种形式，会造成内存空间的浪费！

String.intern() 的例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.out;

public class Main {
    public static void main(String[] args) {
        String s1 = "www.zfl9.com";
        String s2 = "www.zfl9.com";
        String s3 = new String("www.zfl9.com");
        String s4 = new String(s3);

        out.printf("s1 == s2 -> %b\n", s1 == s2); // true
        out.printf("s1 == s3 -> %b\n", s1 == s3); // false
        out.printf("s3 == s4 -> %b\n", s3 == s4); // false

        out.printf("s1.equals(s3) -> %b\n", s1.equals(s3)); // true
        out.printf("s3.equals(s4) -> %b\n", s3.equals(s4)); // true

        out.printf("s1 == s3.intern() == s4.intern() -> %b\n",
                   s1 == s3.intern() && s1 == s4.intern()); // true
        out.printf("s1.intern() == s3.intern() -> %b\n", s1.intern() == s3.intern()); // true
        out.printf("s3.intern() == s4.intern() -> %b\n", s3.intern() == s4.intern()); // true
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:12:43]
$ javac Main.java

# root @ arch in ~/work on git:master x [16:12:56]
$ java Main
s1 == s2 -> true
s1 == s3 -> false
s3 == s4 -> false
s1.equals(s3) -> true
s3.equals(s4) -> true
s1 == s3.intern() == s4.intern() -> true
s1.intern() == s3.intern() -> true
s3.intern() == s4.intern() -> true
</script></code></pre>



好吧，有些扯远了，我们回到包装类中来，包装类有一些共同方法，以 Integer 为例：
**手动装箱**
`public Integer(int value)`
`public Integer(String s) throws NumberFormatException`：解析字符串中的 int。
**自动装箱**
`public static Integer valueOf(int i)`：基本类型的值在区间`[-128, 127]`的对象将入池。
`public static Integer valueOf(String s) throws NumberFormatException`：同上。
`public static Integer valueOf(String s, int radix) throws NumberFormatException`：同上。

`public int intValue()`：返回所包装的基本类型的值；

`public static int parseInt(String s) throws NumberFormatException`：解析字符串中的数字；
`public static int parseInt(String s, int radix) throws NumberFormatException`：同上。

手动装箱因为每次都是使用`new`创建，所以每次创建的对象都是不同的，它们生死于堆上；
而自动装箱则有点不同，它不使用`new`创建，而是使用其静态方法`valueOf()`，`valueOf()`内部维护了一个常量池；

1) 初始时，该 cache 池为空，没有任何包装类对象；
2) 当调用`valueOf(10)`方法自动装箱时，发现 cache 池中没有值等于 10 的对象，于是新建一个对象并丢入 cache 池中；
3) 当再次调用`valueOf(10)`方法自动装箱时，发现 cache 池中已有值相同的对象，于是不再创建新对象，而是将已有对象返回；
4) 但是 cache 池并不是无限大的，是有一定范围的，在 Integer 中，它被限制为只缓存区间`[-128, 127]`的对象，即一个字节表示的整数；
5) 如果传入 valueOf() 的参数不在该范围中，那么等同于手动装箱，即每次都会 new 一个新的对象出来；

除了 Integer 有所谓的 cache 池，Boolean、Byte、Short、Long、Character 也有 cache，如下：
- Short、Long 和 Integer 一样，区间都是 [-128, 127]；
- Boolean、Byte 因为它们占用的内存长度都在 1 字节之内，因此全部取值范围都被缓存；
- 而 Character 相当于无符号的 Short 整型，因此在区间 [0, 127] 的对象也将被缓存；
- 但是 Float、Double 浮点型的对象并不会被缓存，不管它们的取值范围是多少；



为什么浮点数包装类不会被丢入池中，无论它们的大小是多少？
准确原因我也不是很明确，但是我猜测是因为"整数和小数在内存中的表示是不同的"；
对于整型变量，假设它们都是有符号的，如果其值的区间在`[-128, 127]`内，那么就可以将其压缩为一个字节（有符号）来存储。

还有一点要注意：
1) 当`==`运算符的两个操作数都是`引用类型`（包装类）时，比较引用的值，不触发自动拆箱；
2) 如果其中有一个操作数是`算数表达式/数值`则触发`自动拆箱`，这时比较的是`基本类型的值`。

例子一：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.out;

public class Main {
    public static void main(String[] args) {
        Integer a1 = 40;
        Integer a2 = 40;
        Integer a3 = 0;
        Integer b1 = new Integer(40);
        Integer b2 = new Integer(40);
        Integer b3 = new Integer(0);

        out.printf("a1 == a2 -> %b\n", a1 == a2); // true
        out.printf("a1 == a2 + a3 -> %b\n", a1 == a2 + a3); // true
        out.printf("b1 == b2 -> %b\n", b1 == b2); // false
        out.printf("b1 == b2 + b3 -> %b\n", b1 == b2 + b3); // true
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:58:27]
$ javac Main.java

# root @ arch in ~/work on git:master x [16:58:42]
$ java Main
a1 == a2 -> true
a1 == a2 + a3 -> true
b1 == b2 -> false
b1 == b2 + b3 -> true
</script></code></pre>



如果你理解了前面的内容，那么这个例子就很容易理解了：
1) `a1 == a2`：a1、a2 自动装箱，值在区间 [-128, 127]，因此它们都引用同一个对象，而`==`两边的操作数都是引用类型，比较他们的引用的值，因为是同一个对象，所以返回 true；
2) `a1 == a2 + a3`：a1、a2、a3 都是自动装箱，a1 和 a2 都引用自池中的同一对象，`==`的右操作数是一个表达式，触发 a2、a3 的自动拆箱，变为`40 + 0`，即右操作数为 40，因为有一个操作数是值类型，所以触发 a1 的自动拆箱，最终比较的是`40 == 40`，返回 true；
3) `b1 == b2`：因为 b1、b2 都是手动装箱，所以他们引用的是不同的对象，因此返回 false；
4) `b1 == b2 + b3`：右操作数是一个表达式，触发自动拆箱，结果为 40，而 b1 也被触发自动拆箱，结果为 40，因此返回 true。


再来一个例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.out;

public class Main {
    public static void main(String[] args) {
        Integer a1 = 1;
        Integer a2 = 2;
        Integer a3 = 3;
        Integer a4 = 3;

        Integer a5 = 200;
        Integer a6 = 200;

        Long b1 = 3L;
        Long b2 = 2L;

        out.printf("a3 == a4 -> %b\n", a3 == a4); // true
        out.printf("a5 == a6 -> %b\n", a5 == a6); // false
        out.printf("a3 == a1 + a2 -> %b\n", a3 == a1 + a2); // true
        out.printf("a3.equals(a1 + a2) -> %b\n", a3.equals(a1 + a2)); // true
        out.printf("b1 == a1 + a2 -> %b\n", b1 == a1 + a2); // true
        out.printf("b1.equals(a1 + a2) -> %b\n", b1.equals(a1 + a2)); // false
        out.printf("b1.equals(a1 + b2) -> %b\n", b1.equals(a1 + b2)); // true
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:19:29]
$ javac Main.java

# root @ arch in ~/work on git:master x [17:19:43]
$ java Main
a3 == a4 -> true
a5 == a6 -> false
a3 == a1 + a2 -> true
a3.equals(a1 + a2) -> true
b1 == a1 + a2 -> true
b1.equals(a1 + a2) -> false
b1.equals(a1 + b2) -> true
</script></code></pre>



1) `a3 == a4`：自动装箱，比较的是引用，因为在区间 [-128, 127]，true；
2) `a5 == a6`：自动装箱，比较的是引用，因为不在区间 [-128, 127]，false；
3) `a3 == a1 + a2`：触发自动拆箱，比较的是数值，true；
4) `a3.equals(a1 + a2)`：计算`a1 + a2`时触发自动拆箱，然后再次自动装箱，因此返回 true；
5) `b1 == a1 + a2`：计算`a1 + a2`时触发自动拆箱，结果为 int 类型的值 3，b1 也因此自动拆箱，是 long 类型的值 3；然后 int -> long 自动类型转换，因此返回 true；
6) `b1.equals(a1 + a2)`：计算`a1 + a2`时触发自动拆箱，结果为 int 类型的值 3，然后再次装箱为 Integer 引用类型，因为比较的两个对象的类型不同，所以返回 false；
7) `b1.equals(a1 + b2)`：计算`a1 + b2`时触发自动拆箱，并且发生自动类型转换 int -> long，然后装箱为 Long 引用类型，因此返回 true。
