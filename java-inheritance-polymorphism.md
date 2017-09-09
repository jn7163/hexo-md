---
title: Java 继承与多态
date: 2017-09-09 09:22:00
categories:
- java
tags:
- java
keywords: Java 继承与多态
---

> 
Java 继承与多态

<!-- more -->

## 继承的概念
面向对象的三个基本特征是：`封装`、`继承`、`多态`：
- 封装：隐藏对象的属性和实现细节，仅对外提供公共访问方式；
- 继承：子类继承父类的成员，实现代码重用；
- 多态：子类重写父类的方法，实现接口重用；


继承是类与类之间的关系，继承可以理解为一个类从另一个类获取方法和属性的过程；
Java 中类的继承使用 extends 关键字；Java 只支持单继承，不支持多继承；

> 
在 Java 中，类与类之间只能单继承，但是一个类可以同时实现多个接口，并且接口与接口之间可以多继承；

被继承的类称为父类或`基类`，继承的类称为子类或`派生类`；
派生类除了拥有基类的成员，还可以定义自己的新成员，以增强类的功能；

以下是两种典型的使用继承的场景：
1) 当你创建的新类与现有的类相似，只是多出若干成员变量或成员函数时，可以使用继承，这样不但会减少代码量，而且新类会拥有基类的所有功能；
2) 当你需要创建多个类，它们拥有很多相似的成员变量或成员函数时，也可以使用继承；可以将这些类的共同成员提取出来，定义为基类，然后从基类继承，既可以节省代码，也方便后续修改成员；

比如，我们定义了一个基类 People，并且由基类派生出 Student 类：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        People p;

        p = new People("张三", 35);
        p.print();

        p = new Student("李四", 14, 98.0f);
        p.print();
    }
}

class People {
    // 构造函数
    public People(String name, int age) {
        m_name = name;
        m_age = age;
    }

    // Getter、Setter函数
    public String getName() { return m_name; }
    public int getAge() { return m_age; }
    public void setName(String name) { m_name = name; }
    public void setAge(int age) { m_age = age; }

    // 成员函数
    public void print() {
        out.printf("%s, %d 岁, 无业游民\n", m_name, m_age);
    }

    // 成员变量
    protected String m_name;
    protected int m_age;
}

class Student extends People {
    // 构造函数
    public Student(String name, int age, float score) {
        super(name, age); // 调用直接基类People的构造函数, 若不显式调用, 则默认为super(), 即调用无参构造函数
        m_score = score;
    }

    // 新增Getter、Setter函数
    public float getScore() { return m_score; }
    public void setScore(float score) { m_score = score; }

    // 成员函数
    @Override
    public void print() {   // override 重写，多态
        out.printf("%s, %d 岁, 一名学生, 考试成绩 %.1f 分\n", m_name, m_age, m_score);
    }

    // 新增成员变量
    private float m_score;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:25:54]
$ javac Main.java

# root @ arch in ~/work on git:master x [10:26:05]
$ java Main
张三, 35 岁, 无业游民
李四, 14 岁, 一名学生, 考试成绩 98.0 分
</script></code></pre>


Java 和 C++ 一样，派生类构造函数必须调用直接基类中的构造函数；
如果不显式的在派生类构造函数中调用直接基类构造函数，那么默认调用直接基类的无参构造函数，即`super()`；
如果直接基类中没有无参构造函数，那么编译失败，所以建议在派生类的构造函数中都显式的调用直接基类构造函数！

对于需要 override 的派生类成员函数，建议都加上`@Override`伪代码，提示编译器检查 Override 的正确性；

Java 中没有 public、protected、private 继承方式之分，默认都是 public 继承，即不改变基类成员在派生类中的访问性；

**哪些成员可以被继承**
- public、protected 成员函数和成员变量可以被继承；
- private 成员函数不能被继承；private 成员变量可以被继承，但是在派生类中不可见；
- 构造函数不能被继承；
- final 类不能被继承；


> 
`static`成员：与对象无关的函数或变量，存储在`全局数据区`，相当于访问性被限制的全局函数、全局变量；
`final`类：不能被派生，并且 final 类的成员函数都隐式的声明为了 final 成员，但是成员变量不会；
`final`方法：不能被重写；`final`属性：不能被修改，只读；`final`局部变量：不能被修改，只读；


