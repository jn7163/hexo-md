---
title: Java Collection框架
date: 2017-09-30 12:43:00
categories:
- java
tags:
- java
keywords: Java Collection框架
---

> 
Java Collection框架，List 列表、Queue 队列、Deque 双端队列、Set 集合、Map 映射

<!-- more -->

## 集合框架
Java 集合框架中主要封装的是典型的**数据结构**和**算法**，如动态数组、双向链表、队列、栈、Set、Map 等；
Java 的 Collection 框架和 C++ 的标准模板库（STL）是相似的东西，集合框架极大的方便了 Java 程序的编写；

Collection 框架主要位于 java.util 包，类继承结构：
![Java8 Collection 框架，类结构图](/images/java-collection.png)

主要分为以下几个部分：
1) **数据结构**
`List`列表、`Queue`队列、`Deque`双端队列、`Set`集合、`Map`映射
2) **算法**
`Collections`常用算法类、`Arrays`静态数组的排序、查找算法
3) **迭代器**
`Iterator`通用迭代器、`ListIterator`针对 List 特化的迭代器
4) **比较器**
`Comparator`比较器

**简要分析**
1) List 主要实现类：ArrayList、LinkedList
**ArrayList**：”动态数组”，随机存取性能高，尾端插入删除方便，内部插入删除效率低；
**LinkedList**：”双向链表”，插入删除方便，不支持随机存取，只能从头开始遍历；

2) Queue 主要实现类：PriorityQueue
**PriorityQueue**：“最小堆”，“完全二叉树”，优先级队列，使用数组保存了完全二叉树；

3) Deque 主要实现类：ArrayDeque、LinkedList
**ArrayDeque**：基于数组实现的双端队列，“循环数组”，可作为 Stack 栈使用；
**LinkedList**：基于双向链表实现的双端队列，可以作为 Stack 栈使用；

4) Set 主要实现类：HashSet、LinkedHashSet、TreeSet、RegularEnumSet、JumboEnumSet
**HashSet**：基于 HashMap 实现，元素不可重复，特性同 HashMap；
**LinkedHashSet**：基于 LinkedHashMap 实现，元素不可重复，特性同 LinkedHashMap；
**TreeSet**：基于 TreeMap 实现，元素不可重复，特性同 TreeMap；
**RegularEnumSet**：枚举专用的 Set 集合，由 EnumSet 调用，因为该类的权限为 [default]，元素必须为枚举类型，与其他 Set 实现不同，其内部使用位向量实现，拥有极高的时间和空间性能；
**JumboEnumSet**：枚举专用的 Set 集合，由 EnumSet 调用，因为该类的权限为 [default]，元素必须为枚举类型，与其他 Set 实现不同，其内部使用位向量实现，拥有极高的时间和空间性能；

5) Map 主要实现类：HashMap、LinkedHashMap、TreeMap、IdentityHashMap、WeakHashMap、EnumMap
**HashMap**：key 不可重复，使用 equals 判断，根据 key 的 hashCode 存储数据，具有很快的访问速度，记录的遍历顺序与记录的输入顺序基本不一致；最多只允许一条记录的 key 为 null；
**LinkedHashMap**：key 不可重复，使用 equals 判断，HashMap 的子类，内部使用双向链表保存了记录的插入顺序，使得输入的记录顺序和输出的记录顺序是相同的；
**TreeMap**：key 不可重复，使用 equals 判断，能够把它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器，当用 Iterator 遍历时，得到的记录是排过序的；如需使用排序的映射，建议使用 TreeMap；
**IdentityHashMap**：key 不可重复，使用 == 判断，比较内存地址，可以说是某种意义上的可重复 key 的映射；
**WeakHashMap**：弱引用 Map，特别适合用于需要缓存的场景，WeakHashMap 中的 Entry 可能随时被 GC 回收；
**EnumMap**：枚举专用的 Map 映射，其 key 必须为某种枚举类型的枚举常量，内部通过数组实现，因此效率比一般的 Map 高；


## 主要接口
**Iterable 接口**
Iterable，"可迭代的"，如果一个集合类实现了该接口，那么表示这个集合类可进行迭代操作，比如 JDK1.5 提供的 foreach 循环；

Collection 框架主要有两大接口：Collection、Map，其中 Collection 接口继承了 Iterable 接口，因此所有实现了 Collection 接口的类都是可迭代的，可用于类似 foreach 操作；

Iterable 接口位于 java.lang 包，其主要方法为 iterator()，获取当前集合对象的一个迭代器 Iterator。
<pre><code class="language-java line-numbers"><script type="text/plain">Iterator<T> iterator(); // 获取当前集合的迭代器
default void forEach(Consumer<? super T> action); // forEach，与 Lambda 结合使用
</script></code></pre>



**Iterator 接口**
前面说了 Iterable 可迭代对象，它的主要方法就是 iterator()，获取可迭代对象的迭代器；
而迭代器就是 Iterator，这个接口位于 java.util 包，一般来说每个集合类都有自己的特定迭代器实现，一般是一个内部类；

Iterator 接口主要有两个方法：hasNext()、next()，通常这两个方法需要配合使用；
hasNext() 方法，判断该集合对象是否还有下一个可操作对象，如果有则返回 true，通常用于 for、while 条件；
next() 方法，返回下一个可操作对象，同时将迭代器向后推进一个元素单位，指向下一个位置。
<pre><code class="language-java line-numbers"><script type="text/plain">boolean hasNext(); // 判断是否还有元素
E next(); // 获取下一个元素

default void remove(); // 默认不支持该操作，抛出 UnsupportedOperationException 异常
default void forEachRemaining(Consumer<? super E> action); // forEach 剩余的元素
</script></code></pre>



**Collection 接口**
java.util.Collection 接口是 Java Collection 框架的两大主要接口之一，有三个主要子接口 List、Queue、Set；
因为 Collection 接口是 java.lang.Iterable 接口的子接口，因此 List、Queue、Deque、Set 都可直接用于 foreach 循环；
<pre><code class="language-java line-numbers"><script type="text/plain">int size(); // 获取元素的数量
boolean isEmpty(); // 判空
void clear(); // 清空集合

Iterator<E> iterator(); // 获取迭代器

Object[] toArray(); // 转换为数组，Object[]
<T> T[] toArray(T[] a); // 转换为指定类型的数组

boolean add(E e); // 添加元素
boolean remove(Object o); // 删除元素
boolean addAll(Collection<? extends E> c); // 添加指定集合的所有元素
boolean removeAll(Collection<?> c); // 删除指定集合中的所有元素

boolean contains(Object o); // 测试是否包含指定元素
boolean containsAll(Collection<?> c); // 测试是否包含指定集合的所有元素

default boolean removeIf(Predicate<? super E> filter); // 按条件删除元素
boolean retainAll(Collection<?> c); // 只保留指定集合中包含的元素

boolean equals(Object o); // 判等
int hashCode(); // hashCode 值

default Stream<E> stream(); // 返回以此集合为源的顺序流
default Stream<E> parallelStream(); // 返回一个可能的并行流
</script></code></pre>



**Comparator 接口**
java.lang.Comparable，"可比较的"
java.util.Comparator，"比较器"

它们的区别就如同 Iterable、Iterator 之间的区别：
如果一个类实现了 Comparable 接口，表示这个类的对象之间支持比较操作，可直接用于自然排序（一般为升序）；
如果一个类没有实现 Comparable 接口，但是它有专门的 Comparator 比较器，那么也可用于排序，并且自由度更高，可以实现升序、倒序排序；

Comparable 接口：
<pre><code class="language-java line-numbers"><script type="text/plain">public interface Comparable<T> {
    /**
     * @return 小于0: this < o
     *         等于0: this = o
     *         大于0: this > o
     */
    public int compareTo(T o);
}
</script></code></pre>



Comparator 接口（函数式接口）：
<pre><code class="language-java line-numbers"><script type="text/plain">@FunctionalInterface
public interface Comparator<T> {
    /**
     * @param o1 第一个对象
     * @param o2 第二个对象
     * @return 小于0: o1 < o2
     *         等于0: o1 = o2
     *         大于0: o1 > o2
     * @throws NullPointerException 如果参数为 null, 并且此比较器不允许 null 参数
     * @throws ClassCastException 参数的类型不能应用于元素之间的比较
     */
    int compare(T o1, T o2);

