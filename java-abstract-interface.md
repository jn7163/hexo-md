---
title: Java 抽象类和接口
date: 2017-09-09 18:29:00
categories:
- java
tags:
- java
keywords: Java 抽象类和接口
---

> 
Java 抽象类和接口

<!-- more -->

## 内部类
在 Java 中，允许在一个`类`、`方法`、`语句块`的内部定义另一个类，称为`内部类(Inner Class)`，有时也称为`嵌套类(Nested Class)`；

内部类和外层封装它的类之间存在逻辑上的所属关系，一般只用在定义它的类或语句块之内，实现一些没有通用意义的功能逻辑，在外部引用它时必须给出完整的名称；

使用内部类的主要原因有：
- 内部类可以访问外部类中的数据，包括私有的数据；
- 内部类可以对同一个包中的其他类隐藏起来；
- 当想要定义一个回调函数且不想编写大量代码时，使用`匿名内部类`比较便捷；
- 内部类的使用可以减少类的命名冲突；


例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Outer {
    private int cnt = 0;
    private void func() { out.printf("Outer::func()\n"); }

    public class Inner {
        public void func() {
            out.printf("Outer::cnt = %d\n", cnt);
            cnt++;
            out.printf("Outer::cnt = %d\n", cnt);

            out.printf("call Outer::func()\n");
            Outer.this.func(); /* 调用外部类 Outer 的 func() 方法 */
            /* this.func(); 调用内部类 Inner 的 func() 方法 */
            /* func();      调用内部类 Inner 的 func() 方法 */
        }
    }

    public static void main(String[] args) {
        Outer outer = new Outer();  /* 先实例化外部类 Outer */
        Outer.Inner inner = outer.new Inner(); /* 使用 outer.new 操作符实例化内部类 Inner */
        inner.func();
        // or
        out.println("-------------------");
        Outer.Inner inner_ = new Outer().new Inner(); /* 创建匿名外部类 */
        inner_.func();
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:12:08]
$ javac Outer.java

# root @ arch in ~/work on git:master x [20:12:22]
$ ll
total 12K
-rw-r--r-- 1 root root 1021 Sep  9 20:12  Outer.class
-rw-r--r-- 1 root root  807 Sep  9 20:12 'Outer$Inner.class'
-rw-r--r-- 1 root root 1011 Sep  9 20:11  Outer.java

# root @ arch in ~/work on git:master x [20:12:24]
$ java Outer
Outer::cnt = 0
Outer::cnt = 1
call Outer::func()
Outer::func()
-------------------
Outer::cnt = 0
Outer::cnt = 1
call Outer::func()
Outer::func()
</script></code></pre>


这段代码定义了一个外部类 Outer，它包含了一个内部类 Inner；
编译之后会生成两个 .class 文件：`Outer.class`和`Outer$Inner.class`；也就是说，内部类会被编译成独立的字节码文件；

> 
内部类是一种编译器现象，与虚拟机无关；
编译器会把内部类翻译成用`$`分隔外部类名与内部类名的常规类文件，而虚拟机则对此一无所知；

注意：必须先有外部类的对象才能生成内部类的对象，因为内部类需要访问外部类中的成员变量，成员变量必须实例化才有意义；

**内部类的分类**
内部类可分为：成员式内部类、静态内部类、局部内部类、匿名内部类；

**成员式内部类**
在外部类内部直接定义（不在方法内部或代码块内部）的类就是`成员式内部类`，它可以直接使用外部类的所有变量和方法（包括 private 属性的成员）；外部类要想访问内部类的成员变量和方法，则需要通过内部类的对象来获取；

成员式内部类就如同外部类的一个普通成员，因此可以像成员函数那样使用外部类的 public、protected、private 成员；
正因如此，成员式内部类的创建需要建立在已实例化的外部类对象上，因为需要使用对象成员，不实例化就无法在内部类中访问了；

成员式内部类和普通成员一样，支持 public、protected、private、[default] 访问修饰符；支持 static、final、abstract 属性修饰符；
若有 static 修饰符，就为类级（即`静态内部类`），否则为对象级；类级可以通过外部类直接访问，对象级需要先生成外部的对象后才能访问；
对于非 static 修饰的内部类，类内不能声明任何 static 成员；内部类之间可以互相调用，就如同类中的不同方法可以互相调用一样；

成员式内部类的实例化：
1、对于非 static 成员式内部类，需要先创建外部类的对象`outObj`，再由`outObj.new`操作符创建内部类的对象；
2、对于 static 成员式内部类，不需要创建外部类对象，可以直接使用`new`创建，只不过内部类的类名需要带上外部类，如`Outer.Inner inner = new Outer.Inner(param ...);`；
3、当内部类与其外部类中存在同名非静态成员时，默认调用的是内部类中的同名成员，使用`this.var`或`this.func()`显式调用内部类中的成员，使用`outClass.this.var`或`outClass.this.func()`显式调用外部类中的成员；
4、当内部类与其外部类中存在同名静态成员时，不需要使用 this，即`outClass.var`、`outClass.func()`；

在 Outer 类中定义一个没有通用意义的 private 内部类 Inner，用于实现某些功能：
<pre><code class="language-java line-numbers"><script type="text/plain">class Outer {
    private class Inner {
        // TODO
    }
    public void func() {
        Inner inner = new Inner();
        // TODO
    }
}
</script></code></pre>



**局部内部类**
`局部内部类(Local class)`是定义在`代码块`中的类；
- 局部类只在定义了它们的代码块中可见；
- 局部类不可以是 static 的，里边也不能定义 static 成员；
- 局部类不可以被 public、private、protected 修饰；
- 局部类可以被 abstract 修饰；
- 局部类会捕获外部作用域中的变量，`值传递`，`final`修饰；



对于最后一点，需要了解局部内部类的实现细节：

先定义一个局部类 Student：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        String name = "Otokaze";
        int age = 18;
        float score = 120.0f;

        class Student {
            public void print() {
                out.printf("name: %s, age: %d, score: %.1f\n", name, age, score);
            }
        }

        new Student().print();
    }
}
</script></code></pre>



