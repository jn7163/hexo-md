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

`this`可用于`委托构造`，语法为`this(param_list);`；
所谓的委托构造，就是在构造函数中调用本类中的其他构造函数来完成对象的初始化工作；
- 委托语句必须位于构造函数的首行；
- 只能在构造函数中使用委托构造；
- 一个构造函数中最多只能有一个委托构造语句；


## 函数重载、重写
`同一个类`中的多个函数可以有`相同的函数名`，只要它们的`参数列表不同`就可以，这被称为`方法重载(method overloading)`；

参数列表又叫参数签名，包括参数的`类型`、参数的`个数`和参数的`顺序`，只要有一个不同就叫做参数列表不同；

**`overload`、`override`区别**
`函数重载（overload）`
**同一作用域中**，函数有`同样的名称`，但是`参数列表不相同`的情形，这样的同名不同参数的函数之间，互相称之为重载函数；

基本条件：
- 函数名必须相同；
- 函数参数必须不相同；
- 函数返回值可以相同，也可以不相同；
- 函数的访问修饰符可以不同；
- 函数的检查异常可以不同；


`函数重写（override）`
子类重新定义父类中有`相同名称`和`参数`的`虚函数`，主要在`继承关系`中出现；
> 
Java 中的成员函数默认为虚函数，除非显式声明为 final 函数；

基本条件：
- 必须存在继承关系，被重写函数和重写函数分别位于基类和派生类中；
- 参数列表必须完全与被重写方法的相同；
- 返回类型必须完全与被重写方法的返回类型相同；
- 访问级别的限制性一定不能比被重写方法的强；
- 访问级别的限制性可以比被重写方法的弱；
- 重写方法一定不能抛出新的检查异常或比被重写的方法声明的检查异常更广泛的检查异常；
- 重写的方法能够抛出更少或更有限的异常；
- 不能重写被标示为 final 的方法；
- 如果不能继承一个方法，则不能重写这个方法；
- static方法不能被重写，构造函数也不能被重写，因为重写的基本条件是被重写函数能够被继承；


重载和重写的本质：
- 重载是一个`编译期概念`、重写是一个`运行期概念`；
- 重载遵循所谓“编译期绑定”，即在编译时根据参数变量的类型判断应该调用哪个函数；
- 重写遵循所谓“运行期绑定”，即在运行的时候，根据引用变量所指向的实际对象的类型来调用函数；
- 重载是`编译时多态`，即`静态多态`；重写是`运行时多态`，即`动态多态`；


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
虽然 Java 语言是典型的面向对象编程语言，但其中的八种基本数据类型并不支持面向对象编程，基本类型的数据不具备“对象”的特性 —— 不携带属性、没有方法可调用；沿用它们只是为了迎合人类根深蒂固的习惯，并的确能简单、有效地进行常规数据处理；

这种借助于非面向对象技术的做法有时也会带来不便，比如引用类型数据均继承了 Object 类的特性，要转换为 String 类型（经常有这种需要）时只要简单调用 Object 类中定义的 toString() 即可，而基本数据类型转换为 String 类型则要麻烦得多；

为解决此类问题 ，Java 为每种基本数据类型分别设计了对应的类，称之为`包装类(Wrapper Classes)`；

基本数据类型对应的包装类：
- `byte`：`Byte`
- `short`：`Short`
- `int`：`Integer`
- `long`：`Long`
- `float`：`Float`
- `double`：`Double`
- `char`：`Character`
- `boolean`：`Boolean`
- `void`：`Void`


`包装类`是一种`引用类型`，而`基本数据类型`是一种`值类型`；

除了`Boolean`、`Character`类型直接继承`Object`类，`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`都是继承`Object`的子类`Number`；

每个包装类的对象可以封装一个相应的基本类型的数据，并提供了其它一些有用的方法；包装类对象一经创建，所封装的基本类型数据的值不会改变；

基本类型和对应的包装类可以相互装换：
- 由基本类型向对应的包装类转换称为`装箱`，例如把 int 包装成 Integer 类的对象；
- 包装类向对应的基本类型转换称为`拆箱`，例如把 Integer 类的对象重新简化为 int；


包装类共同的方法，以 Integer 类为例：
- `Integer(int i)`：构造函数；
- `Integer(String s)`：构造函数，如`new Integer("35.24");`；
- `int intValue()`：成员方法，返回基本类型的值；
- `static int parseInt(String s)`：静态方法，从字符串中解析 int；
- `static int parseInt(String s, int radix)`：静态方法，从字符串中解析 int，并指定字符串中的数值进制；
- `int hashCode()`：成员方法，返回 hashCode 值；
- `boolean equals(Object obj)`：成员方法，比较两个对象；
- `String toString()`：成员方法，转换为字符串形式；