    boolean equals(Object obj);

    // 获取自然排序比较器
    public static <T extends Comparable<? super T>> Comparator<T> naturalOrder();

    // 获取倒序比较器
    public static <T extends Comparable<? super T>> Comparator<T> reverseOrder() {
        return Collections.reverseOrder();
    }

    // 获取倒序比较器(强制降序)
    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }
}
</script></code></pre>



**Cloneable 接口**
java.lang.Cloneable 接口和 java.io.Serializable 接口都是"标记接口"，没有任何方法签名。
稍微研究一下 Collection 框架的源码就可以发现，它们几乎都实现了 Cloneable、Serializable 接口。

Cloneable 接口：如果一个类实现了该接口，表示这个类的对象支持 clone() 操作，即对象是可克隆的；
Serializable 接口：如果一个类实现了该接口，表示这个类支持 readObject()/writeObject() 序列化操作。

在 Object 类中，有一个 protected 的 clone() 方法，并且是一个 native 方法，声明如下：
`protected native Object clone() throws CloneNotSupportedException;`
默认情况下，我们不能直接调用一个对象的 clone() 方法，因为它被 protected 所修饰；

那么我们先把它给重写，并且把其访问性改为 public 试试：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        A a1 = new A();
        A a2 = a1.clone();
    }
}

class A {
    @Override
    public A clone() throws CloneNotSupportedException {
        return (A) super.clone();
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [21:06:21]
$ javac Main.java

# root @ arch in ~/work on git:master x [21:06:35]
$ java Main
Exception in thread "main" java.lang.CloneNotSupportedException: A
	at java.lang.Object.clone(Native Method)
	at A.clone(Main.java:11)
	at Main.main(Main.java:4)
</script></code></pre>



编译没有问题，但是运行时抛出异常 CloneNotSupportedException，不支持 clone 操作。
这是因为我们没有实现 Cloneable 接口，就和 Serializable 序列化一样，会进行类型检查；
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        A a1 = new A();
        A a2 = a1.clone();

        System.out.println(a1);
        System.out.println(a2);
    }
}

class A implements Cloneable {
    @Override
    public A clone() throws CloneNotSupportedException {
        return (A) super.clone();
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [21:09:10]
$ javac Main.java

# root @ arch in ~/work on git:master x [21:09:21]
$ java Main
A@15db9742
A@6d06d69c
</script></code></pre>



可以了哈，先提前说一下，这种直接调用 Object.clone() 方法的克隆方式称为"浅拷贝"；我们来看一下它的局限性：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Arrays;

public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        A a1 = new A();
        System.out.printf("a1 -> i: %d, f: %.2f, obj: %s, arr: %s %s\n", a1.i, a1.f, a1.obj, a1.arr, Arrays.toString(a1.arr));

        A a2 = a1.clone();
        System.out.printf("a2 -> i: %d, f: %.2f, obj: %s, arr: %s %s\n", a2.i, a2.f, a2.obj, a2.arr, Arrays.toString(a2.arr));

        a1.i = 20;
        a2.i = 40;
        System.out.printf("a1 -> i: %d, f: %.2f, obj: %s, arr: %s %s\n", a1.i, a1.f, a1.obj, a1.arr, Arrays.toString(a1.arr));
        System.out.printf("a2 -> i: %d, f: %.2f, obj: %s, arr: %s %s\n", a2.i, a2.f, a2.obj, a2.arr, Arrays.toString(a2.arr));

        a1.arr[0] = 8;
        System.out.printf("a1 -> i: %d, f: %.2f, obj: %s, arr: %s %s\n", a1.i, a1.f, a1.obj, a1.arr, Arrays.toString(a1.arr));
        System.out.printf("a2 -> i: %d, f: %.2f, obj: %s, arr: %s %s\n", a2.i, a2.f, a2.obj, a2.arr, Arrays.toString(a2.arr));

        a2.arr[0] = 9;
        System.out.printf("a1 -> i: %d, f: %.2f, obj: %s, arr: %s %s\n", a1.i, a1.f, a1.obj, a1.arr, Arrays.toString(a1.arr));
        System.out.printf("a2 -> i: %d, f: %.2f, obj: %s, arr: %s %s\n", a2.i, a2.f, a2.obj, a2.arr, Arrays.toString(a2.arr));
    }
}

class A implements Cloneable {
    public int i = 10;
    public float f = 3.14f;
    public Object obj = new Object();
    public int[] arr = {1, 2, 3, 4, 5};