编译之后生成两个独立的 .class 文件：
<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:30:45]
$ javac Main.java

# root @ arch in ~/work on git:master x [9:30:48]
$ ll
total 12K
-rw-r--r-- 1 root root 903 Sep 10 09:30 'Main$1Student.class'
-rw-r--r-- 1 root root 423 Sep 10 09:30  Main.class
-rw-r--r-- 1 root root 390 Sep 10 09:27  Main.java

# root @ arch in ~/work on git:master x [9:30:50]
$ java Main
name: Otokaze, age: 18, score: 120.0
</script></code></pre>



表面上 Student 类使用的变量 name、age、score 和 main() 中的 name、age、score 是同一个变量；
但是实际上并不是，Student 中获取的变量只是外部作用域 main() 中变量的一个 final 拷贝；

它们看起来类似：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        String name = "Otokaze";
        int age = 18;
        float score = 120.0f;

        new Student(name, age, score).print();
    }
}

class Student {
    public Student(String _name, int _age, float _score) {
        name = _name;
        age = _age;
        score = _score;
    }

    public void print() {
        out.printf("name: %s, age: %d, score: %.1f\n", name, age, score);
    }

    private final String name;
    private final int age;
    private final float score;
}
</script></code></pre>



因此，如果在 Student 中尝试修改自动捕获到的外部变量，将导致一个编译错误，因为 final 变量是不允许被修改的；

再看这个例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        String name = "Otokaze";
        int age = 18;
        float score = 120.0f;

        class Student {
            public void print() {
                out.printf("name: %s, age: %d, score: %.1f\n", name, age, score);
            }

            public void func(int n) {
                out.printf("[Student] n = %d\n", n);
                n++;
                out.printf("[Student] n = %d\n", n);
            }
        }

        new Student().print();

        int n = 0;
        new Student().func(n);
        out.printf("[main] n = %d\n", n);
    }
}
</script></code></pre>



<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:44:18]
$ javac Main.java