## super关键字
super 关键字与 this 类似，this 用来表示当前类的实例，super 用来表示父类；
super 用在子类中，通过`.`来获取父类的成员变量和方法；super 也可以用在子类的子类中，Java 能自动向上层类追溯；

super 关键字的功能：
- 调用名字被屏蔽的基类成员；
- 调用直接基类的构造方法（在派生类构造函数中调用，必须位于第一行，一个构造函数中只能有一个 super 语句）；


例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        Derived d = null;
        d = new Derived(); // 调用 Derived()
        d = new Derived(1); // 调用 Derived(int)
        new Derived().func();
    }
}

class Base {
    public Base() {
        out.printf("Base::Base()\n");
    }
    public Base(int i) {
        out.printf("Base::Base(int)\n");
    }

    protected String base = "Base_String";
}

class Derived extends Base {
    public Derived() {
        out.printf("Derived::Derived()\n"); // default -> super()
    }
    public Derived(int i) {
        super(i);
        out.printf("Derived::Derived(int)\n");
    }

    public void func() {
        out.printf("this -> base: %s\n", base); // "Derived_String"
        out.printf("super -> base: %s\n", super.base); // "Base_String"
    }

    private String base = "Derived_String";
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:12:23]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:12:39]
$ java Main
Base::Base()
Derived::Derived()
Base::Base(int)
Derived::Derived(int)
Base::Base()
Derived::Derived()
this -> base: Derived_String
super -> base: Base_String
</script></code></pre>


## 多态和动态绑定
在 Java 中，父类的引用可以引用父类的实例，也可以引用子类的实例；
多态存在的三个必要条件：`继承`、`重写`、`父类引用变量引用子类对象`；

绑定、静态绑定、动态绑定的概念：
1) **绑定**
绑定指的是一个方法的调用与方法所在的类（方法主体）关联起来；
对 Java 来说，绑定分为`静态绑定`和`动态绑定`；或者叫做前期绑定和后期绑定；

2) **静态绑定**
在程序执行前方法已经被绑定，针对 Java 简单的可以理解为程序编译期的绑定；
在 Java 中，被 final、static、private 修饰的方法和构造方法是静态绑定的；

3) **动态绑定**
在程序运行期间根据具体对象的类型进行方法绑定；
Java 提供了一些机制（类似 C++ 中虚函数表），可在运行期间判断对象的类型，并分别调用适当的方法；
也就是说，编译器此时依然不知道对象的实际类型以及要调用的方法主体，但方法调用机制能自己去调查，找到正确的方法主体；


例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        Master ma = new Master();
        ma.feed(new Animal(), new Food());
        ma.feed(new Dog(), new Bone());
        ma.feed(new Cat(), new Fish());
    }
}

// Master 主人
class Master {
    public void feed(Animal an, Food f) {
        an.eat(f);
    }
}

// Animal 类
class Animal {
    public void eat(Food f) {
        out.printf("我是动物, 我在吃%s\n", f.getFood());
    }
}

// Animal派生类 - Dog
class Dog extends Animal {
    @Override
    public void eat(Food f) {
        out.printf("我是小狗, 我在吃%s\n", f.getFood());
    }
}

// Animal派生类 - Cat
class Cat extends Animal {
    @Override
    public void eat(Food f) {
        out.printf("我是小猫, 我在吃%s\n", f.getFood());
    }
}

// Food 食物
class Food {
    public String getFood() { return "食物"; }
}

// Food派生类 - Bone
class Bone extends Food {
    @Override
    public String getFood() { return "骨头"; }
}

// Food派生类 - Fish
class Fish extends Food {
    @Override
    public String getFood() { return "小鱼"; }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:49:33] C:1
$ javac Main.java

# root @ arch in ~/work on git:master x [14:49:35]
$ java Main
我是动物, 我在吃食物
我是小狗, 我在吃骨头
我是小猫, 我在吃小鱼
</script></code></pre>


## instanceof运算符
多态性带来了一个问题，就是如何判断一个变量所实际引用的对象的类型；
C++ 使用`RTTI`运行时类型识别，Java 使用 instanceof 操作符；

instanceof 的语法为`obj instanceof ClassName`，运算结果为一个 bool 值；
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        Main ma = new Main();
        if (ma instanceof Object) {
            out.printf("对象ma 属于 类Object\n");
        }
        if (ma instanceof Main) {
            out.printf("对象ma 属于 类Main\n");
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:02:01]
$ javac Main.java

