---
title: Java 泛型编程
date: 2017-09-10 13:59:00
categories:
- java
tags:
- java
keywords: Java 泛型编程 generic paradigm
---

> 
Java 泛型编程

<!-- more -->

## 泛型的概念
Java 泛型（generics）是 JDK 5 中引入的一个新特性，泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型；

泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数；
Java 泛型以 C++ 中的模板为参照，有 C++ 模板知识的小伙伴可以很快的熟悉 Java 中的泛型编程；

类型参数只能是引用类型，不能是基本类型，但是传递基本类型的实参并不会报错，因为会被自动装箱为对应的包装类；

和 C++ 一样，类型参数使用`<>`尖括号包围，并且它的位置是放在函数返回值类型之前，Java 中的泛型不需要模板头；

## 泛型方法
我们可以使用泛型来设计一个通用数组打印函数：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        print(new Integer[] {1, 2, 3, 4, 5});
        print(new Character[] {'B', 'A', 'I', 'D', 'U'});
        print(new String[] {"C", "C++", "Java", "Python", "Perl"});
        print(new Float[] {});
        print(new Double[] {1.1, 2.2, 3.3, 4.4, 5.5, null, 6.6, 7.7, 8.8, null, 9.9});
    }

    public static <T> void print(T[] array) {
        if (array.length == 0) {
            out.printf("Array is empty\n");
            return;
        }

        out.printf("Array[%d] = { ", array.length);
        for (T elem : array) {
            if (elem == null) {
                out.printf("null, ");
                continue;
            }
            out.printf("%s, ", elem.toString());
        }
        out.printf("\b\b }\n");
    }
}
</script></code></pre>



<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:33:29]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:33:57]
$ java Main
Array[5] = { 1, 2, 3, 4, 5 }
Array[5] = { B, A, I, D, U }
Array[5] = { C, C++, Java, Python, Perl }
Array is empty
Array[11] = { 1.1, 2.2, 3.3, 4.4, 5.5, null, 6.6, 7.7, 8.8, null, 9.9 }
</script></code></pre>



在 Java 中，泛型方法不需要指明类型参数 T 的实际类型，编译器可以自己推断，并且我们也没办法显式指明泛型方法中的类型参数 T 的实际类型（C++ 中可以显式指明类型参数的实际类型）；

**extends 限制泛型的类型**
泛型所代表的类型是宽泛的，没有限制的，可以是除了基本类型之外的所有类型；
但是有时候我们不需要这么大的范围，因为有些方法并不是所有引用类型都有的；

这个时候我们就可以使用`extends`关键字来限制泛型的类型范围了，比如`<T extends Number> T max(T a, T b, T c)`；
表示类型参数 T 必须属于 Number 类，也就是说 T 必须是 Number 类及其派生类的类型，其他的类型都是不行的；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Integer[] arr1 = {1, 2, 3, 4, 5};
        print(arr1);
        out.printf("\tmax_val = %s\n", max(arr1).toString());

        String[] arr2 = {"baidu", "aliyun", "google", "qq", "facebook"};
        print(arr2);
        out.printf("\tmax_val = %s\n", max(arr2).toString());
    }

    public static <T extends Comparable<T>> T max(T[] nums) {
        T max_val = nums[0];
        for (int i=1; i<nums.length; i++) {
            if (nums[i].compareTo(max_val) > 0) {
                max_val = nums[i];
            }
        }
        return max_val;
    }

    public static <T> void print(T[] array) {
        if (array.length == 0) {
            out.printf("Array is empty\n");
            return;
        }

        out.printf("Array[%d] = { ", array.length);
        for (T elem : array) {
            if (elem == null) {
                out.printf("null, ");
                continue;
            }
            out.printf("%s, ", elem.toString());
        }
        out.printf("\b\b }");
    }
}
</script></code></pre>



<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:10:00]
$ javac Main.java

# root @ arch in ~/work on git:master x [15:10:17]
$ java Main
Array[5] = { 1, 2, 3, 4, 5 }	max_val = 5
Array[5] = { baidu, aliyun, google, qq, facebook }	max_val = qq
</script></code></pre>



## 泛型类
泛型类的声明和非泛型类的声明类似，除了在类名后面添加了`<T1[, T2, T3, ...]>`泛型声明；
泛型类和泛型方法一样，泛型类的类型参数声明部分也包含一个或多个类型参数，参数间用逗号隔开；

但是有一点和泛型方法不同，泛型类在使用的时候需要显式的给类型参数赋值，指明具体的类型；
**如果不显示的指明类型参数的实际类型，那么就会擦除泛型类型，并向上转型为 Object 类**；

泛型类例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Point<Integer, Integer> p1 = new Point<Integer, Integer>(1, 2);
        p1.print();

        Point<Integer, String> p2 = new Point<Integer, String>(119, "火警电话");
        p2.print();
    }
}