# root @ arch in ~/work on git:master x [9:44:20]
$ java Main
name: Otokaze, age: 18, score: 120.0
[Student] n = 0
[Student] n = 1
[main] n = 0
</script></code></pre>



因为传递的参数 n 位于 Student 定义之后，所以没有被 Student 捕获为 final 变量；
因此在 Student 内部修改 n 的值并不会导致编译错误，只不过这个 n 与外部的 n 是没有关联的；

**匿名内部类**
1、匿名内部类**必须继承一个类或者实现一个接口**，但两者不可兼得，同时也只能继承一个类或者实现一个接口；
2、匿名内部类没有构造函数，因为没有类名因此无法实现构造函数；可以使用初始化代码块替代构造函数；
3、匿名内部类中不能定义任何 static 成员；
4、匿名内部类是局部内部类的一种特殊形式，因此局部内部类的所有限制同样对匿名内部类有效；
5、匿名内部类不能是 abstract 的，它必须要实现继承的类或者实现的接口的所有抽象方法；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

abstract class Bird {
    public abstract void fly();
}

public class Main {
    public static void main(String[] args) {
        new Bird() {
            public void fly() {
                out.printf("%s能飞 %d 米\n", "燕子", 10000);
            }
        }.fly();
    }
}
</script></code></pre>



<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:01:42] C:1
$ javac Main.java

# root @ arch in ~/work on git:master x [10:01:44]
$ java Main
燕子能飞 10000 米
</script></code></pre>



匿名内部类只是局部内部类的一种，所以局部内部类的限制同样适用于匿名内部类：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        String name = "Otokaze";
        int age = 19;
        float score = 143.0f;

        new Main() {
            public void print() {
                out.printf("name: %s, age: %d, score: %.1f\n", name, age, score);
                /*
                 * name = "Unnamed";
                 * age = 0;
                 * score = 0.0f;
                 * 这些操作都是不被允许的，会导致编译错误
                 */
            }
        }.print();
    }
}
</script></code></pre>



## 抽象类
在自上而下的继承层次结构中，位于上层的类更具有通用性，甚至可能更加抽象；
从某种角度看，祖先类更加通用，它只包含一些最基本的成员，人们只将它作为派生其他类的基类，而不会用来创建对象；
甚至，你可以只给出方法的定义而不实现，由子类根据具体需求来具体实现；

这种**只给出方法定义而不具体实现的方法**被称为`抽象方法`，抽象方法是没有方法体的，在代码的表达上就是没有“{}”；
如果一个类包含一个或多个抽象方法就必须被声明为`抽象类`；抽象类不能被实例化，抽象方法必须在子类中被实现；

使用`abstract`修饰符来表示`抽象方法`和`抽象类`；抽象类除了包含抽象方法外，还可以包含具体的变量和具体的方法；

> 
一个类即使没有任何抽象方法，也可以被声明为抽象类，防止被实例化；

抽象类不能直接使用，必须用子类去实现抽象类，然后使用其子类的实例；
可以创建一个变量，其类型是一个抽象类，并让它指向具体子类的一个实例，即多态的使用；
特别注意，`abstract`不能和`static`一起使用，即不能有抽象静态方法，也不能有抽象构造函数；

如果一个抽象基类拥有多个抽象方法，那么继承他的派生类也必须实现所有抽象方法，否则不能创建对象；
当然如果没有实现所有的抽象方法，也可以将派生类声明为抽象类，让它的派生类去实现剩下的抽象方法；

典型的错误：抽象类一定包含抽象方法；但是反过来说“包含抽象方法的类一定是抽象类”就是正确的；
事实上，抽象类可以是一个完全正常实现的类；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public final class Main {
    public static void main(String[] args) {
        People stu = new Student("Otokaze", 18, 111.5f);
        stu.work();
    }
}

abstract class People {
    public People(String name, int age) {
        m_name = name;
        m_age = age;
    }

    public String getName() {
        return m_name;
    }
    public int getAge() {
        return m_age;
    }
    public void setName(String name) {
        m_name = name;
    }
    public void setAge(int age) {
        m_age = age;
    }

    public abstract void work();

    protected String m_name;
    protected int m_age;
}