    @Override
    public A clone() throws CloneNotSupportedException {
        return (A) super.clone();
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [21:27:26]
$ javac Main.java

# root @ arch in ~/work on git:master x [21:27:37]
$ java Main
a1 -> i: 10, f: 3.14, obj: java.lang.Object@15db9742, arr: [I@6d06d69c [1, 2, 3, 4, 5]
a2 -> i: 10, f: 3.14, obj: java.lang.Object@15db9742, arr: [I@6d06d69c [1, 2, 3, 4, 5]
a1 -> i: 20, f: 3.14, obj: java.lang.Object@15db9742, arr: [I@6d06d69c [1, 2, 3, 4, 5]
a2 -> i: 40, f: 3.14, obj: java.lang.Object@15db9742, arr: [I@6d06d69c [1, 2, 3, 4, 5]
a1 -> i: 20, f: 3.14, obj: java.lang.Object@15db9742, arr: [I@6d06d69c [8, 2, 3, 4, 5]
a2 -> i: 40, f: 3.14, obj: java.lang.Object@15db9742, arr: [I@6d06d69c [8, 2, 3, 4, 5]
a1 -> i: 20, f: 3.14, obj: java.lang.Object@15db9742, arr: [I@6d06d69c [9, 2, 3, 4, 5]
a2 -> i: 40, f: 3.14, obj: java.lang.Object@15db9742, arr: [I@6d06d69c [9, 2, 3, 4, 5]
</script></code></pre>



对于值类型 int、float，没有所谓的深拷贝浅拷贝之分，因为它就是一个简简单单的数值；
对于引用类型（数组也是引用类型），默认的浅拷贝只会拷贝它的引用，也就是内存地址；

这样的话两个对象之间修改就会互相影响，很显然不是我们想要的理想结果；
我们既然通过 clone 创建新对象，目的就是为了它们之间的修改不会对对方产生影响。

那么如何做到"深拷贝"呢，其实也很简单，只要把每个引用对象都拷贝就行了，如下：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Arrays;

public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        A a1 = new A();
        A a2 = a1.clone();
        System.out.println(a1 + ", " + a1.arr + ", " + Arrays.toString(a1.arr));
        System.out.println(a2 + ", " + a2.arr + ", " + Arrays.toString(a2.arr));

        a1.arr[0] = 8;
        a2.arr[0] = 9;
        System.out.println(a1 + ", " + a1.arr + ", " + Arrays.toString(a1.arr));
        System.out.println(a2 + ", " + a2.arr + ", " + Arrays.toString(a2.arr));
    }
}

class A implements Cloneable {
    public int[] arr = {1, 2, 3, 4, 5};

    @Override
    public A clone() throws CloneNotSupportedException {
        A result = (A) super.clone();
        result.arr = this.arr.clone();
        return result;
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [21:42:01]
$ javac Main.java

# root @ arch in ~/work on git:master x [21:42:11]
$ java Main
A@15db9742, [I@6d06d69c, [1, 2, 3, 4, 5]
A@7852e922, [I@4e25154f, [1, 2, 3, 4, 5]
A@15db9742, [I@6d06d69c, [8, 2, 3, 4, 5]
A@7852e922, [I@4e25154f, [9, 2, 3, 4, 5]
</script></code></pre>



目前看来好像达到了我们的"深拷贝"目的，但是，有个问题，如果有的数据成员没有实现 Cloneable 接口怎么办；
如果是我们自己的类还好说，进行简单的修改就好了，但是如果是其他的呢，比如 Java API 的，我们不可能去修改。

其实还有一种办法，可以做到完美的"深拷贝"，那就是 java.io.Serializable 接口，对的，通过序列化来进行深拷贝。
如果你还不了解**对象序列化**，请猛戳[Serializable - 对象序列化与反序列化](https://www.zfl9.com/java-io.html#Object-引用类型)

特别的，如果序列化对象的成员变量为引用类型，并且它不属于 String、Array、Enum，Serializable 类型，那么在执行序列化的过程中，将会抛出运行时异常 NotSerializableException；

因此对于上述情况的引用类型成员变量，有三种选择：
1) 使用 transient 关键字声明该成员，在序列化时跳过该成员变量，在反序列化之后自动赋值 null；
2) 如果该引用类型是你自己创建的类，那么可以直接让其实现 java.io.Serializable 接口；
3) 创建一个派生类，这个派生类没有任何声明的方法和属性，仅仅用于序列化和反序列化。

其中第一、二种方式并不是我们想要的，我们来实现第三种方式：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
import java.io.IOException;

public class Main {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        A.Serial a1 = new A.Serial();
        A a2 = serial(a1); // 发生向上转型

        System.out.println(a1 + ", " + a1.obj);
        System.out.println(a2 + ", " + a2.obj);
    }

    @SuppressWarnings("unchecked")
    private static <T extends Serializable> T serial(T t) throws IOException, ClassNotFoundException {
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        ObjectOutputStream objOut = new ObjectOutputStream(out);
        objOut.writeObject(t);
        ObjectInputStream objIn = new ObjectInputStream(new ByteArrayInputStream(out.toByteArray()));
        return (T) objIn.readObject();
    }
}

class A { // 注意 A 不能实现 Serializable 接口！
    public Object obj = new Object();

    // 专用于序列化的静态内部类 A.Serial
    public static class Serial extends A implements Serializable {
        private static final long serialVersionUID = 1L;
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:02:48]
$ javac Main.java

# root @ arch in ~/work on git:master x [9:03:00]
$ java Main
A$Serial@55f96302, java.lang.Object@119d7047
A$Serial@776ec8df, java.lang.Object@4eec7777
</script></code></pre>



通过这种方式虽然完美的解决的 Object.clone()"深拷贝"的问题，但是效率相对来说比较低下。


## List 列表
**ListIterator 接口**
ListIterator 是 Iterator 的一个子接口，是专门用于 List 列表的迭代器。
<pre><code class="language-java line-numbers"><script type="text/plain">public interface ListIterator<E> extends Iterator<E> {
    boolean hasNext(); // [正向] 判断是否还有元素
    E next(); // [正向] 获取下一个元素

    boolean hasPrevious(); // [逆向] 判断是否还有元素
    E previous(); // [逆向] 获取下一个元素

    int nextIndex(); // [正向] 获取下一个元素的索引, 如果当前迭代器位于列表末尾则为列表大小
    int previousIndex(); // [逆向] 获取下一个元素的索引, 如果当前迭代器位于列表开头则返回 -1

    void remove(); // [正向/逆向] 移除下一个元素

    void set(E e); // [正向/逆向] 替换下一个元素

    void add(E e); // [正向/逆向] 在下一元素之前插入指定元素
}
</script></code></pre>



逆向迭代的例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Iterator;
import java.util.ListIterator;
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>(10);

        // 填充元素
        for (int i = 0; i < 10; i++) {
            list.add(i);
        }

        // Iterable.forEach() 使用 Lambda
        list.forEach((e) -> System.out.print(e + ", "));
        System.out.println();

        // Iterable.iterator() 获取迭代器
        for (Iterator<Integer> iter = list.iterator(); iter.hasNext();) {
            System.out.print(iter.next() + ", ");
        }
        System.out.println();

        // JDK1.5 foreach循环 [语法糖]
        for (int e : list) {
            System.out.print(e + ", ");
        }
        System.out.println();

        // ListIterator 逆向迭代
        for (ListIterator<Integer> iter = list.listIterator(list.size()); iter.hasPrevious();) {
            System.out.print(iter.previous() + ", ");
        }
        System.out.println();
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [11:15:23]
$ javac Main.java

# root @ arch in ~/work on git:master x [11:18:40]
$ java Main
0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
9, 8, 7, 6, 5, 4, 3, 2, 1, 0,
</script></code></pre>



**List 接口**
<pre><code class="language-java line-numbers"><script type="text/plain">int size(); // 获取元素的数量
boolean isEmpty(); // 判空
void clear(); // 清空列表

boolean contains(Object o); // 测试是否包含指定元素
boolean containsAll(Collection<?> c); // 测试是否包含指定的所有元素

int indexOf(Object o); // 查找指定元素第一次出现的索引值
int lastIndexOf(Object o); // 最后一次出现的索引值

Iterator<E> iterator(); // 获取迭代器
ListIterator<E> listIterator(); // 获取 List 专用迭代器
ListIterator<E> listIterator(int index); // 指定起始索引

Object[] toArray(); // 转换为数组，注意是 Object[]
<T> T[] toArray(T[] a); // 转换为指定类型数组，泛型方法

E get(int index); // 获取指定索引的元素
E set(int index, E element); // 设置指定索引的元素

boolean add(E e); // 追加元素
boolean remove(Object o); // 移除元素(首次出现的)

void add(int index, E element); // 在指定索引上插入元素
E remove(int index); // 删除指定索引上的元素

boolean addAll(Collection<? extends E> c); // 添加所有元素
boolean addAll(int index, Collection<? extends E> c); // 插入指定元素到指定位置
boolean removeAll(Collection<?> c); // 删除指定的所有元素
boolean retainAll(Collection<?> c); // 只保留指定集合中包含的元素

List<E> subList(int fromIndex, int toIndex); // 提取子列表，[fromIndex, toIndex)

default void replaceAll(UnaryOperator<E> operator); // 替换所有元素为 operator 的结果
default void sort(Comparator<? super E> c); // 按照指定比较器进行排序

boolean equals(Object o); // 判等
int hashCode(); // hashCode 值
</script></code></pre>



**ArrayList 类**
<pre><code class="language-java line-numbers"><script type="text/plain">public ArrayList(int initialCapacity); // 指定初始化容量
public ArrayList(); // 默认为一个空的 Object[] 数组
public ArrayList(Collection<? extends E> c); // 根据指定集合构造一个列表

public void trimToSize(); // 回收多余的容量
public void ensureCapacity(int minCapacity); // 扩容(仅当 minCapacity 大于当前容量时有效)

public int size(); // 获取元素的个数
public boolean isEmpty(); // 判空
public void clear(); // 全部元素置为 null

public boolean contains(Object o); // 测试是否包含指定元素
public int indexOf(Object o); // 获取指定元素第一次出现的索引值
public int lastIndexOf(Object o); // 最后一次出现的索引值

public Object clone(); // 拷贝动态数组

public Object[] toArray(); // 转换为静态数组
public <T> T[] toArray(T[] a); // 指定数组类型

public E get(int index); // 获取指定索引值的元素
public E set(int index, E element); // 设置指定索引值的元素

public boolean add(E e); // 追加指定元素
public void add(int index, E element); // 在指定位置插入元素
public E remove(int index); // 移除指定位置的元素
public boolean remove(Object o); // 移除第一个相等的指定元素

public boolean addAll(Collection<? extends E> c); // 追加指定集合的元素
public boolean addAll(int index, Collection<? extends E> c); // 插入到指定位置
public boolean removeAll(Collection<?> c); // 移除所有与指定集合中相同的元素
public boolean retainAll(Collection<?> c); // 只保留与指定集合中相同的元素

public Iterator<E> iterator(); // 获取迭代器
public ListIterator<E> listIterator(); // 获取 List 迭代器
public ListIterator<E> listIterator(int index); // 获取 List 迭代器，指定起始索引

public List<E> subList(int fromIndex, int toIndex); // 提取子串

public void sort(Comparator<? super E> c); // 自定义排序

public void forEach(Consumer<? super E> action); // forEach 循环
public boolean removeIf(Predicate<? super E> filter); // 按条件删除
public void replaceAll(UnaryOperator<E> operator); // 全部替换
</script></code></pre>



**LinkedList 类**
<pre><code class="language-java line-numbers"><script type="text/plain">public LinkedList(); // 默认构造，空函数体
public LinkedList(Collection<? extends E> c); // 使用指定集合进行填充

public int size(); // 获取元素个数
public void clear(); // 清空列表
public Object clone(); // 拷贝

public Object[] toArray(); // 转换为 Object[] 数组
public <T> T[] toArray(T[] a); // 指定数组元素类型

public ListIterator<E> listIterator(int index); // 获取 List 迭代器
public Iterator<E> descendingIterator(); // [Deque] 获取逆向迭代器

public boolean add(E e); // 在链表尾部追加元素
public boolean remove(Object o); // 删除与指定元素相同的第一个元素
public boolean addAll(Collection<? extends E> c); // 在链表尾部追加指定集合的元素
public boolean addAll(int index, Collection<? extends E> c); // 插入至指定索引处

public E get(int index); // 获取指定索引的元素
public E set(int index, E element); // 更新指定索引上的元素
public void add(int index, E element); // 在指定索引处插入指定元素
public E remove(int index); // 删除指定索引上的元素

public boolean contains(Object o); // 测试是否包含指定元素
public int indexOf(Object o); // 查找指定元素第一次出现的索引值
public int lastIndexOf(Object o); // 最后一次匹配时的索引值

public boolean removeFirstOccurrence(Object o); // 删除链表中与指定元素首次匹配的元素
public boolean removeLastOccurrence(Object o); // 最后一次出现

// Deque 双端队列
/* 抛异常 */
public void addFirst(E e); // 在头部添加元素
public void addLast(E e); // 在尾部添加元素
public E removeFirst(); // 在头部弹出元素
public E removeLast(); // 在尾部弹出元素
public E getFirst(); // 在头部窥探元素
public E getLast(); // 在尾部窥探元素
/* 返回值 */
public boolean offerFirst(E e); // 在头部添加元素
public boolean offerLast(E e); // 在尾部添加元素
public E pollFirst(); // 在头部弹出元素
public E pollLast(); // 在尾部弹出元素
public E peekFirst(); // 在头部窥探元素
public E peekLast(); // 在尾部窥探元素

// Queue 队列
/* 抛异常 */
public boolean add(E e); // 等同于 addLast()
public E remove(); // 等同于 removeFirst()
public E element(); // 等同于 getFirst()
/* 返回值 */
public boolean offer(E e); // 等同于 offerLast()
public E poll(); // 等同于 pollFirst()
public E peek(); // 等同于 peekFirst()

// Stack 栈
/* 抛异常 */
public void push(E e); // 等同于 addFirst()
public E pop(); // 等同于 removeFirst()
</script></code></pre>



## Queue 队列
**Queue 接口**
Queue 队列，遵循 FIFO 先进先出原则，在队尾插入元素叫做入队，在队头弹出元素叫做出队。
<pre><code class="language-java line-numbers"><script type="text/plain">// 失败时抛异常IllegalStateException、NoSuchElementException
boolean add(E e); // 在队尾插入元素
E remove(); // 在队头弹出元素
E element(); // 在队头窥探元素

// 失败时返回false、null值
boolean offer(E e); // 在队尾插入元素
E poll(); // 在队头弹出元素
E peek(); // 在队头窥探元素
</script></code></pre>



**Deque 接口**
Deque 双端队列，是具有**队列**、**栈**性质的一种数据结构，在 Java 中一般作为 Stack 栈使用。
java.util.Deque 是 java.util.Queue 的子接口，主要有两个实现类：ArrayDeque、LinkedList；
<pre><code class="language-java line-numbers"><script type="text/plain">public int size(); // 获取元素个数

Iterator<E> iterator(); // 获取迭代器
Iterator<E> descendingIterator(); // 获取逆向迭代器

// 失败时抛异常IllegalStateException、NoSuchElementException
void addFirst(E e); // 在头部添加元素
void addLast(E e); // 在尾部添加元素
E removeFirst(); // 在头部弹出元素
E removeLast(); // 在尾部弹出元素
E getFirst(); // 在头部窥探元素
E getLast(); // 在尾部窥探元素

// 失败时返回false、null值
boolean offerFirst(E e); // 在头部添加元素
boolean offerLast(E e); // 在尾部添加元素
E pollFirst(); // 在头部弹出元素
E pollLast(); // 在尾部弹出元素
E peekFirst(); // 在头部窥探元素
E peekLast(); // 在尾部窥探元素

// 将 Deque 当做 Queue 使用
boolean add(E e); // 等同于 addLast()
E remove(); // 等同于 removeFirst()
E element(); // 等同于 getFirst()

boolean offer(E e); // 等同于 offerLast()
E poll(); // 等同于 pollFirst()
E peek(); // 等同于 peekFirst()

// 将 Deque 当做 Stack 使用
void push(E e); // 等同于 addFirst()
E pop(); // 等同于 removeFirst()

boolean remove(Object o); // 等同于 removeFirstOccurrence()
boolean removeFirstOccurrence(Object o); // 移除首次匹配指定元素的队列元素
boolean removeLastOccurrence(Object o); // 移除最后一次匹配指定元素的队列元素

boolean contains(Object o); // 测试是否包含指定元素
</script></code></pre>



**PriorityQueue 类**
java.util.PriorityQueue 优先队列，通过二叉小顶堆实现，可以用一棵完全二叉树表示；

PriorityQueue 优先队列的作用是能保证每次取出的元素都是队列中**权值最小**的（Java 的优先队列每次取最小元素，C++ 的优先队列每次取最大元素）。

这里牵涉到了大小关系，元素大小的评判可以通过元素本身的**自然顺序**（natural ordering），也可以通过构造时传入的**比较器**（Comparator，类似于 C++ 的仿函数）。

PriorityQueue 实现了 Queue 接口，**不允许放入 null 元素**；其通过堆实现，具体说是通过**完全二叉树（complete binary tree）**实现的小顶堆（任意一个非叶子节点的权值，都不大于其左右子节点的权值），也就意味着可以通过数组来作为 PriorityQueue 的底层实现。
![PriorityQueue 通过用数组表示的小顶堆实现](/images/java-priorityqueue.png)

上图中我们给每个元素按照层序遍历的方式进行了编号，如果你足够细心，会发现父节点和子节点的编号是有联系的，更确切的说父子节点的编号之间有如下关系：
> 
`leftNo = parentNo * 2 + 1`
`rightNo = parentNo * 2 + 2`
`parentNo = (nodeNo - 1) / 2`

通过上述三个公式，可以轻易计算出某个节点的父节点以及子节点的下标。这也就是为什么可以直接用数组来存储堆的原因。

PriorityQueue 的 peek() 和 element() 操作是常数时间，add()、offer()、无参数的 remove()、poll() 方法的时间复杂度都是 log(N)。

**主要方法**
<pre><code class="language-java line-numbers"><script type="text/plain">/* 构造函数 */
public PriorityQueue(); // 初始容量 11
public PriorityQueue(int initialCapacity); // 指定初始容量
public PriorityQueue(Comparator<? super E> comparator); // 指定比较器
public PriorityQueue(int initialCapacity, Comparator<? super E> comparator); // 指定初始容量、比较器
public PriorityQueue(Collection<? extends E> c); // 通过其他集合进行填充
public PriorityQueue(PriorityQueue<? extends E> c); // 拷贝构造
public PriorityQueue(SortedSet<? extends E> c); // 通过 SortedSet 填充

public int size(); // 获取元素个数
public Comparator<? super E> comparator(); // 获取当前实例的比较器
public void clear(); // 清空队列

public boolean add(E e); // 内部调用 offer()，添加元素
public boolean remove(Object o); // 删除与指定元素相匹配的单个实例

public boolean contains(Object o); // 测试是否包含指定元素

public Object[] toArray(); // 转换为 Object[] 数组
public <T> T[] toArray(T[] a); // 指定数组元素类型

public Iterator<E> iterator(); // 获取迭代器, 不保证按照优先级遍历

public boolean offer(E e); // 添加元素, 失败返回 false
public E poll(); // 弹出队头元素, 队列为空返回 null
public E peek(); // 窥探队头元素, 队列为空返回 null
</script></code></pre>



例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Random;
import java.util.PriorityQueue;

public class Main {
    public static void main(String[] args) {
        PriorityQueue<Integer> queue1 = new PriorityQueue<>(10);

        Random rand = new Random();
        for (int i = 0; i < 10; i++) {
            queue1.offer(rand.nextInt(90) + 10); // 随机两位数
        }

        for (int i = 0; i < 10; i++) { // 默认是自然排序 [升序]
            System.out.print(queue1.poll() + ", ");
        }
        System.out.println();

        /*
         * (int n1, int n2) -> { return n1 - n2; } 升序
         * (int n1, int n2) -> { return n2 - n1; } 降序
         */
        PriorityQueue<Integer> queue2 = new PriorityQueue<>(10, (n1, n2) -> n2 - n1);

        for (int i = 0; i < 10; i++) {
            queue2.offer(rand.nextInt(90) + 10); // 随机两位数
        }

        for (int i = 0; i < 10; i++) {
            System.out.print(queue2.poll() + ", ");
        }
        System.out.println();
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:13:51]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:14:01]
$ java Main
10, 10, 15, 24, 36, 39, 69, 70, 85, 90,
96, 91, 87, 73, 71, 61, 60, 49, 24, 11,

# root @ arch in ~/work on git:master x [14:14:03]
$ java Main
12, 36, 45, 50, 64, 73, 80, 88, 90, 91,
87, 82, 68, 53, 49, 38, 36, 36, 32, 30,

# root @ arch in ~/work on git:master x [14:14:04]
$ java Main
15, 28, 42, 69, 82, 83, 90, 92, 96, 99,
93, 87, 64, 59, 59, 56, 51, 31, 23, 17,
</script></code></pre>



**ArrayDeque 类**
ArrayDeque 内部使用"循环数组"实现，LinkedList 内部使用"双向链表"实现；
ArrayDeque 不允许 null 元素，LinkedList 允许 null 元素；
<pre><code class="language-java line-numbers"><script type="text/plain">public ArrayDeque(); // 数组初始大小为 16
public ArrayDeque(int numElements); // 指定数组大小
public ArrayDeque(Collection<? extends E> c); // 使用指定集合进行填充

// Deque 双端队列
/* 抛异常 NullPointerException、NoSuchElementException */
public void addFirst(E e); // 在头部添加元素
public void addLast(E e); // 在尾部添加元素
public E removeFirst(); // 在头部弹出元素
public E removeLast(); // 在尾部弹出元素
public E getFirst(); // 在头部窥探元素
public E getLast(); // 在尾部窥探元素
/* 返回值 false、null */
public boolean offerFirst(E e); // 在头部添加元素
public boolean offerLast(E e); // 在尾部添加元素
public E pollFirst(); // 在头部弹出元素
public E pollLast(); // 在尾部弹出元素
public E peekFirst(); // 在头部窥探元素
public E peekLast(); // 在尾部窥探元素

// Queue 队列
/* 抛异常 */
public boolean add(E e); // 等同于 addLast()
public E remove(); // 等同于 removeFirst()
public E element(); // 等同于 getFirst()
/* 返回值 */
public boolean offer(E e); // 等同于 offerLast()
public E poll(); // 等同于 pollFirst()
public E peek(); // 等同于 peekFirst()

// Stack 栈
/* 抛异常 */
public void push(E e); // 等同于 addFirst()
public E pop(); // 等同于 removeFirst()

public boolean removeFirstOccurrence(Object o); // 删除首次匹配的队列元素
public boolean removeLastOccurrence(Object o); // 删除最后一次匹配的队列元素

public int size(); // 获取元素个数
public boolean isEmpty(); // 判空
public void clear(); // 清空队列
public boolean remove(Object o); // 移除首次匹配的元素

public Iterator<E> iterator(); // 获取迭代器
public Iterator<E> descendingIterator(); // 获取逆向迭代器

public boolean contains(Object o); // 测试是否包含指定元素

public Object[] toArray(); // 转换为 Object[] 数组
public <T> T[] toArray(T[] a); // 转换为指定类型数组

public ArrayDeque<E> clone(); // 克隆当前队列
</script></code></pre>



## Map 映射
**Map 接口**
<pre><code class="language-java line-numbers"><script type="text/plain">public interface Map<K, V> {
    interface Entry<K, V> { // Map.Entry 子接口, public static interface Map.Entry<K, V>
        K getKey(); // 获取 key
        V getValue(); // 获取 value
        V setValue(V value); // 更新 value

        boolean equals(Object o); // entry 判等
        int hashCode(); // hashCode 值

        // [自然排序] 获取 key 的比较器
        public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K, V>> comparingByKey();
        // [自然排序] 获取 value 的比较器
        public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K, V>> comparingByValue();
        // [自定义排序] 获取 key 的比较器
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp);
        // [自定义排序] 获取 value 的比较器
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp);
    }

    int size(); // 获取元素个数
    boolean isEmpty(); // 判空
    void clear(); // 清空 map

    boolean containsKey(Object key); // 测试是否包含指定key
    boolean containsValue(Object value); // 测试是否包含指定value

    V get(Object key); // 获取指定 key 的 value；不存在则返回 null
    V put(K key, V value); // 添加 key-value 键值对，如果存在则更新 value，返回旧值
    V remove(Object key); // 删除指定 key-value 键值对，返回旧值
    void putAll(Map<? extends K, ? extends V> m); // 添加指定 map 中的所有 key-value 对

    Set<K> keySet(); // 返回 key 集合，该 set 集合与当前 map 映射之间会互相影响
    Collection<V> values(); // 返回 value 集合，该 value 集合与当前 map 映射之间会互相影响
    Set<Map.Entry<K, V>> entrySet(); // 返回 key-value 集合，该 set 集合与当前 map 映射之间会互相影响

    boolean equals(Object o); // 比较两个 map
    int hashCode(); // 获取 hashCode 值

    default void forEach(BiConsumer<? super K, ? super V> action); // Lambda foreach循环

    default V getOrDefault(Object key, V defaultValue); // 如果指定 key 不存在则返回默认 value
    default V putIfAbsent(K key, V value); // 如果 key 不存在则 put(key, value), 否则返回原有 value
    default boolean remove(Object key, Object value); // 当 map 存在指定 key-value 映射时，执行 remove() 操作

    default boolean replace(K key, V oldValue, V newValue); // 当 map 存在指定 key-value 映射时，执行 replace() 操作
    default V replace(K key, V value); // 替换指定 key 的 value
    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function); // 替换value
}
</script></code></pre>



**HashMap 类**
HashMap 允许存在 null 键、null 值，不保证遍历的顺序与插入的顺序是一样的，也不保证遍历的顺序可以永恒不变(自动扩容的原因)。

HashMap 是**数组**和**链表**的结合体，HashMap 底层是一个`Entry[]`数组，该数组可以自动扩容；
Entry[] 数组的每个元素都是一个链表，Entry 有一个 next 指针指向下一个 Node 节点。如下图：
![HashMap 数据结构](/images/java-collection-hashmap.png)

因为 HashMap 使用 hash() 哈希函数计算 key-value 键值对的索引值，因此它同时具有数组和链表的优点：查找，插入/删除都具有很高的性能。

<pre><code class="language-java line-numbers"><script type="text/plain">/**
 * 主要构造函数，有两个影响 HashMap 性能的参数.
 * @param initialCapacity 初始容量，即 Entry[] 数组的初始长度，默认为 16
 * @param loadFactor 负载因子，默认为 0.75f
 *                   如果当前元素数量(即 Entry 键值对的数量)超过 Entry[].length * loadFactor 时，
 *                   HashMap 内部将对 Entry[] 数组进行扩容，新容量为当前容量的 2 倍。
 *                   在扩容时需要重新计算 hash，并且需要拷贝元素，开销非常大。
 *                   因此，如果可以预见元素的数量，务必指明 initialCapacity 的大小
 */
public HashMap(int initialCapacity, float loadFactor);
public HashMap(int initialCapacity);
public HashMap();
public HashMap(Map<? extends K, ? extends V> m);

public int size(); // 元素数量
public boolean isEmpty(); // 判空
public void clear(); // 清空元素

public V get(Object key); // 按 key 查找 value，不存在则返回 null
public V getOrDefault(Object key, V defaultValue); // 获取指定 key 的 value，如果没有则返回默认值

public V put(K key, V value); // 设置指定 key-value 对，返回旧值
public V putIfAbsent(K key, V value); // 如果 key 不存在则 put()，否则返回原有 value
public void putAll(Map<? extends K, ? extends V> m); // 将指定 map 全部 put()

public V remove(Object key); // 删除指定 key 映射，并返回相应的 value
public boolean remove(Object key, Object value); // 当存在指定 key-value 时才进行 remove()

public V replace(K key, V value); // 直接替换
public boolean replace(K key, V oldValue, V newValue); // 当存在指定 key-value 时才进行 replace()
public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function); // 全部执行 replace()

public boolean containsKey(Object key); // 判断是否存在指定 key 映射
public boolean containsValue(Object value); // 判断是否存在指定 value 值

public Set<K> keySet(); // 获取 key 的集合，它们之间有关联
public Collection<V> values(); // 获取 value 的集合，他们之间有关联
public Set<Map.Entry<K,V>> entrySet(); // 获取 key-value 对的集合，通常用于遍历

public void forEach(BiConsumer<? super K, ? super V> action); // JDK1.8 forEach，内部使用 entrySet()

public Object clone(); // 拷贝当前 map
</script></code></pre>



**LinkedHashMap 类**
LinkedHashMap 是 HashMap 的子类，底层还是使用的 HashMap，没有变化；
不同的是，LinkedHashMap 可以按照**插入顺序**进行遍历，也可以根据**访问顺序**进行遍历。

可以保存顺序的原理就是它除了使用哈希表保存键值对之外，还在内部维护了一个运行时的双向链表；
当按照**访问顺序**进行排序时，LinkedHashMap 会将最近访问的键值对移动到链表的尾部。
<pre><code class="language-java line-numbers"><script type="text/plain">// accessOrder 为 true 则按照访问顺序排列，为 false 则按照插入顺序排列
public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder);
public LinkedHashMap(int initialCapacity, float loadFactor); // 插入顺序
public LinkedHashMap(int initialCapacity);
public LinkedHashMap();
public LinkedHashMap(Map<? extends K, ? extends V> m);

public void clear();

public boolean containsValue(Object value); // 判断是否存在指定 value

public V get(Object key); // 获取指定 key 的 value
public V getOrDefault(Object key, V defaultValue);
public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function);

/* 有序的 */
public Set<K> keySet();
public Collection<V> values();
public Set<Map.Entry<K,V>> entrySet();

public void forEach(BiConsumer<? super K, ? super V> action); // JDK1.8 forEach
</script></code></pre>



LinkedHashMap 例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Map;
import java.util.LinkedHashMap;

public class Main {
    public static void main(String[] args) {
        LinkedHashMap<String, String> map = new LinkedHashMap<>(16, 0.75f, true);
        for (int i = 0; i < 10; i++) {
            map.put("K" + i, "V" + i);
        }

        for (Map.Entry<?, ?> entry : map.entrySet()) {
            System.out.println("key: " + entry.getKey() + ", value: " + entry.getValue());
        }

        map.get("K1");
        map.get("K2");
        map.get("K3");

        System.out.println();
        for (Map.Entry<?, ?> entry : map.entrySet()) {
            System.out.println("key: " + entry.getKey() + ", value: " + entry.getValue());
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:59:45]
$ javac Main.java

# root @ arch in ~/work on git:master x [21:00:00]
$ java Main
key: K0, value: V0
key: K1, value: V1
key: K2, value: V2
key: K3, value: V3
key: K4, value: V4
key: K5, value: V5
key: K6, value: V6
key: K7, value: V7
key: K8, value: V8
key: K9, value: V9

key: K0, value: V0
key: K4, value: V4
key: K5, value: V5
key: K6, value: V6
key: K7, value: V7
key: K8, value: V8
key: K9, value: V9
key: K1, value: V1
key: K2, value: V2
key: K3, value: V3
</script></code></pre>



**TreeMap 类**
TreeMap 是一个**有序**的 key-value 集合，它是通过**红黑树**实现的；
TreeMap 实现了 NavigableMap 接口，意味着它支持一系列的导航方法；比如返回有序的 key 集合。

TreeMap 基于`红黑树（Red-Black tree）`实现。该映射根据其键的**自然顺序**进行排序，或者根据创建映射时提供的 Comparator 进行排序，具体取决于使用的构造方法。

TreeMap 不允许 null 键，但是可以存在 null 值。
TreeMap 的基本操作 containsKey、get、put 和 remove 的时间复杂度是 log(n)。
<pre><code class="language-java line-numbers"><script type="text/plain">public TreeMap(); // 自然顺序，key 必须实现 java.lang.Comparable 接口
public TreeMap(Comparator<? super K> comparator); // 自定义排序，key 不必实现 Comparable 接口，提供比较器即可
public TreeMap(Map<? extends K, ? extends V> m); // 自然排序
public TreeMap(SortedMap<K, ? extends V> m); // 从 m 中获取比较器

public int size(); // 获取元素个数
public Comparator<? super K> comparator(); // 获取当前比较器
public void clear(); // 清空元素
public Object clone(); // 拷贝map

public V get(Object key); // 获取指定 value
public V put(K key, V value); // 添加指定 key-value，并返回旧值
public void putAll(Map<? extends K, ? extends V> map); // 全部 put()
public V remove(Object key); // 删除指定 key-value，并返回相应的 value

public boolean containsKey(Object key); // 是否包含指定 key
public boolean containsValue(Object value); // 是否包含指定 value

public K firstKey(); // 获取第一个 key，(lowest)，如果为空则抛出 NoSuchElementException
public K lastKey(); // 获取最后一个 key，(highest)，如果为空则抛出 NoSuchElementException

public Map.Entry<K,V> firstEntry(); // 获取第一个 Entry，如果为空则返回 null
public Map.Entry<K,V> lastEntry(); // 获取最后一个 Entry，如果为空则返回 null
public Map.Entry<K,V> pollFirstEntry(); // 弹出第一个 Entry，如果为空则返回 null
public Map.Entry<K,V> pollLastEntry(); // 弹出最后一个 Entry，如果为空则返回 null

/*
 * lower "小于"，即当前 key 的前一个 key/Entry (与 key 的 Comparator 有关)
 * floor "小于等于"
 * higher "大于"，即当前 key 的后一个 key/Entry (与 key 的 Comparator 有关)
 * ceiling "大于等于"
 */
public Map.Entry<K,V> lowerEntry(K key); // 获取一个"小于"指定 key 的 Entry
public K lowerKey(K key); // 获取一个"小于"指定 key 的 key
public Map.Entry<K,V> floorEntry(K key); // 获取一个"小于等于"指定 key 的 Entry
public K floorKey(K key); // 获取一个"小于等于"指定 key 的 key

public Map.Entry<K,V> higherEntry(K key); // 获取一个"大于"指定 key 的 Entry
public K higherKey(K key); // 获取一个"大于"指定 key 的 key
public Map.Entry<K,V> ceilingEntry(K key); // 获取一个"大于等于"指定 key 的 Entry
public K ceilingKey(K key); // 获取一个"大于等于"指定 key 的 key

public Set<K> keySet(); // [升序] 获取一个包含所有 key 的 Set 集合，此 set 集合的修改会影响当前 map
public NavigableSet<K> navigableKeySet(); // [升序] 获取一个包含所有 key 的 NavigableSet 集合，此 set 集合的修改会影响当前 map
public NavigableSet<K> descendingKeySet(); // [降序]

public Collection<V> values(); // [升序]，value 的集合，会互相影响

public Set<Map.Entry<K,V>> entrySet(); // [升序] entry 的集合，会互相影响

public NavigableMap<K, V> descendingMap(); // [降序] 互相影响

// 提取子 map
public NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive, K toKey, boolean toInclusive);
// 提取 head 部分
public NavigableMap<K,V> headMap(K toKey, boolean inclusive);
// 提取 tail 部分
public NavigableMap<K,V> tailMap(K fromKey, boolean inclusive);

public SortedMap<K,V> subMap(K fromKey, K toKey); // [fromKey, toKey) 区间
public SortedMap<K,V> headMap(K toKey); // [firstKey, toKey) 区间
public SortedMap<K,V> tailMap(K fromKey); // [fromKey, lastKey] 区间

public V replace(K key, V value); // 直接替换
public boolean replace(K key, V oldValue, V newValue); // 如果存在指定 key-value，则替换 value
public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function); // 全部替换

public void forEach(BiConsumer<? super K, ? super V> action); // forEach 循环
</script></code></pre>



例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.TreeMap;

public class Main {
    public static void main(String[] args) {
        TreeMap<Integer, Integer> map = new TreeMap<>(); // 默认升序
        for (int i = 0; i < 10; i++) {
            map.put(9 - i, 9 - i);
        }
        System.out.println(map);

        TreeMap<Integer, Integer> map2 = new TreeMap<>((n1, n2) -> n2 - n1); // 降序
        for (int i = 0; i < 10; i++) {
            map2.put(i, i);
        }
        System.out.println(map2);

        System.out.println(map.get(map.lowerKey(5))); // 4
        System.out.println(map.get(map.floorKey(5))); // 5
        System.out.println(map.get(map.higherKey(5))); // 6
        System.out.println(map.get(map.ceilingKey(5))); // 5
        System.out.println();
        System.out.println(map2.get(map2.lowerKey(5))); // 6
        System.out.println(map2.get(map2.floorKey(5))); // 5
        System.out.println(map2.get(map2.higherKey(5))); // 4
        System.out.println(map2.get(map2.ceilingKey(5))); // 5
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:40:22] C:127
$ javac Main.java

# root @ arch in ~/work on git:master x [17:40:24]
$ java Main
{0=0, 1=1, 2=2, 3=3, 4=4, 5=5, 6=6, 7=7, 8=8, 9=9}
{9=9, 8=8, 7=7, 6=6, 5=5, 4=4, 3=3, 2=2, 1=1, 0=0}
4
5
6
5

6
5
4
5
</script></code></pre>



**IdentityHashMap 类**
IdentityHashMap 与常用的 HashMap 的区别是：
1) 前者比较 key 时是“引用相等”而后者是“对象相等”，即对于 k1 和 k2，当 k1 == k2 时，IdentityHashMap 认为两个 key 相等，而 HashMap 只有在 k1.equals(k2) == true 时才会认为两个 key 相等；
2) IdentityHashMap 允许使用 null 作为 key 和 value；不保证任何 Key-value 对的之间的顺序，更不能保证他们的顺序随时间的推移不会发生变化；
3) IdentityHashMap 有其特殊用途，比如**序列化**或者**深度复制**、**记录对象代理**等；

举个例子，JVM 中的所有对象都是独一无二的，就算两个对象的数据完全相同，对于 JVM 来说，他们也是完全不同的；
因此，如果要用一个 map 来记录这样的对象，你就需要用 IdentityHashMap，而不能使用其他 Map 来实现。
<pre><code class="language-java line-numbers"><script type="text/plain">public IdentityHashMap(); // 默认容量 32
public IdentityHashMap(int expectedMaxSize); // 指定初始容量
public IdentityHashMap(Map<? extends K, ? extends V> m); // 从其他 map 中获取 Entry

public int size(); // 获取元素个数
public boolean isEmpty(); // 判空
public void clear(); // 清空

public V get(Object key); // get 指定 value
public V put(K key, V value); // put 指定 key-value
public void putAll(Map<? extends K, ? extends V> m);
public V remove(Object key); // 移除

public boolean containsKey(Object key); // 判断是否存在指定 key
public boolean containsValue(Object value); // 判断是否存在指定 value

public boolean equals(Object o); // 判等 Object.equals() 方法
public int hashCode(); // 获取 hashCode 值

public Object clone(); // 拷贝当前对象

public Set<K> keySet(); // 获取 key 视图
public Collection<V> values(); // 获取 value 视图
public Set<Map.Entry<K,V>> entrySet(); // 获取 entry 视图

public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function); // 全部替换
public void forEach(BiConsumer<? super K, ? super V> action); // forEach 循环
</script></code></pre>



例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.IdentityHashMap;

public class Main {
    public static void main(String[] args) {
        IdentityHashMap<Integer, Integer> map = new IdentityHashMap<>();
        Integer[] array = {1, 2, 3, 100, 200, 300};
        for (Integer i : array) {
            map.put(i, i);
        }
        System.out.println(map);

        map.put(1, 11);
        map.put(2, 22);
        map.put(3, 33);
        System.out.println(map);

        map.put(100, 111);
        map.put(200, 222);
        map.put(300, 333);
        System.out.println(map);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [19:20:13]
$ javac Main.java

# root @ arch in ~/work on git:master x [19:20:34]
$ java Main
{1=1, 3=3, 300=300, 200=200, 100=100, 2=2}
{1=11, 3=33, 300=300, 200=200, 100=100, 2=22}
{1=11, 3=33, 300=333, 300=300, 200=200, 100=111, 200=222, 2=22}
</script></code></pre>



> 
Integer 自动装箱并非每次都是 new 一个新的 Integer 对象，自动装箱调用 Integer.valueOf() 方法；
valueOf() 首先检查 int 是否在区间 [-128, 127]，如果是则返回已存在的 Integer 对象(如果有的话)，否则 new 一个新的对象；

**WeakHashMap 类**
和 HashMap 一样，WeakHashMap 也是一个 key-value 映射表，并且键、值都可以为 null；
但是，WeakHashMap 的键(key)是**弱引用(WeakReference)**的，即"弱键"，但是值(value)是强引用的。

要明白 WeakHashMap 的特性，必须先了解 Java 的四种引用类型：
1) **强引用（StrongReference）**
一般情况下我们使用的都是强引用，如`Test t = new Test();`的 t 就是一个强引用；
> 当内存不足，JVM 宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会回收强引用对象来释放内存

2) **软引用（SoftReference）**
> 如果一个对象只有软引用，则内存空间足够，GC 就不会回收它；如果内存空间不足了，就会回收这些对象的内存

3) **弱引用（WeakReference）**
> 只具有弱引用的对象拥有更短暂的生命周期；在 GC 线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存

4) **虚引用（PhantomReference）**
> 虚引用主要用来跟踪对象被垃圾回收器回收的活动，虚引用必须和`引用队列（ReferenceQueue）`联合使用

“虚引用”顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期；如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。
当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

除了强引用之外，其他三种引用都在 java.lang.ref 包中。

这里解释一下**引用队列（ReferenceQueue）**：
> 
如果我们在创建一个引用对象时，指定了 ReferenceQueue，那么当引用对象指向的对象达到合适的状态（根据引用类型不同而不同）时，GC 会把引用对象本身添加到这个队列中，方便我们处理它；
因为引用对象指向的对象 GC 会自动清理，但是引用对象本身也是对象（是对象就占用一定资源），所以需要我们自己清理。

举个例子：`SoftReference<String> ss = new SoftReference<String>("abc", queue);`
其中，ss 是一个引用对象（一个普通的对象，本身是强引用），指向`"abc"`对象（该对象只有一个软引用 ss 指向它）；
`"abc"`对象会在一定时机被 GC 自动清理，但是 ss 对象本身的清理工作依赖于 queue，当 ss 出现在 queue 中时，说明其指向的对象已经无效，可以放心清理 ss 了。

**引用对象的四种状态**
每一时刻，Reference 对象都处于下面四种状态中：
1) `Active`："活动状态"，新创建的引用对象都是这个状态，GC 会根据引用对象是否在创建时指定 ReferenceQueue 参数进行状态转移，如果指定了，那么转移到 Pending 状态，如果没指定，转移到 Inactive 状态；
2) `Pending`："待定状态"，该状态的引用对象等着被内部线程 ReferenceHandler 处理（会调用 ReferenceQueue.enqueue 方法）
3) `Enqueued`："入队状态"，调用 ReferenceQueue.enqueued 方法后的引用对象处于这个状态中；
4) `Inactive`："死亡状态"，处于该状态的引用对象将被 GC 自动清理。