class Point<X, Y> {
    public Point(X x, Y y) {
        m_x = x;
        m_y = y;
    }

    public X getX() { return m_x; }
    public Y getY() { return m_y; }
    public void setX(X x) { m_x = x; }
    public void setY(Y y) { m_y = y; }

    public void print() {
        out.printf("x=%s, y=%s\n", m_x, m_y);
    }

    private X m_x;
    private Y m_y;
}
</script></code></pre>



<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:30:13]
$ javac Main.java

# root @ arch in ~/work on git:master x [15:30:26]
$ java Main
x=1, y=2
x=119, y=火警电话
</script></code></pre>



**类型通配符 ?**
我们将上面的 Point 类修改一下，将 print() 成员函数改为 static 函数：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Point<Integer, Integer> p1 = new Point<Integer, Integer>(1, 2);
        print(p1);

        Point<Integer, String> p2 = new Point<Integer, String>(119, "火警电话");
        print(p2);
    }

    public static void print(Point<?, ?> p) {
        out.printf("x=%s, y=%s\n", p.getX(), p.getY());
    }
}

class Point<X, Y> {
    public Point(X x, Y y) {
        m_x = x;
        m_y = y;
    }

    public X getX() { return m_x; }
    public Y getY() { return m_y; }
    public void setX(X x) { m_x = x; }
    public void setY(Y y) { m_y = y; }

    private X m_x;
    private Y m_y;
}
</script></code></pre>



<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:14:53]
$ javac Main.java

# root @ arch in ~/work on git:master x [16:15:07]
$ java Main
x=1, y=2
x=119, y=火警电话
</script></code></pre>



因为 print() 函数的形参为 Point 类的实例，但是 Point 类是一个泛型类，在使用它的时候必须指明具体的类型 X、Y；
可是我们并不能在编写函数的时候就准确的推断出 X、Y 的实际类型，因为这有非常多种可能性，这时候通配符`?`就派上用场了；

`?`是一个占位符，可以让编译器自动填写类型参数的实际类型；

**限制类型通配符**
类型通配符`?`和类型参数`T`一样，它们的可接收的类型是宽泛的，没有限制的，可以将任何合法的类型传递给它们；

这种宽泛的类型在有些情况下可能会出现问题，就如同前一节中的 compareTo() 方法一样，只有实现了 Comparable 接口的类才有；

1) 使用`? extends Number`，表示匹配到的类型必须为 Number 类及其派生类；
2) 使用`? super Number`，表示匹配到的类型必须为 Number 类及其基类；


## 伪泛型
**语法糖**
`语法糖（Syntactic Sugar）`，也称`糖衣语法`，是由英国计算机学家 Peter.J.Landin 发明的一个术语，指在计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用；

Java 中最常用的语法糖主要有`泛型`、`变长参数`、`条件编译`、`自动拆装箱`、`内部类`等；虚拟机并不支持这些语法，它们在编译阶段就被还原回了简单的基础语法结构，这个过程称为**解语法糖**；

泛型是 JDK1.5 之后引入的一项新特性，Java 语言在还没有出现泛型时，只能通过 Object 是所有类型的父类和类型强制转换这两个特点的配合来实现泛型的功能，这样实现的泛型功能要在程序运行期才能知道 Object 真正的对象类型，在编译期间，编译器无法检查这个 Object 的强制转型是否成功，这便将一些风险转接到了程序运行期中；

Java 语言在 JDK1.5 之后引入的泛型实际上只在程序源码中存在，在编译后的字节码文件中，就已经被替换为了原来的原生类型，并且在相应的地方插入了强制转型代码；
因此对于运行期的 Java 语言来说，`ArrayList<String>`和`ArrayList<Integer>`就是同一个类；所以泛型技术实际上是 Java 语言的一颗语法糖，Java 语言中的泛型实现方法称为`类型擦除`，基于这种方法实现的泛型被称为`伪泛型`；

下面是一段简单的 Java 泛型代码：
<pre><code class="language-java line-numbers"><script type="text/plain">Map<Integer, String> map = new HashMap<Integer, String>();

map.put(1,"No.1");
map.put(2,"No.2");

System.out.println(map.get(1));
System.out.println(map.get(2));
</script></code></pre>



将这段 Java 代码编译成 Class 文件，然后再用字节码反编译工具进行反编译后，将会发现泛型都变回了原生类型，如下面的代码所示：
<pre><code class="language-java line-numbers"><script type="text/plain">Map map = new HashMap();

map.put(1,"No.1");
map.put(2,"No.2");

System.out.println((String)map.get(1));
System.out.println((String)map.get(2));
</script></code></pre>