# root @ arch in ~/work on git:master x [15:02:15]
$ java Main
对象ma 属于 类Object
对象ma 属于 类Main
</script></code></pre>


## 向上转型和向下转型
在继承链中，我们将子类向父类转换称为“向上转型”，将父类向子类转换称为“向下转型”；
> 
向上转型非常安全，可以由编译器自动完成；向下转型有风险，需要程序员手动干预；

1) 向上转型：
很多时候，我们会将变量定义为父类的类型，却引用子类的对象，这个过程就是向上转型；
程序运行时通过动态绑定来实现对子类方法的调用，也就是多态性（动态多态）；

2) 向下转型：
然而有些时候为了完成某些父类没有的功能，我们需要将向上转型后的子类对象再转成子类，调用子类的方法，这就是向下转型；
> 
注意：不能直接将父类的对象强制转换为子类类型，只能将向上转型后的子类对象再次转换为子类类型；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        A a = new B();  // upcasting: B -> A

        if (a instanceof A) {
            out.printf("a instanceof A\n");
        }
        if (a instanceof B) {
            out.printf("a instanceof B\n");
            B b = (B)a; // downcasting: A -> B [successfully]
        }
        if (a instanceof C) {
            out.printf("a instanceof C\n");
            C c = (C)a; // downcasting: A -> C [failed]
        }
    }
}

class A {}
class B extends A {}
class C extends B {}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:34:57]
$ javac Main.java

# root @ arch in ~/work on git:master x [15:34:59]
$ java Main
a instanceof A
a instanceof B
</script></code></pre>


因为向下转型存在风险，所以在接收到父类的一个引用时，请务必使用 instanceof 运算符来判断该对象是否是你所要的子类；


## static、final
**static 关键字**
static 可用于修饰成员变量和成员函数，不能修饰局部变量（C/C++ 可以）
- 静态成员函数只能调用类的静态成员；
- 静态成员一般通过类来访问，不推荐通过对象来访问，编译器也会警告；
- 静态成员函数中不存在当前对象，因而不能使用 this，当然也不能使用 super；


**final 关键字**
在 Java 中，声明类、变量和方法时，可使用关键字 final 来修饰；
final 所修饰的数据具有“终态”的特征，表示“最终的”意思；具体规定如下：
- final 修饰的类不能被继承；
- final 修饰的方法不能被子类重写；
- final 修饰的成员变量必须在声明的同时进行赋值，或者在构造函数中进行赋值；
- final 修饰的局部变量即成为常量，只能赋值一次（可在定义的同时进行赋值，也可以在定义之后进行一次赋值）；
- final 修饰的引用类型变量不能指向其它对象；但可以改变对象的内容；


## Object类
Java 中的 Object 类是所有类的父类，它提供了以下 11 个方法：
- `public final native Class<?> getClass()`：返回一个 Class 类对象，该对象保存着类的类型信息；
- `public native int hashCode()`：返回当前对象的 Hash 值；
- `public boolean equals(Object obj)`：比较两个对象是否相等；
- `protected native Object clone() throws CloneNotSupportedException`：创建一个当前对象的拷贝；
- `public String toString()`：返回一个描述当前对象的字符串；
- `public final native void notify()`：唤醒一个在此对象监视器上等待的线程；
- `public final native void notifyAll()`：唤醒所有在此对象监视器上等待的线程；
- `public final native void wait(long timeout) throws InterruptedException`：wait 方法会让当前线程等待直到另外一个线程调用对象的 notify 或 notifyAll 方法，或者超过参数设置的 timeout 超时时间；
- `public final void wait(long timeout, int nanos) throws InterruptedException`：跟 wait(long timeout) 方法类似，多了一个 nanos 参数，这个参数表示额外时间（以纳秒为单位，范围是0-999999），所以超时的时间还需要加上 nanos 纳秒；
- `public final void wait() throws InterruptedException`：和前两个 wait 方法一样，只不过没有超时时间，即一直等待；
- `protected void finalize() throws Throwable {}`：该方法的作用是实例被垃圾回收器回收的时候触发的操作，默认为空；