![java.lang.ref 引用对象的四种状态](/images/java-collection-weakref.png)

总之，我们目前只需要知道以下两点就可以了：
> 
如果构造函数中指定了 ReferenceQueue，那么引用对象需要手动进行清理；
如果构造函数中没有指定 ReferenceQueue，那么 GC 会自动清理引用对象。

**软引用、弱引用的应用场景**
软引用/弱引用都可用来实现内存敏感的高速缓存；通常和引用队列联合使用，如果引用对象所引用的对象被 GC 回收，Java 虚拟机就会把这个引用对象加入到与之关联的引用队列中，方便我们进行后需清理工作。

区别是：软引用的生命周期比弱引用的生命周期更长一些，因为软引用只有在内存不足时才会进行垃圾回收，而弱引用随时都可能被 GC 回收。

WeakHashMap 和 HashMap 最主要的区别就在于 Entry 类，WeakHashMap 的 Entry 中的 key 使用的是弱引用对象。

**主要方法**
<pre><code class="language-java line-numbers"><script type="text/plain">public WeakHashMap(int initialCapacity, float loadFactor); // 初始容量、负载因子
public WeakHashMap(int initialCapacity); // 初始容量，默认负载因子为 0.75f
public WeakHashMap(); // 默认初始容量为 16，默认负载因子为 0.75f
public WeakHashMap(Map<? extends K, ? extends V> m); // 使用指定map进行填充