class Student extends People {
    public Student(String name, int age, float score) {
        super(name, age);
        m_score = score;
    }

    public float getScore() {
        return m_score;
    }
    public void setScore(float score) {
        m_score = score;
    }

    @Override
    public void work() {
        out.printf("name: %s, age: %d, score: %.1f\n", m_name, m_age, m_score);
    }

    private float m_score;
}
</script></code></pre>



<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:51:10]
$ javac Main.java

# root @ arch in ~/work on git:master x [10:51:16]
$ java Main
name: Otokaze, age: 18, score: 111.5
</script></code></pre>



## 接口
在抽象类中，可以包含一个或多个抽象方法；但在`接口(interface)`中，所有的方法必须都是抽象的，不能有方法体，它比抽象类更加“抽象”；

接口使用`interface`关键字来声明，接口可以看做是一种特殊的抽象类，它可以指定一个类必须做什么，而不是规定它如何去做；

现实中也有很多接口的实例，比如说串口电脑硬盘，Serial ATA 委员会指定了 Serial ATA 2.0 规范，这种规范就是接口；
Serial ATA 委员会不负责生产硬盘，只是指定通用的规范；
希捷、日立、三星等生产厂家会按照规范生产符合接口的硬盘，这些硬盘就可以实现通用化；
如果正在用一块 160G 日立的串口硬盘，现在要升级了，可以购买一块 320G 的希捷串口硬盘，安装上去就可以继续使用了；

**接口是若干`常量`和`抽象方法`的集合**
接口中声明的`成员变量`默认都是`public static final`的，必须显示的初始化；因而在常量声明时可以省略这些修饰符；
接口中声明的`成员函数`默认都是`public abstract`的，不能提供任何方法体，需要让实现接口的子类去具体定义；

**为什么使用接口**
接口是`可插入性`的保证；在一个继承链中的任何一个类都可以实现一个接口，这个接口会影响到此类的所有子类，但不会影响到此类的任何父类；此类将不得不实现这个接口所规定的方法，而子类可以从此类自动继承这些方法，这时候，这些子类具有了可插入性；

我们关心的不是哪一个具体的类，而是这个类是否实现了我们需要的接口；

接口提供了关联以及方法调用上的可插入性，软件系统的规模越大，生命周期越长，接口使得软件系统的灵活性和可扩展性，可插入性方面得到保证；

接口在面向对象的 Java 程序设计中占有举足轻重的地位；事实上在设计阶段最重要的任务之一就是设计出各部分的接口，然后通过接口的组合，形成程序的基本框架结构；

**接口的使用**
接口的使用与类的使用有些不同；在需要使用类的地方，会直接使用 new 关键字来构建一个类的实例，但接口不可以这样使用，因为接口不能直接使用 new 关键字来构建实例；

接口必须通过类来`实现(implements)`它的抽象方法，然后再实例化类；类实现接口的关键字为`implements`；
如果一个类不能实现该接口的所有抽象方法，那么这个类必须被定义为`抽象类`；

不允许创建接口的实例，但允许定义接口类型的引用变量，该变量指向了实现接口的类的实例；

**接口和抽象类、普通类的共同点**
目前看来和抽象类差不多；确实如此，接口本就是从抽象类中演化而来的，因而除特别规定，接口享有和类同样的“待遇”；
比如一个文件中可以定义多个类或接口，但最多只能有一个 public 的类或接口，如果有则源文件必须取和 public 的类或接口相同的名字；

但是接口并不是类，编写接口的方式和类很相似，但是它们属于不同的概念；类描述对象的属性和方法；接口则包含类要实现的方法；

除非实现接口的类是抽象类，否则该类要定义接口中的所有方法；
接口无法被实例化，但是可以被实现；一个实现接口的类，必须实现接口内所描述的所有方法，否则就必须声明为抽象类；