**getClass() 方法**
返回调用对象的 Class 对象（Class 类位于`java.lang.Class`），和 C++ 中的`typeid()`函数是一个概念；
Class 类可以理解为 C++ 中的 type_info 类，里面保存了类的类型信息，如类的名称、类的属性等；

有 3 种方式可以获取一个 Class 对象：
- `getClass()`成员函数：在运行期间获取调用对象的实际 Class 对象；
- `class`静态成员变量：该静态属性保存着对应的 Class 对象（编译期间确定）；
- `forName(String class_name)`静态方法：返回名字为 class_name 的类的 Class 对象（检查异常 ClassNotFoundException）；


Class 类的常用方法：
1、`getName()`：返回类的名称；
2、`toString()`：返回类的描述字符串；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String args[]) {
        Main obj = new Main();
        out.printf("obj: %s\n", obj.getClass().getName());
        out.printf("obj: %s\n", obj.getClass().toString());
        out.printf("obj: %s\n", Main.class.getName());

        try {
            Class<?> c = Class.forName("java.lang.Integer");
            out.printf("find success: %s\n", c.getName());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:36:03]
$ javac Main.java

# root @ arch in ~/work on git:master x [17:36:14]
$ java Main
obj: Main
obj: class Main
obj: Main
find success: java.lang.Integer
</script></code></pre>


**hashCode() 和 equals()**
在 Java 中 hashCode() 的实现总是伴随着 equals()，他们是紧密配合的，你要是自己设计了其中一个，就要设计另外一个；
当然在多数情况下，这两个方法是不用我们考虑的，直接使用默认方法就可以帮助我们解决很多问题；但是在有些情况，我们必须要自己动手来实现它，才能确保程序更好的运作；

对于 equals()，我们必须遵循如下规则：
- 对称性：如果 x.equals(y) 返回是 true，那么 y.equals(x) 也应该返回是 true；
- 反射性：x.equals(x) 必须返回是 true；
- 类推性：如果 x.equals(y) 返回是 true，而且 y.equals(z) 返回是 true，那么 z.equals(x) 也应该返回是 true；
- 一致性：如果 x.equals(y) 返回是 true，只要 x 和 y 内容一直不变，不管你重复 x.equals(y) 多少次，返回都是 true；


任何情况下，x.equals(null)，永远返回是 false；x.equals(和x不同类型的对象) 永远返回是 false；

对于 hashCode()，我们应该遵循如下规则：
1. 在一个应用程序执行期间，如果一个对象的 equals 方法做比较所用到的信息没有被修改的话，则对该对象调用 hashCode 方法多次，它必须始终如一地返回同一个整数；
2. 如果两个对象根据 equals(Object o) 方法是相等的，则调用这两个对象中任一对象的 hashCode 方法必须产生相同的整数结果；
3. 如果两个对象根据 equals(Object o) 方法是不相等的，则调用这两个对象中任一个对象的 hashCode 方法，不要求产生不同的整数结果；但如果能不同，则可能提高散列表的性能；


至于两者之间的关联关系，我们只需要记住如下即可：
1) 如果 x.equals(y) 返回 true，那么 x 和 y 的 hashCode() 必须相等；
2) 如果 x.equals(y) 返回 false，那么 x 和 y 的 hashCode() 有可能相等，也有可能不等；

如果你还是不清楚它们之间的关系，那么请看图：
![hashCode 和 equals 的联系](/images/java-hashcode-equals.png)


整个处理流程：
1、判断两个对象的 hashcode 是否相等，若不等，则认为两个对象不等，完毕，若相等，则比较 equals；
2、若两个对象的 equals 不等，则可以认为两个对象不等，否则认为他们相等；

**clone() 方法**
clone() 方法用于创建并返回当前对象的一份拷贝；
对于任何对象 x，表达式`x.clone() != x`为 true，`x.clone().getClass() == x.getClass()`也为 true；

Object 类的 clone 方法是一个 protected 的 native 方法；
由于 Object 本身没有实现 Cloneable 接口，所以不重写 clone 方法并且进行调用的话会发生 CloneNotSupportedException 异常；

**toString() 方法**
toString 方法的结果应该是一个简明但易于读懂的字符串；建议 Object 所有的子类都重写这个方法；

默认实现，即输出`"类的名字"@"十六进制的实例哈希码"`：
<pre><code class="language-java line-numbers"><script type="text/plain">public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
</script></code></pre>