public int size(); // 获取元素个数
public boolean isEmpty(); // 判空
public void clear(); // 清空

public V get(Object key); // get 指定 value
public V put(K key, V value); // put 指定 key-value
public void putAll(Map<? extends K, ? extends V> m); // put all
public V remove(Object key); // remove 指定 key-value

public boolean containsKey(Object key); // 判断是否存在指定 key
public boolean containsValue(Object value); // 判断是否存在指定 value

public Set<K> keySet(); // key 视图
public Collection<V> values(); // value 视图
public Set<Map.Entry<K,V>> entrySet(); // entry 视图

public void forEach(BiConsumer<? super K, ? super V> action); // JDK1.8 forEach迭代
public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function); // 全部替换
</script></code></pre>



**EnumMap 类**
1) EnumMap 是枚举的专属的 Map 键值对集合（key 必须为 Enum 类型）；
2) EnumMap 因为其内部是通过数组实现的，所以相对来说效率比普通的 Map 高；
3) EnumMap 虽说是枚举专属集合，但其操作与一般的 Map 差不多，都实现了 Map 接口；
<pre><code class="language-java line-numbers"><script type="text/plain">public EnumMap(Class<K> keyType); // 需要传入 key 的 Class 对象
public EnumMap(EnumMap<K, ? extends V> m); // 拷贝构造
public EnumMap(Map<K, ? extends V> m); // 从其他 map 获取 entry

