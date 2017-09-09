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