手动装拆箱例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        int m = 100;
        String n = "100";
        Integer Int_m = new Integer(m);
        Integer Int_n = new Integer(n);

        out.printf("Int_m == Int_n ? %b\n", Int_m == Int_n); // false
        out.printf("Int_m.intValue() == Int_n.intValue() ? %b\n", Int_m.intValue() == Int_n.intValue()); // true
        out.printf("Int_m.equals(Int_n) ? %b\n", Int_m.equals(Int_n)); // true
        out.printf("Int_n.equals(Int_m) ? %b\n", Int_n.equals(Int_m)); // true
        out.printf("Int_m == new Integer(100) ? %b\n", Int_m == new Integer(100)); // false

        String arr[] = {"123", "10101", "67", "af", "P"};
        try {
            out.printf("\"%s\" -> %d\n", arr[0], Integer.parseInt(arr[0], 10));
            out.printf("\"%s\"-> %d\n", arr[1], Integer.parseInt(arr[1], 2));
            out.printf("\"%s\" -> %#o\n", arr[2], Integer.parseInt(arr[2], 8));
            out.printf("\"%s\" -> %#x\n", arr[3], Integer.parseInt(arr[3], 16));
            out.printf("\"%s\" -> %d\n", arr[4], Integer.parseInt(arr[4], 10));
        } catch (NumberFormatException e) {
            out.printf("Integer.parseInt() convert fail\n");
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:25:51]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:26:15]
$ java Main
Int_m == Int_n ? false
Int_m.intValue() == Int_n.intValue() ? true
Int_m.equals(Int_n) ? true
Int_n.equals(Int_m) ? true
Int_m == new Integer(100) ? false
"123" -> 123
"10101"-> 21
"67" -> 067
"af" -> 0xaf
Integer.parseInt() convert fail
</script></code></pre>


**自动装箱和拆箱**
上面的例子中需要手动实例化一个包装类，称为`手动装箱拆箱`；jdk1.5 之前必须手动装拆箱；
jdk1.5 之后可以`自动装箱拆箱`，也就是在进行基本数据类型和对应的包装类转换时，系统将自动进行，这将大大方便程序员的代码书写；
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        Integer m = 100;
        int n = m;
        out.printf("m = %d, n = %d\n", m, n);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:41:43]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:41:46]
$ java Main
m = 100, n = 100
</script></code></pre>


> 
`自动装箱`是通过调用包装器的`valueOf()`方法实现的，而`自动拆箱`是通过调用包装器的`xxxValue()`方法实现的（xxx 代表对应的基本数据类型）

**Integer的valueOf()方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
</script></code></pre>


> 
`IntegerCache.low`为 -128，`IntegerCache.high`为 127；

也就是说，如果 i 的值在区间`[-128, 127]`内，则返回缓存中已存在的对象的引用，否则新建一个对象；
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        Integer n1 = 100;
        Integer n2 = 100;
        Integer n3 = 200;
        Integer n4 = 200;

        out.printf("n1 == n2 ? %b\n", n1 == n2); // true
        out.printf("n3 == n4 ? %b\n", n3 == n4);  // false
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:17:59]
$ javac Main.java

# root @ arch in ~/work on git:master x [15:18:03]
$ java Main
n1 == n2 ? true
n3 == n4 ? false
</script></code></pre>


float、double、boolean、byte、short、int、long、char 的 valueOf() 实现：
<pre><code class="language-java line-numbers"><script type="text/plain">public static Float valueOf(float f) {
    return new Float(f);
}

public static Double valueOf(double d) {
    return new Double(d);
}

public static final Boolean TRUE = new Boolean(true);
public static final Boolean FALSE = new Boolean(false);
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}

public static Byte valueOf(byte b) {
    final int offset = 128;
    return ByteCache.cache[(int)b + offset];
}

public static Short valueOf(short s) {
    final int offset = 128;
    int sAsInt = s;
    if (sAsInt >= -128 && sAsInt <= 127) { // must cache
        return ShortCache.cache[sAsInt + offset];
    }
    return new Short(s);
}

public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}

public static Long valueOf(long l) {
    final int offset = 128;
    if (l >= -128 && l <= 127) { // will cache
        return LongCache.cache[(int)l + offset];
    }
    return new Long(l);
}

public static Character valueOf(char c) {
    if (c <= 127) { // must cache
        return CharacterCache.cache[(int)c];
    }
    return new Character(c);
}
</script></code></pre>


当“==”运算符的两个操作数都是包装器类型的引用时，比较指向的是否是同一个对象；
而如果其中有一个操作数是`表达式（即包含算术运算）`则比较的是`数值`（`触发自动拆箱`）；
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        Integer i1 = 1;
        Integer i2 = 2;
        Integer i3 = 3;
        Integer i4 = 3;

        Integer i5 = 231;
        Integer i6 = 231;

        Long l1 = 3l;
        Long l2 = 2l;

        out.printf("i3 == i4 ? %b\n", i3 == i4); // true 比较refer
        out.printf("i5 == i6 ? %b\n", i5 == i6); // false 比较refer
        out.printf("i3 == (i1 + i2) ? %b\n", i3 == (i1 + i2));// true 比较数值 [自动拆箱]
        out.printf("i3.equals(i1 + i2) ? %b\n", i3.equals(i1 + i2));// true 比较refer的值 [自动装箱]
        out.printf("l1 == (i1 + i2) ? %b\n", l1 == (i1 + i2));// true 比较数值 [自动拆箱]
        out.printf("l1.equals(i1 + i2) %b\n", l1.equals(i1 + i2));// false 比较refer [引用类型不同]
        out.printf("l1.equals(i1 + l2) %b\n", l1.equals(i1 + l2));// true 自动类型转换 int -> long
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:30:15] C:130
$ javac Main.java

# root @ arch in ~/work on git:master x [16:30:17]
$ java Main
i3 == i4 ? true
i5 == i6 ? false
i3 == (i1 + i2) ? true
i3.equals(i1 + i2) ? true
l1 == (i1 + i2) ? true
l1.equals(i1 + i2) false
l1.equals(i1 + l2) true
</script></code></pre>