public int size(); // 元素个数
public void clear(); // 清空

public boolean containsKey(Object key); // 是否包含指定 key
public boolean containsValue(Object value); // 是否包含指定 value

public V get(Object key); // get 指定 value
public V put(K key, V value); // put 指定 key-value
public void putAll(Map<? extends K, ? extends V> m); // put all
public V remove(Object key); // remove 指定 key-value

public Set<K> keySet(); // key 视图
public Collection<V> values(); // value 视图
public Set<Map.Entry<K,V>> entrySet(); // entry 视图

public boolean equals(Object o);
public int hashCode();

public EnumMap<K, V> clone();
</script></code></pre>



## Set 集合
HashSet、LinkedHashSet、TreeSet 都是其对应的 Map 映射的一个包装类；
因此它们的相关特性也和底层容器一样，它们只使用了 key-value 键值对的 key 部分。

**Set 接口**
<pre><code class="language-java line-numbers"><script type="text/plain">int size(); // 元素个数
boolean isEmpty(); // 判空
void clear(); // 清空集合

boolean equals(Object o);
int hashCode();

boolean contains(Object o); // 测试是否包含指定元素

Iterator<E> iterator(); // 获取迭代器

Object[] toArray(); // 转换为 Object[] 数组
<T> T[] toArray(T[] a); // 转换为指定类型数组