**接口的特性**
但是接口有自己的一些特性，归纳如下：
1) 接口中每一个方法也是隐式抽象的，接口中的方法会被隐式的指定为`public abstract`，并且只能是`public abstract`；
2) 接口中可以含有变量，但是接口中的变量会被隐式的指定为`public static final`静态常量，并且只能是`public static final`；
3) 接口中没有构造方法，不能被实例化；
4) 一个接口不实现另一个接口，但可以继承多个其他接口；`接口`的`多继承`特点弥补了`类`的`单继承`；
5) **一个类只能继承一个父类，但却可以实现多个接口**；实现接口使用关键字`implements`；

**接口和抽象类的区别**
1) 抽象类本质还是一个类，但是接口不是类；
2) 抽象类可以有构造函数，但是接口没有构造函数；
3) 抽象类可以有具体的方法，但是接口中只能有抽象方法；
4) 抽象类中的成员变量可以是各种修饰符的，但是接口中的成员变量只能是 public static final 的；
5) 抽象类中可以有 static 代码块和 static 方法，但是接口中不能有 static 代码块和 static 方法；
6) 一个类只能继承一个抽象类，而一个类却可以实现多个接口；

**接口的声明格式**
`public/[default] interface 接口名称 [extends 接口1[, 接口2, ...]] { 声明变量、抽象方法 }`

**类实现接口的格式**
`[修饰符] class 类名 [extends 父类] implements 接口1[, 接口2, ...] { 实现抽象方法 }`

**重写接口中声明的方法**时，需要注意以下规则：
1) 类在实现接口的方法时，不能抛出`强制性异常`，只能在`接口`中，或者`继承接口的抽象类`中抛出该强制性异常；
2) 类在重写方法时要保持一致的方法名，并且应该保持相同或者相兼容的返回值类型；即应遵循`方法重写`的规则；

**标记接口**
标识接口是没有任何方法和属性的接口，它仅仅表明它的类属于一个特定的类型，供其他代码来测试允许做一些事情；
标识接口作用：简单形象的说就是给某个对象打个标（盖个戳），使对象拥有某个或某些特权；

**没有任何方法的接口被称为标记接口**；标记接口主要用于以下两种目的：
1) 建立一个公共的父接口；
2) 向一个类添加数据类型；
这种情况是标记接口最初的目的，实现标记接口的类不需要定义任何接口方法(因为标记接口根本就没有方法)，但是该类通过多态性变成一个接口类型；

**如何选择抽象类和接口**
接口和抽象类各有优缺点，在接口和抽象类的选择上，必须遵守这样一个原则：
1) 行为模型应该总是通过接口而不是抽象类定义，所以通常是优先选用接口，尽量少用抽象类；
2) 选择抽象类的时候通常是如下情况：需要定义子类的行为，又要为子类提供通用的功能；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Pupil s = new Pupil("小明", 7, 50);
        s.say();
        s.exam();
    }
}

interface People {
    public abstract String getName();
    public abstract int getAge();
    public abstract void setName(String name);
    public abstract void setAge(int age);

    public abstract void say();
}

interface Student {
    public abstract float getScore();
    public abstract void setScore(float score);

    public abstract void exam();
}

class Pupil implements People, Student {
    public Pupil() {
        m_name = "Unnamed";
        m_age = 0;
        m_score = 0.0f;
    }
    public Pupil(String name, int age, float score) {
        m_name = name;
        m_age = age;
        m_score = score;
    }

    @Override
    public String getName() { return m_name; }
    @Override
    public int getAge() { return m_age; }
    @Override
    public float getScore() { return m_score; }
    @Override
    public void setName(String name) { m_name = name; }
    @Override
    public void setAge(int age) { m_age = age; }
    @Override
    public void setScore(float score) { m_score = score; }

    @Override
    public void say() {
        out.printf("大家好，我叫%s，今年 %d 岁，是一名一年级的小学生。\n", m_name, m_age);
    }

    @Override
    public void exam() {
        out.printf("我正在考试，请保持安静 ... \n");
    }

    private String m_name;
    private int m_age;
    private float m_score;
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:28:32]
$ javac Main.java

# root @ arch in ~/work on git:master x [13:28:47]
$ java Main
大家好，我叫小明，今年 7 岁，是一名一年级的小学生。
我正在考试，请保持安静 ...
</script></code></pre>