boolean add(E e); // 添加元素到此集合, 如果添加成功返回true, 已存在则返回false
boolean remove(Object o); // 移除指定元素, 删除成功返回true

/*
 * 如果此集合包含指定集合 c 中的所有元素, 则返回 true
 * 如果集合 c 是一个 Set，表示集合 c 是当前集合的子集
 */
boolean containsAll(Collection<?> c);

/*
 * 将指定集合 c 中的所有元素添加自当前集合中
 * 如果集合 c 是一个 Set，那么可用于并集操作
 */
boolean addAll(Collection<? extends E> c);

/*
 * 仅保留此集合中包含在指定集合 c 中的元素
 * 如果集合 c 是一个 Set，那么当前集合将是它们的交集
 */
boolean retainAll(Collection<?> c);

/*
 * 删除此集合中包含在指定集合 c 中的所有元素
 * 如果集合 c 是一个 Set，那么当前集合将是集合 c 相对于当前集合的补集
 */
boolean removeAll(Collection<?> c);
</script></code></pre>



**HashSet 类**
<pre><code class="language-java line-numbers"><script type="text/plain">public HashSet(int initialCapacity, float loadFactor);
public HashSet(int initialCapacity);
public HashSet();
public HashSet(Collection<? extends E> c);

public Iterator<E> iterator(); // 获取迭代器
public int size(); // 元素个数
public boolean isEmpty();
public void clear();

public boolean contains(Object o);

public boolean add(E e);
public boolean remove(Object o);

public Object clone();
</script></code></pre>



**LinkedHashSet 类**
<pre><code class="language-java line-numbers"><script type="text/plain">public LinkedHashSet(int initialCapacity, float loadFactor);
public LinkedHashSet(int initialCapacity);
public LinkedHashSet();
public LinkedHashSet(Collection<? extends E> c);
</script></code></pre>



**TreeSet 类**
<pre><code class="language-java line-numbers"><script type="text/plain">public TreeSet();
public TreeSet(Comparator<? super E> comparator);
public TreeSet(Collection<? extends E> c);
public TreeSet(SortedSet<E> s);

public Iterator<E> iterator();
public Iterator<E> descendingIterator();

public NavigableSet<E> descendingSet();

public int size();
public boolean isEmpty();
public void clear();
public Comparator<? super E> comparator();

public boolean contains(Object o);

public boolean add(E e);
public boolean addAll(Collection<? extends E> c);
public boolean remove(Object o);

// 提取子 set
public NavigableSet<E> subSet(E fromElement, boolean fromInclusive, E toElement, boolean toInclusive);
public NavigableSet<E> headSet(E toElement, boolean inclusive); // 提取 head 部分
public NavigableSet<E> tailSet(E fromElement, boolean inclusive); // 提取 tail 部分

public SortedSet<E> subSet(E fromElement, E toElement); // [from, to)
public SortedSet<E> headSet(E toElement); // [first, to)
public SortedSet<E> tailSet(E fromElement); // [from, last]

public E first(); // 获取第一个元素，set 为空则抛出 NoSuchElementException 异常
public E last(); // 获取最后一个元素，set 为空则抛出 NoSuchElementException 异常
public E pollFirst(); // 弹出
public E pollLast(); // 弹出

public E lower(E e); // "小于"
public E floor(E e); // "小于等于"
public E higher(E e); // "大于"
public E ceiling(E e); // "大于等于"

public Object clone();
</script></code></pre>



**EnumSet 抽象类**
EnumSet 和 EnumMap 一样，都是枚举专用（JDK1.5 引入了 Enum 枚举类型）的容器。

EnumSet 是一个抽象类，它有两个实现类（实现类的访问权限是包权限，因此无法在外部直接访问）：RegularEnumSet、JumboEnumSet。

EnumSet 抽象类提供了几个静态方法，用于创建这两个实现类的对象，具体使用谁由 EnumSet 内部决定。

1) EnumSet 是与枚举类型一起使用的专用 Set 集合，EnumSet 中所有元素都必须是 Enum 枚举类型；
2) EnumSet 与其他 Set 接口的实现类 HashSet/TreeSet（内部都是用对应的 HashMap/TreeMap 实现的）不同的是，EnumSet 的内部实现是**位向量**，它是一种极为高效的位运算操作，由于直接存储和操作都是 bit，因此 EnumSet 空间和时间性能都十分可观，足以媲美传统上基于 int 的“位标志”的运算；
3) EnumSet 不允许使用 null 元素；试图插入 null 元素将抛出 NullPointerException，但试图测试判断是否存在 null 元素或移除 null 元素则不会抛出异常；
<pre><code class="language-java line-numbers"><script type="text/plain">// 创建一个空 set 集合
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType);
// 创建一个包含所有枚举常量的 set 集合
public static <E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType);
// 创建 s 的补集
public static <E extends Enum<E>> EnumSet<E> complementOf(EnumSet<E> s);

// 拷贝构造
public static <E extends Enum<E>> EnumSet<E> copyOf(EnumSet<E> s);
public static <E extends Enum<E>> EnumSet<E> copyOf(Collection<E> c);

// 创建一个包含指定元素的 set 集合
public static <E extends Enum<E>> EnumSet<E> of(E e);
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2);
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3);
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4);
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4, E e5);
public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest); // 可变参数(本质为数组)

// 指定范围(由枚举类型的 ordinal 决定)
public static <E extends Enum<E>> EnumSet<E> range(E from, E to);

public EnumSet<E> clone(); // 克隆
</script></code></pre>
