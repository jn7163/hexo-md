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



## Collections
java.util.Collections 类是 Java 集合框架的一员，它提供了很多 static 方法服务于各集合对象；
如：对集合元素的排序、取极值、批量拷贝、集合结构转换、循环移位以及匹配性检查等功能；
> 因为 Collections 类的构造方法被声明为 private，因此无法进行对象的实例化。

**常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">// 调用 list.sort(null) [自然排序]
public static <T extends Comparable<? super T>> void sort(List<T> list);
// 调用 list.sort(c) [自定义排序]
public static <T> void sort(List<T> list, Comparator<? super T> c);

// 使用二分查找算法，该 list 必须是按照自然排序的升序列表，否则结果是未定义的
public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key);
// 使用二分查找算法，该 list 必须是按照指定排序的升序列表，否则结果是未定义的
public static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c);

public static void reverse(List<?> list); // 翻转 list

public static void shuffle(List<?> list); // 随机排列 list
public static void shuffle(List<?> list, Random rnd); // 指定随机源

public static void swap(List<?> list, int i, int j); // 交换两个指定索引的元素

public static <T> void fill(List<? super T> list, T obj); // 使用指定 obj 填充 list

public static <T> void copy(List<? super T> dest, List<? extends T> src); // 拷贝 list

// 返回自然排序情况下的 minimum 元素，元素必须实现 Comparable 接口
public static <T extends Object & Comparable<? super T>> T min(Collection<? extends T> coll);
// 返回指定比较器下的 minimum 元素，需要额外提供 Comparator 比较器
public static <T> T min(Collection<? extends T> coll, Comparator<? super T> comp);

public static <T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll);
public static <T> T max(Collection<? extends T> coll, Comparator<? super T> comp);

public static void rotate(List<?> list, int distance); // 旋转元素

public static <T> boolean replaceAll(List<T> list, T oldVal, T newVal); // 将指定 oldVal 全部替换为 newVal

// 返回指定 target 子列表在 source 源列表中首次匹配的索引位置，没有匹配则返回 -1
public static int indexOfSubList(List<?> source, List<?> target);
public static int lastIndexOfSubList(List<?> source, List<?> target); // 最后一次匹配的索引位置

// 获取不可修改的集合，尝试修改将导致 UnsupportedOperationException 异常
public static <T> Collection<T> unmodifiableCollection(Collection<? extends T> c);
public static <T> List<T> unmodifiableList(List<? extends T> list);
public static <T> Set<T> unmodifiableSet(Set<? extends T> s);
public static <T> SortedSet<T> unmodifiableSortedSet(SortedSet<T> s);
public static <T> NavigableSet<T> unmodifiableNavigableSet(NavigableSet<T> s);
public static <K,V> Map<K,V> unmodifiableMap(Map<? extends K, ? extends V> m);
public static <K,V> SortedMap<K,V> unmodifiableSortedMap(SortedMap<K, ? extends V> m);
public static <K,V> NavigableMap<K,V> unmodifiableNavigableMap(NavigableMap<K, ? extends V> m);

// 获取线程安全的集合
public static <T> Collection<T> synchronizedCollection(Collection<T> c);
public static <T> List<T> synchronizedList(List<T> list);
public static <T> Set<T> synchronizedSet(Set<T> s);
public static <T> SortedSet<T> synchronizedSortedSet(SortedSet<T> s);
public static <T> NavigableSet<T> synchronizedNavigableSet(NavigableSet<T> s);
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m);
public static <K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m);
public static <K,V> NavigableMap<K,V> synchronizedNavigableMap(NavigableMap<K,V> m);

// 返回类型安全的集合
public static <E> Collection<E> checkedCollection(Collection<E> c, Class<E> type);
public static <E> List<E> checkedList(List<E> list, Class<E> type);
public static <E> Queue<E> checkedQueue(Queue<E> queue, Class<E> type);
public static <E> Set<E> checkedSet(Set<E> s, Class<E> type);
public static <E> SortedSet<E> checkedSortedSet(SortedSet<E> s, Class<E> type);
public static <E> NavigableSet<E> checkedNavigableSet(NavigableSet<E> s, Class<E> type);
public static <K, V> Map<K, V> checkedMap(Map<K, V> m, Class<K> keyType, Class<V> valueType);
public static <K,V> SortedMap<K,V> checkedSortedMap(SortedMap<K, V> m, Class<K> keyType, Class<V> valueType);
public static <K,V> NavigableMap<K,V> checkedNavigableMap(NavigableMap<K, V> m, Class<K> keyType, Class<V> valueType);

public static <T> Iterator<T> emptyIterator(); // 获取一个没有任何元素的空迭代器
public static <T> ListIterator<T> emptyListIterator(); // ListIterator
public static <T> Enumeration<T> emptyEnumeration(); // 返回没有元素的枚举
public static final <T> List<T> emptyList();
public static final <T> Set<T> emptySet();
public static <E> SortedSet<E> emptySortedSet();
public static <E> NavigableSet<E> emptyNavigableSet();
public static final <K,V> Map<K,V> emptyMap();
public static final <K,V> SortedMap<K,V> emptySortedMap();
public static final <K,V> NavigableMap<K,V> emptyNavigableMap();

// 返回一个只包含指定对象的不可变集合
public static <T> List<T> singletonList(T o);
public static <T> Set<T> singleton(T o);
public static <K,V> Map<K,V> singletonMap(K key, V value);
// 返回一个包含指定多个副本的不可变集合
public static <T> List<T> nCopies(int n, T o);

public static <T> Comparator<T> reverseOrder(); // 获取自然排序的降序比较器
public static <T> Comparator<T> reverseOrder(Comparator<T> cmp); // 获取指定比较器的降序比较器

public static <T> Enumeration<T> enumeration(final Collection<T> c); // 返回指定集合的枚举
public static <T> ArrayList<T> list(Enumeration<T> e); // 返回指定枚举的 list 列表

public static int frequency(Collection<?> c, Object o); // 统计集合中指定元素的个数
public static boolean disjoint(Collection<?> c1, Collection<?> c2); // 如果两个指定集合没有相同的元素则返回true

public static <T> boolean addAll(Collection<? super T> c, T... elements); // 添加所有传入的元素

public static <E> Set<E> newSetFromMap(Map<E, Boolean> map); // 从 map 中创建对应的 set 集合
public static <T> Queue<T> asLifoQueue(Deque<T> deque); // 返回一个 LIFO 双端队列
</script></code></pre>



## Collection 总结
经过前面大量篇幅的学习，现在我们已经熟悉了 Java Collection 框架的基本使用和各种容器的优点、缺点、特性、适用场景等。
但是仅仅学习怎么使用还是不够的，我们不仅要会用，更应该去学习其实现原理，因此，现在就来实现一个简单的 Collection 框架吧。
> 
github 仓库：[Java - Collection 框架的简单实现](https://github.com/zfl9/java-collection)

**ArrayList**
"动态数组"，其实就是 built-in 数组的包装，其最主要的功能就是要实现容量的自动增长。
<pre><code class="language-java line-numbers"><script type="text/plain">package com.zfl9.collection;

import java.util.Arrays;
import java.util.Iterator;
import java.lang.reflect.Array;

public class ArrayList<E> implements Iterable<E> {
    private Object[] array; // Object[] 数组，用于存放所有元素
    private int size; // 元素的数量
    private final int DEFAULT_CAPACITY = 10; // 默认容量(数组的长度)

    /* 构造函数 */
    public ArrayList() {
        array = new Object[DEFAULT_CAPACITY];
    }
    public ArrayList(int capacity) {
        array = new Object[capacity];
    }

    public int size() { // 获取元素个数
        return size;
    }
    public int capacity() { // 获取当前数组的容量(数组的长度)
        return array.length;
    }
    public boolean empty() { // 判断是否为空数组
        return size == 0;
    }
    public void clear() { // 清空所有元素
        size = 0;
        array = null;
    }

    /* 直接扩容到指定大小 */
    private void manualGrow(int capacity) {
        array = Arrays.copyOf(array, capacity);
    }
    /* 自动扩容 1.5 倍增长 */
    private void autoGrow(int capacity) {
        if (capacity > array.length) {
            capacity *= 1.5;
            array = Arrays.copyOf(array, capacity);
        }
    }

    // 扩容, 只有当 capacity 参数大于当前容量时生效
    public void ensureCapacity(int capacity) {
        if (capacity > array.length) {
            manualGrow(capacity);
        }
    }
    // 缩减容量(回收所有未填充元素的空间)
    public void trimToSize() {
        manualGrow(size);
    }

    /* getter、setter 方法 */
    @SuppressWarnings("unchecked")
    public E get(int index) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("get(), index: %d, size: %d", index, size));
        }
        return (E) array[index];
    }
    @SuppressWarnings("unchecked")
    public E set(int index, E elem) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("set(), index: %d, size: %d", index, size));
        }
        E oldElem = (E) array[index];
        array[index] = elem;
        return oldElem;
    }

    /* 在尾部插入、删除 */
    public void append(E elem) {
        autoGrow(size + 1);
        array[size++] = elem;
    }
    @SuppressWarnings("unchecked")
    public E pop() {
        if (size != 0) {
            E elem = (E) array[size-- - 1];
            array[size] = null;
            return elem;
        } else {
            return null;
        }
    }

    /* 在任意位置插入、删除 */
    public void insert(int index, E elem) {
        if (index > size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("insert(), index: %d, size: %d", index, size));
        }
        autoGrow(size++ + 1);
        for (int i = size - 1; i > index; i--) {
            array[i] = array[i - 1];
        }
        array[index] = elem;
    }
    @SuppressWarnings("unchecked")
    public E delete(int index) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("delete(), index: %d, size: %d", index, size));
        }
        E elem = (E) array[index];
        size--;
        for (int i = index; i < size; i++) {
            array[i] = array[i + 1];
        }
        array[size] = null;
        return elem;
    }

    /* 查找指定元素首次出现的索引 */
    public int find(E elem) {
        for (int i = 0; i < size; i++) {
            if (elem.equals(array[i])) {
                return i;
            }
        }
        return -1;
    }

    /* 升序、降序 */
    public void sort() {
        Arrays.sort(array, 0, size);
    }
    @SuppressWarnings("unchecked")
    public void reverseSort() {
        Arrays.sort(array, 0, size, (Object o1, Object o2) -> {
            return ((Comparable<? super E>) o2).compareTo((E) o1);
        });
    }

    /* 转换为 built-in 数组 */
    @SuppressWarnings("unchecked")
    public E[] toArray() {
        if (size == 0) {
            return null;
        }
        E[] array = (E[]) Array.newInstance(this.array[0].getClass(), size);
        System.arraycopy(this.array, 0, array, 0, size);
        return array;
    }

    @Override
    public String toString() {
        if (size == 0) {
            return "(null)";
        } else {
            StringBuilder sb = new StringBuilder("{ ");
            for (int i = 0; i < size; i++) {
                sb.append(array[i] + ", ");
            }
            sb.append("\b\b }");
            return sb.toString();
        }
    }

    @Override
    public Iterator<E> iterator() { // 获取迭代器
        return new Itr();
    }

    private class Itr implements Iterator<E> {
        private int cursor;

        @Override
        public boolean hasNext() {
            return cursor < size;
        }

        @Override
        @SuppressWarnings("unchecked")
        public E next() {
            return (E) array[cursor++];
        }
    }
}
</script></code></pre>



**LinkList**
"单向链表"，只有 next 节点，每次访问元素只能从 HEAD 头结点开始遍历。
<pre><code class="language-java line-numbers"><script type="text/plain">package com.zfl9.collection;

import java.util.Arrays;
import java.util.Iterator;
import java.lang.reflect.Array;

public class LinkList<E> implements Iterable<E> {
    // 节点 Node
    private static class Node<E> {
        public E data; // 数据域
        public Node<E> next; // 指针域

        public Node(E data, Node<E> next) {
            this.data = data;
            this.next = next;
        }
    }

    private final Node<E> HEAD = new Node<E>(null, null); // 头结点, 数据域为 null
    private int size; // 元素个数

    public LinkList() {}

    public int size() {
        return size;
    }
    public boolean empty() {
        return HEAD.next == null;
    }
    public void clear() {
        for (Node<E> temp; HEAD.next != null;) {
            temp = HEAD.next.next;
            HEAD.next.data = null;
            HEAD.next.next = null;
            HEAD.next = temp;
        }
        size = 0;
    }

    /* 在链表尾部插入、删除节点 */
    public void append(E elem) {
        Node<E> node = HEAD;
        for (; node.next != null; node = node.next);
        node.next = new Node<E>(elem, null);
        size++;
    }
    public E pop() {
        if (HEAD.next == null) {
            throw new IndexOutOfBoundsException(String.format("pop(), size: %d\n", size));
        }
        Node<E> node = HEAD;
        for (; node.next.next != null; node = node.next);
        E elem = node.next.data;
        node.next.data = null;
        node.next = null;
        size--;
        return elem;
    }

    /* 在链表任意位置插入、删除节点 */
    public void insert(int index, E elem) {
        if (index > size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("insert(), index: %d, size: %d", index, size));
        }
        Node<E> node = HEAD;
        for (int i = 0; i < index; i++, node = node.next);
        node.next = new Node<E>(elem, node.next);
        size++;
    }
    public E delete(int index) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("delete(), index: %d, size: %d", index, size));
        }
        Node<E> node = HEAD;
        for (int i = 0; i < index; i++, node = node.next);
        Node<E> temp = node.next.next;
        E elem = node.next.data;
        node.next.data = null;
        node.next.next = null;
        node.next = temp;
        size--;
        return elem;
    }

    /* getter、setter */
    public E get(int index) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("get(), index: %d, size: %d", index, size));
        }
        Node<E> node = HEAD;
        for (int i = 0; i < index; i++, node = node.next);
        return node.next.data;
    }
    public E set(int index, E elem) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("set(), index: %d, size: %d", index, size));
        }
        Node<E> node = HEAD;
        for (int i = 0; i < index; i++, node = node.next);
        E oldElem = node.next.data;
        node.next.data = elem;
        return oldElem;
    }

    /* 查找指定元素首次出现的索引 */
    public int find(E elem) {
        Node<E> node = HEAD;
        for (int i = 0; i < size; i++, node = node.next) {
            if (elem.equals(node.next.data)) {
                return i;
            }
        }
        return -1;
    }

    /* 升序、降序 */
    public void sort() {
        sort(true);
    }
    public void reverseSort() {
        sort(false);
    }

    @SuppressWarnings("unchecked")
    private void sort(boolean ascending) {
        boolean sorted = true;
        Node<E> node = null;
        for (int i = 0; i < size - 1; i++) {
            node = HEAD.next;
            for (int j = 0; j < size - 1 - i; j++, node = node.next) {
                if (ascending ?
                    ((Comparable<? super E>)node.data).compareTo(node.next.data) > 0 :
                    ((Comparable<? super E>)node.next.data).compareTo(node.data) > 0)
                {
                    sorted = false;
                    E temp = node.data;
                    node.data = node.next.data;
                    node.next.data = temp;
                }
            }
            if (sorted) {
                break;
            }
        }
    }

    /* 转换为 built-in 数组 */
    @SuppressWarnings("unchecked")
    public E[] toArray() {
        if (size == 0) {
            return null;
        }
        E[] array = (E[]) Array.newInstance(HEAD.next.data.getClass(), size);
        Node<E> node = HEAD.next;
        for (int i = 0; i < size; i++, node = node.next) {
            array[i] = node.data;
        }
        return array;
    }

    @Override
    public String toString() {
        if (HEAD.next == null) {
            return "(null)";
        }
        StringBuilder sb = new StringBuilder("{ ");
        for (Node<E> node = HEAD; node.next != null; node = node.next) {
            sb.append(node.next.data + ", ");
        }
        sb.append("\b\b }");
        return sb.toString();
    }

    /* 获取迭代器 */
    @Override
    public Iterator<E> iterator() {
        return new Itr();
    }

    private class Itr implements Iterator<E> {
        private Node<E> node = HEAD;

        @Override
        public boolean hasNext() {
            return node.next != null;
        }

        @Override
        public E next() {
            node = node.next;
            return node.data;
        }
    }
}
</script></code></pre>



**DLinkList**
"双向链表"，有 prev/next 两个节点，头结点 HEAD、尾节点 TAIL；头结点的 prev 为 null，尾节点的 next 为 null。
<pre><code class="language-java line-numbers"><script type="text/plain">package com.zfl9.collection;

import java.util.Iterator;
import java.lang.reflect.Array;

public class DLinkList<E> implements Iterable<E> {
    // 节点 Node
    private static class Node<E> {
        public E data; // 数据域
        public Node<E> prev; // prev 指针
        public Node<E> next; // next 指针

        public Node(Node<E> prev, E data, Node<E> next) {
            this.prev = prev;
            this.data = data;
            this.next = next;
        }
    }

    /* 头尾结点 */
    private final Node<E> HEAD = new Node<E>(null, null, null);
    private final Node<E> TAIL = new Node<E>(null, null, null);
    private int size;

    public DLinkList() {
        HEAD.next = TAIL;
        TAIL.prev = HEAD;
    }

    public int size() {
        return size;
    }
    public boolean empty() {
        return HEAD.next == TAIL;
    }
    public void clear() {
        for (Node<E> temp; HEAD.next != TAIL;) {
            temp = HEAD.next.next;
            HEAD.next.prev = null;
            HEAD.next.data = null;
            HEAD.next.next = null;
            HEAD.next = temp;
            temp.prev = HEAD;
        }
        size = 0;
    }

    /* 在链表尾部插入、删除 */
    public void append(E elem) {
        TAIL.prev = new Node<E>(TAIL.prev, elem, TAIL);
        TAIL.prev.prev.next = TAIL.prev;
        size++;
    }
    public E pop() {
        if (HEAD.next == TAIL) {
            throw new IndexOutOfBoundsException(String.format("pop(), size: %d", size));
        }
        E elem = TAIL.prev.data;
        Node<E> temp = TAIL.prev.prev;
        TAIL.prev.prev = null;
        TAIL.prev.data = null;
        TAIL.prev.next = null;
        TAIL.prev = temp;
        temp.next = TAIL;
        size--;
        return elem;
    }

    /* 在任意位置插入、删除 */
    public void insert(int index, E elem) {
        if (index > size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("insert(), index: %d, size: %d", index, size));
        }
        if (index < size / 2) {
            Node<E> node = HEAD;
            for (int i = 0; i < index; i++, node = node.next);
            node.next = new Node<E>(node, elem, node.next);
            node.next.next.prev = node.next;
            size++;
        } else {
            Node<E> node = TAIL;
            for (int i = 0; i < size - index; i++, node = node.prev);
            node.prev = new Node<E>(node.prev, elem, node);
            node.prev.prev.next = node.prev;
            size++;
        }
    }
    public E delete(int index) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("delete(), index: %d, size: %d", index, size));
        }
        if (index < size / 2) {
            Node<E> node = HEAD;
            for (int i = 0; i < index; i++, node = node.next);
            E elem = node.next.data;
            Node<E> temp = node.next.next;
            node.next.prev = null;
            node.next.data = null;
            node.next.next = null;
            node.next = temp;
            temp.prev = node;
            size--;
            return elem;
        } else {
            Node<E> node = TAIL;
            for (int i = 0; i < size - index - 1; i++, node = node.prev);
            E elem = node.prev.data;
            Node<E> temp = node.prev.prev;
            node.prev.prev = null;
            node.prev.data = null;
            node.prev.next = null;
            node.prev = temp;
            temp.next = node;
            size--;
            return elem;
        }
    }

    /* getter、setter */
    public E get(int index) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("get(), index: %d, size: %d", index, size));
        }
        if (index < size / 2) {
            Node<E> node = HEAD.next;
            for (int i = 0; i < index; i++, node = node.next);
            return node.data;
        } else {
            Node<E> node = TAIL.prev;
            for (int i = 0; i < size - index - 1; i++, node = node.prev);
            return node.data;
        }
    }
    public E set(int index, E elem) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException(String.format("set(), index: %d, size: %d", index, size));
        }
        if (index < size / 2) {
            Node<E> node = HEAD.next;
            for (int i = 0; i < index; i++, node = node.next);
            E oldElem = node.data;
            node.data = elem;
            return oldElem;
        } else {
            Node<E> node = TAIL.prev;
            for (int i = 0; i < size - index - 1; i++, node = node.prev);
            E oldElem = node.data;
            node.data = elem;
            return oldElem;
        }
    }

    /* 查找指定元素首次出现的索引 */
    public int find(E elem) {
        Node<E> node = HEAD;
        for (int i = 0; i < size; i++, node = node.next) {
            if (node.next.data.equals(elem)) {
                return i;
            }
        }
        return -1;
    }

    /* 升序、降序 */
    public void sort() {
        sort(true);
    }
    public void descSort() {
        sort(false);
    }
    @SuppressWarnings("unchecked")
    private void sort(boolean ascending) {
        boolean sorted = true;
        Node<E> node = null;
        for (int i = 0; i < size - 1; i++) {
            node = HEAD.next;
            for (int j = 0; j < size - 1 - i; j++, node = node.next) {
                if (ascending ?
                    ((Comparable<? super E>) node.data).compareTo(node.next.data) > 0 :
                    ((Comparable<? super E>) node.next.data).compareTo(node.data) > 0)
                {
                    sorted = false;
                    E temp = node.data;
                    node.data = node.next.data;
                    node.next.data = temp;
                }
            }
            if (sorted) {
                break;
            }
        }
    }

    /* 转换为 built-in 数组 */
    @SuppressWarnings("unchecked")
    public E[] toArray() {
        if (size == 0) {
            return null;
        }
        E[] array = (E[]) Array.newInstance(HEAD.next.data.getClass(), size);
        Node<E> node = HEAD.next;
        for (int i = 0; i < size; i++, node = node.next) {
            array[i] = node.data;
        }
        return array;
    }

    @Override
    public String toString() { // 正向遍历
        if (HEAD.next == TAIL) {
            return "(null)";
        }
        StringBuilder sb = new StringBuilder("{ ");
        Node<E> node = HEAD.next;
        for (int i = 0; i < size; i++, node = node.next) {
            sb.append(node.data + ", ");
        }
        sb.append("\b\b }");
        return sb.toString();
    }
    public String toStringReversed() { // 逆向遍历
        if (TAIL.prev == HEAD) {
            return "(null)";
        }
        StringBuilder sb = new StringBuilder("{ ");
        Node<E> node = TAIL.prev;
        for (int i = 0; i < size; i++, node = node.prev) {
            sb.append(node.data + ", ");
        }
        sb.append("\b\b }");
        return sb.toString();
    }

    @Override
    public Iterator<E> iterator() { // 正向
        return new Itr(true);
    }
    public Iterator<E> descIterator() { // 逆向
        return new Itr(false);
    }

    private class Itr implements Iterator<E> {
        private boolean ascending;
        private Node<E> node;

        public Itr(boolean ascending) {
            this.ascending = ascending;
            node = ascending ? HEAD : TAIL;
        }

        @Override
        public boolean hasNext() {
            return ascending ? node.next != TAIL : node.prev != HEAD;
        }

        @Override
        public E next() {
            node = ascending ? node.next : node.prev;
            return node.data;
        }
    }
}
</script></code></pre>



**HashMap**
"哈希表"，使用 hash() 函数计算 key 的 hash 值，使用 indexFor() 函数计算指定 hash 的数组索引，使用"拉链法"解决 hash 冲突。
<pre><code class="language-java line-numbers"><script type="text/plain">package com.zfl9.collection;

import java.util.Arrays;
import java.util.Iterator;
import java.lang.reflect.Array;

public class HashMap<K, V> implements Iterable<HashMap.Entry<K, V>> {
    // 键值对 key-value，称为记录 Entry
    public static class Entry<K, V> {
        int hash; // 记录 hash
        K key; // 记录 key
        V value; // 记录 value
        Entry<K, V> next; // next 指针

        Entry(int hash, K key, V value, Entry<K, V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        /* getter、setter */
        public K getKey() {
            return key;
        }
        public V getValue() {
            return value;
        }
        public V setValue(V value) {
            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }
    }

    Entry<K, V>[] table; // Entry[] 数组
    int capacity; // 数组的容量
    static final int DEFAULT_CAPACITY = 32; // 默认为 32
    float loadFactor; // 负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f; // 默认为 0.75
    int size;

    /* 构造函数 */
    public HashMap(int capacity, float loadFactor) {
        if (capacity % 2 != 0 || capacity <= 0) {
            throw new IllegalArgumentException("illegal capacity: " + capacity);
        }
        if (loadFactor <= 0) {
            throw new IllegalArgumentException("illegal loadFactor: " + loadFactor);
        }
        this.capacity = capacity;
        this.loadFactor = loadFactor;
        @SuppressWarnings({"rawtypes", "unchecked"})
        Entry<K, V>[] table = (Entry<K, V>[]) new Entry[capacity];
        this.table = table;
    }
    public HashMap(int capacity) {
        this(capacity, DEFAULT_LOAD_FACTOR);
    }
    public HashMap() {
        this(DEFAULT_CAPACITY, DEFAULT_LOAD_FACTOR);
    }

    public int size() {
        return size;
    }
    public int capacity() {
        return capacity;
    }
    public float loadFactor() {
        return loadFactor;
    }
    public boolean empty() {
        return size == 0;
    }
    public void clear() {
        Entry<K, V> temp;
        for (int i = 0; i < capacity; i++) {
            while (table[i] != null) {
                temp = table[i].next;
                table[i].hash = 0;
                table[i].key = null;
                table[i].value = null;
                table[i].next = null;
                table[i] = temp;
            }
        }
        size = 0;
    }

    int hash(K key) { // 计算 key 的散列值，参考自 JDK1.8 源码
        int hash;
        return key == null ? 0 : (hash = key.hashCode()) ^ (hash >>> 16);
    }
    int indexFor(int hash) { // 根据 hash 计算对应的数组索引
        return hash & (capacity - 1);
    }

    /* 自动扩容 */
    private void autoGrow(int minSize) {
        if (minSize > capacity * loadFactor) {
            table = Arrays.copyOf(table, capacity * 2);
            capacity = table.length;
            Entry<K, V> head = new Entry<K, V>(0, null, null, null), entry, temp;
            int index;
            for (int i = 0; i < capacity / 2; i++) {
                head.next = table[i];
                for (entry = head; entry.next != null;) {
                    if ((index = indexFor(entry.next.hash)) != i) {
                        temp = entry.next;
                        entry.next = entry.next.next;
                        temp.next = table[index];
                        table[index] = temp;
                    } else {
                        entry = entry.next;
                    }
                }
                table[i] = head.next;
            }
        }
    }

    // 添加 key-value 对
    public V put(K key, V value) {
        int hash = hash(key);
        int index = indexFor(hash);
        for (Entry<K, V> entry = table[index]; entry != null; entry = entry.next) {
            if (entry.hash == hash) {
                V oldValue = entry.value;
                entry.value = value;
                return oldValue;
            }
        }
        autoGrow(size + 1);
        index = indexFor(hash);
        table[index] = new Entry<K, V>(hash, key, value, table[index]);
        size++;
        return null;
    }
    // 获取指定 value
    public V get(K key) {
        int hash = hash(key);
        int index = indexFor(hash);
        for (Entry<K, V> entry = table[index]; entry != null; entry = entry.next) {
            if (entry.hash == hash) {
                return entry.value;
            }
        }
        return null;
    }
    // 移除指定 key-value
    public V remove(K key) {
        int hash = hash(key);
        int index = indexFor(hash);
        if (table[index] == null) {
            return null;
        }
        Entry<K, V> head = new Entry<K, V>(0, null, null, table[index]), entry, temp;
        for (entry = head; entry.next != null; entry = entry.next) {
            if (entry.next.hash == hash) {
                temp = entry.next.next;
                V value = entry.next.value;
                entry.next.hash = 0;
                entry.next.key = null;
                entry.next.value = null;
                entry.next.next = null;
                entry.next = temp;
                table[index] = head.next;
                size--;
                return value;
            }
        }
        return null;
    }

    // 判断是否存在指定 key、value
    public boolean hasKey(K key) {
        int hash = hash(key);
        int index = indexFor(hash);
        for (Entry<K, V> entry = table[index]; entry != null; entry = entry.next) {
            if (entry.hash == hash) {
                return true;
            }
        }
        return false;
    }
    public boolean hasValue(V value) {
        for (int i = 0; i < capacity; i++) {
            for (Entry<K, V> entry = table[i]; entry != null; entry = entry.next) {
                if (entry.value.equals(value)) {
                    return true;
                }
            }
        }
        return false;
    }

    // 获取一个包含所有 key 的数组
    public K[] keys() {
        if (size == 0) {
            return null;
        }
        K key = null;
        for (int i = 0; i < capacity; i++) {
            for (Entry<K, V> entry = table[i]; entry != null; entry = entry.next) {
                if (entry.key != null) {
                    key = entry.key;
                    break;
                }
            }
            if (key != null) {
                break;
            }
        }
        @SuppressWarnings("unchecked")
        K[] array = (K[]) Array.newInstance(key.getClass(), size);
        int index = 0;
        for (int i = 0; i < capacity; i++) {
            for (Entry<K, V> entry = table[i]; entry != null; entry = entry.next) {
                array[index++] = entry.key;
            }
        }
        return array;
    }
    // 获取一个包含所有 value 的数组
    public V[] values() {
        if (size == 0) {
            return null;
        }
        V value = null;
        for (int i = 0; i < capacity; i++) {
            for (Entry<K, V> entry = table[i]; entry != null; entry = entry.next) {
                if (entry.value != null) {
                    value = entry.value;
                    break;
                }
            }
            if (value != null) {
                break;
            }
        }
        @SuppressWarnings("unchecked")
        V[] array = (V[]) Array.newInstance(value.getClass(), size);
        int index = 0;
        for (int i = 0; i < capacity; i++) {
            for (Entry<K, V> entry = table[i]; entry != null; entry = entry.next) {
                array[index++] = entry.value;
            }
        }
        return array;
    }

    @Override
    public String toString() {
        if (size == 0) {
            return "(null)";
        }
        StringBuilder sb = new StringBuilder("{ ");
        Entry<K, V> entry;
        for (int i = 0; i < capacity; i++) {
            for (entry = table[i]; entry != null; entry = entry.next) {
                sb.append("{" + entry.key + ", " + entry.value + "}, ");
            }
        }
        sb.append("\b\b }");
        return sb.toString();
    }

    @Override
    public Iterator<HashMap.Entry<K, V>> iterator() { // 获取迭代器
        return new Itr();
    }

    private class Itr implements Iterator<HashMap.Entry<K, V>> {
        @SuppressWarnings({"rawtypes", "unchecked"})
        private HashMap.Entry<K, V>[] array = (HashMap.Entry<K, V>[]) new HashMap.Entry[size];
        private int cursor;

        {
            int index = 0;
            for (int i = 0; i < capacity; i++) {
                for (Entry<K, V> entry = table[i]; entry != null; entry = entry.next) {
                    array[index++] = entry;
                }
            }
        }

        @Override
        public boolean hasNext() {
            return cursor < size;
        }

        @Override
        public HashMap.Entry<K, V> next() {
            return array[cursor++];
        }
    }
}
</script></code></pre>



**HashSet 包装类**
<pre><code class="language-java line-numbers"><script type="text/plain">package com.zfl9.collection;

import java.util.Iterator;

public class HashSet<E> implements Iterable<E> {
    private HashMap<E, Object> map; // 使用 HashMap 存储

    /* 构造函数 */
    public HashSet(int capacity, float loadFactor) {
        map = new HashMap<E, Object>(capacity, loadFactor);
    }
    public HashSet(int capacity) {
        map = new HashMap<E, Object>(capacity);
    }
    public HashSet() {
        map = new HashMap<E, Object>();
    }

    /* 查询、清空 */
    public int size() {
        return map.size;
    }
    public int capacity() {
        return map.capacity;
    }
    public float loadFactor() {
        return map.loadFactor;
    }
    public boolean empty() {
        return map.empty();
    }
    public void clear() {
        map.clear();
    }

    // 添加、移除
    public void add(E elem) {
        map.put(elem, null);
    }
    public void remove(E elem) {
        map.remove(elem);
    }

    // 子集
    public boolean containsAll(HashSet<E> set) {
        HashMap.Entry<E, Object> entry;
        for (int i = 0; i < set.map.capacity; i++) {
            for (entry = set.map.table[i]; entry != null; entry = entry.next) {
                if (!map.hasKey(entry.key)) {
                    return false;
                }
            }
        }
        return true;
    }

    // 并集
    public void addAll(HashSet<E> set) {
        HashMap.Entry<E, Object> entry;
        for (int i = 0; i < set.map.capacity; i++) {
            for (entry = set.map.table[i]; entry != null; entry = entry.next) {
                map.put(entry.key, null);
            }
        }
    }

    // 交集
    public void retainAll(HashSet<E> set) {
        HashMap<E, Object> newMap = new HashMap<>(map.capacity, map.loadFactor);
        HashMap.Entry<E, Object> entry;
        for (int i = 0; i < map.capacity; i++) {
            for (entry = map.table[i]; entry != null; entry = entry.next) {
                if (set.map.hasKey(entry.key)) {
                    newMap.put(entry.key, null);
                }
            }
        }
        map = newMap;
    }

    // 补集
    public void removeAll(HashSet<E> set) {
        HashMap.Entry<E, Object> entry;
        for (int i = 0; i < set.map.capacity; i++) {
            for (entry = set.map.table[i]; entry != null; entry = entry.next) {
                map.remove(entry.key);
            }
        }
    }

    // 判断是否存在指定元素
    public boolean has(E elem) {
        return map.hasKey(elem);
    }

    // 获取一个包含所有元素的数组
    public E[] values() {
        return map.keys();
    }

    @Override
    public String toString() {
        if (map.empty()) {
            return "(null)";
        }
        StringBuilder sb = new StringBuilder("{ ");
        for (int i = 0; i < map.capacity; i++) {
            for (HashMap.Entry<E, Object> entry = map.table[i]; entry != null; entry = entry.next) {
                sb.append(entry.key + ", ");
            }
        }
        sb.append("\b\b }");
        return sb.toString();
    }

    @Override
    public Iterator<E> iterator() {
        return new Itr();
    }

    private class Itr implements Iterator<E> {
        private E[] array = values();
        private int cursor;

        @Override
        public boolean hasNext() {
            return cursor < array.length;
        }

        @Override
        public E next() {
            return array[cursor++];
        }
    }
}
</script></code></pre>



**LinkHashMap 类**
"哈希表" + "双向链表"，可保存 Entry 的插入顺序，同时可按照 Entry 的访问顺序进行动态排列，最新访问的记录排在链表的头部。
与 HashMap 相比，它的 Entry 增加了两个新的指针 before、after，分别指向前一个、后一个 Entry 记录。
<pre><code class="language-java line-numbers"><script type="text/plain">package com.zfl9.collection;

import java.util.Arrays;
import java.util.Iterator;
import java.lang.reflect.Array;

public class LinkHashMap<K, V> implements Iterable<LinkHashMap.Entry<K, V>> {
    // Entry 记录
    public static class Entry<K, V> {
        int hash;
        K key;
        V value;
        Entry<K, V> next;
        Entry<K, V> before; // 指向前一个节点的指针
        Entry<K, V> after; // 指向后一个节点的指针

        Entry(int hash, K key, V value, Entry<K, V> next, Entry<K, V> before, Entry<K, V> after) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
            this.before = before;
            this.after = after;
        }
        Entry(int hash, K key, V value, Entry<K, V> next) {
            this(hash, key, value, next, null, null);
        }

        /* getter、setter */
        public K getKey() { return key; }
        public V getValue() { return value; }
        public V setValue(V value) {
            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }
    }

    Entry<K, V>[] table;
    int capacity;
    static final int DEFAULT_CAPACITY = 32;
    float loadFactor;
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    int size;
    final Entry<K, V> HEAD = new Entry<K, V>(0, null, null, null, null, null); // 头结点
    final Entry<K, V> TAIL = new Entry<K, V>(0, null, null, null, null, null); // 尾结点
    boolean accessOrder; // 是否按照访问顺序进行动态排列

    /* 构造函数 */
    public LinkHashMap(int capacity, float loadFactor, boolean accessOrder) {
        if (capacity % 2 !=0 || capacity <= 0) {
            throw new IllegalArgumentException("illegal capacity: " + capacity);
        }
        if (loadFactor <= 0) {
            throw new IllegalArgumentException("illegal loadFactor: " + loadFactor);
        }
        this.capacity = capacity;
        this.loadFactor = loadFactor;
        this.accessOrder = accessOrder;
        @SuppressWarnings({"rawtypes", "unchecked"})
        Entry<K, V>[] table = (Entry<K, V>[]) new Entry[capacity];
        this.table = table;
        HEAD.after = TAIL;
        TAIL.before = HEAD;
    }
    public LinkHashMap(int capacity, float loadFactor) {
        this(capacity, loadFactor, false);
    }
    public LinkHashMap(int capacity, boolean accessOrder) {
        this(capacity, DEFAULT_LOAD_FACTOR, accessOrder);
    }
    public LinkHashMap(int capacity) {
        this(capacity, DEFAULT_LOAD_FACTOR, false);
    }
    public LinkHashMap(boolean accessOrder) {
        this(DEFAULT_CAPACITY, DEFAULT_LOAD_FACTOR, accessOrder);
    }
    public LinkHashMap() {
        this(DEFAULT_CAPACITY, DEFAULT_LOAD_FACTOR, false);
    }

    /* 查询、清空 */
    public int size() {
        return size;
    }
    public int capacity() {
        return capacity;
    }
    public float loadFactor() {
        return loadFactor;
    }
    public boolean accessOrder() {
        return accessOrder;
    }
    public boolean empty() {
        return size == 0;
    }
    public void clear() {
        Entry<K, V> temp;
        for (int i = 0; i < capacity; i++) {
            while (table[i] != null) {
                temp = table[i].next;
                table[i].hash = 0;
                table[i].key = null;
                table[i].value = null;
                table[i].next = null;
                table[i].before = null;
                table[i].after = null;
                table[i] = temp;
            }
        }
        HEAD.after = TAIL;
        TAIL.before = HEAD;
        size = 0;
    }

    /* hash 相关 */
    int hash(K key) {
        int hash;
        return key == null ? 0 : (hash = key.hashCode()) ^ (hash >>> 16);
    }
    int indexFor(int hash) {
        return hash & (capacity - 1);
    }

    /* 自动扩容 */
    private void autoGrow(int minSize) {
        if (minSize > capacity * loadFactor) {
            table = Arrays.copyOf(table, capacity * 2);
            capacity = table.length;
            Entry<K, V> head = new Entry<K, V>(0, null, null, null), entry, temp;
            int index;
            for (int i = 0; i < capacity / 2; i++) {
                head.next = table[i];
                for (entry = head; entry.next != null;) {
                    if ((index = indexFor(entry.next.hash)) != i) {
                        temp = entry.next;
                        entry.next = entry.next.next;
                        temp.next = table[index];
                        table[index] = temp;
                    } else {
                        entry = entry.next;
                    }
                }
                table[i] = head.next;
            }
        }
    }

    /* put、get、remove */
    public V put(K key, V value) {
        int hash = hash(key);
        int index = indexFor(hash);
        for (Entry<K, V> entry = table[index]; entry != null; entry = entry.next) {
            if (entry.hash == hash) {
                V oldValue = entry.value;
                entry.value = value;
                return oldValue;
            }
        }
        autoGrow(size + 1);
        index = indexFor(hash);
        table[index] = new Entry<K, V>(hash, key, value, table[index], TAIL.before, TAIL);
        TAIL.before = table[index];
        table[index].before.after = table[index];
        size++;
        return null;
    }
    public V get(K key) {
        int hash = hash(key);
        int index = indexFor(hash);
        for (Entry<K, V> entry = table[index]; entry != null; entry = entry.next) {
            if (entry.hash == hash) {
                if (accessOrder) {
                    entry.before.after = entry.after;
                    entry.after.before = entry.before;
                    entry.after = HEAD.after;
                    entry.after.before = entry;
                    HEAD.after = entry;
                    entry.before = HEAD;
                }
                return entry.value;
            }
        }
        return null;
    }
    public V remove(K key) {
        int hash = hash(key);
        int index = indexFor(hash);
        if (table[index] == null) {
            return null;
        }
        Entry<K, V> head = new Entry<K, V>(0, null, null, table[index]), entry, temp;
        for (entry = head; entry.next != null; entry = entry.next) {
            if (entry.next.hash == hash) {
                temp = entry.next.next;
                entry.next.before.after = entry.next.after;
                entry.next.after.before = entry.next.before;
                V value = entry.next.value;
                entry.next.hash = 0;
                entry.next.key = null;
                entry.next.value = null;
                entry.next.next = null;
                entry.next.before = null;
                entry.next.after = null;
                entry.next = temp;
                table[index] = head.next;
                size--;
                return value;
            }
        }
        return null;
    }

    // 是否包含指定 key、value
    public boolean hasKey(K key) {
        int hash = hash(key);
        int index = indexFor(hash);
        for (Entry<K, V> entry = table[index]; entry != null; entry = entry.next) {
            if (entry.hash == hash) {
                return true;
            }
        }
        return false;
    }
    public boolean hasValue(V value) {
        for (Entry<K, V> entry = HEAD.after; entry != TAIL; entry = entry.after) {
            if (entry.value.equals(value)) {
                return true;
            }
        }
        return false;
    }

    // 获取 key、value 的数组
    public K[] keys() {
        if (size == 0) {
            return null;
        }
        @SuppressWarnings("unchecked")
        K[] array = (K[]) Array.newInstance(HEAD.after.key.getClass(), size);
        int index = 0;
        for (Entry<K, V> entry = HEAD.after; entry != TAIL; entry = entry.after) {
            array[index++] = entry.key;
        }
        return array;
    }
    public V[] values() {
        if (size == 0) {
            return null;
        }
        @SuppressWarnings("unchecked")
        V[] array = (V[]) Array.newInstance(HEAD.after.value.getClass(), size);
        int index = 0;
        for (Entry<K, V> entry = HEAD.after; entry != TAIL; entry = entry.after) {
            array[index++] = entry.value;
        }
        return array;
    }

    @Override
    public String toString() {
        if (size == 0) {
            return "(null)";
        }
        StringBuilder sb = new StringBuilder("{ ");
        for (Entry<K, V> entry = HEAD.after; entry != TAIL; entry = entry.after) {
            sb.append("{" + entry.key + ", " + entry.value + "}, ");
        }
        sb.append("\b\b }");
        return sb.toString();
    }
    public String toDescString() {
        if (size == 0) {
            return "(null)";
        }
        StringBuilder sb = new StringBuilder("{ ");
        for (Entry<K, V> entry = TAIL.before; entry != HEAD; entry = entry.before) {
            sb.append("{" + entry.key + ", " + entry.value + "}, ");
        }
        sb.append("\b\b }");
        return sb.toString();
    }

    @Override
    public Iterator<LinkHashMap.Entry<K, V>> iterator() {
        return new Itr(true);
    }
    public Iterator<LinkHashMap.Entry<K, V>> descIterator() {
        return new Itr(false);
    }

    private class Itr implements Iterator<LinkHashMap.Entry<K, V>> {
        private Entry<K, V> entry;
        private boolean ascending;

        private Itr(boolean ascending) {
            this.ascending = ascending;
            entry = ascending ? HEAD : TAIL;
        }

        @Override
        public boolean hasNext() {
            return ascending ? entry.after != TAIL : entry.before != HEAD;
        }

        @Override
        public LinkHashMap.Entry<K, V> next() {
            return entry = ascending ? entry.after : entry.before;
        }
    }
}
</script></code></pre>



**LinkHashSet 包装类**
<pre><code class="language-java line-numbers"><script type="text/plain">package com.zfl9.collection;

import java.util.Iterator;

public class LinkHashSet<E> implements Iterable<E> {
    private LinkHashMap<E, Object> map; // 使用 LinkHashMap 存储

    /* 构造函数 */
    public LinkHashSet(int capacity, float loadFactor, boolean accessOrder) {
        map = new LinkHashMap<E, Object>(capacity, loadFactor, accessOrder);
    }
    public LinkHashSet(int capacity, float loadFactor) {
        map = new LinkHashMap<E, Object>(capacity, loadFactor);
    }
    public LinkHashSet(int capacity, boolean accessOrder) {
        map = new LinkHashMap<E, Object>(capacity, accessOrder);
    }
    public LinkHashSet(int capacity) {
        map = new LinkHashMap<E, Object>(capacity);
    }
    public LinkHashSet(boolean accessOrder) {
        map = new LinkHashMap<E, Object>(accessOrder);
    }
    public LinkHashSet() {
        map = new LinkHashMap<E, Object>();
    }

    /* 查询、清空 */
    public int size() {
        return map.size;
    }
    public int capacity() {
        return map.capacity;
    }
    public float loadFactor() {
        return map.loadFactor;
    }
    public boolean accessOrder() {
        return map.accessOrder;
    }
    public boolean empty() {
        return map.empty();
    }
    public void clear() {
        map.clear();
    }

    /* 添加、移除 */
    public void add(E elem) {
        map.put(elem, null);
    }
    public void remove(E elem) {
        map.remove(elem);
    }

    // 子集
    public boolean containsAll(LinkHashSet<E> set) {
        for (LinkHashMap.Entry<E, Object> entry = set.map.HEAD.after; entry != set.map.TAIL; entry = entry.after) {
            if (!map.hasKey(entry.key)) {
                return false;
            }
        }
        return true;
    }

    // 并集
    public void addAll(LinkHashSet<E> set) {
        for (LinkHashMap.Entry<E, Object> entry = set.map.HEAD.after; entry != set.map.TAIL; entry = entry.after) {
            map.put(entry.key, null);
        }
    }

    // 交集
    public void retainAll(LinkHashSet<E> set) {
        LinkHashMap<E, Object> newMap = new LinkHashMap<>(map.capacity, map.loadFactor, map.accessOrder);
        for (LinkHashMap.Entry<E, Object> entry = map.HEAD.after; entry != map.TAIL; entry = entry.after) {
            if (set.map.hasKey(entry.key)) {
                newMap.put(entry.key, null);
            }
        }
        map = newMap;
    }

    // 补集
    public void removeAll(LinkHashSet<E> set) {
        for (LinkHashMap.Entry<E, Object> entry = set.map.HEAD.after; entry != set.map.TAIL; entry = entry.after) {
            map.remove(entry.key);
        }
    }

    // 是否包含指定元素
    public boolean has(E elem) {
        return map.hasKey(elem);
    }

    // 获取包含所有元素的数组
    public E[] values() {
        return map.keys();
    }

    @Override
    public String toString() {
        if (map.size == 0) {
            return "(null)";
        }
        StringBuilder sb = new StringBuilder("{ ");
        for (LinkHashMap.Entry<E, Object> entry = map.HEAD.after; entry != map.TAIL; entry = entry.after) {
            sb.append(entry.key + ", ");
        }
        sb.append("\b\b }");
        return sb.toString();
    }
    public String toDescString() {
        if (map.size == 0) {
            return "(null)";
        }
        StringBuilder sb = new StringBuilder("{ ");
        for (LinkHashMap.Entry<E, Object> entry = map.TAIL.before; entry != map.HEAD; entry = entry.before) {
            sb.append(entry.key + ", ");
        }
        sb.append("\b\b }");
        return sb.toString();
    }

    @Override
    public Iterator<E> iterator() {
        return new Itr(true);
    }
    public Iterator<E> descIterator() {
        return new Itr(false);
    }

    private class Itr implements Iterator<E> {
        private boolean ascending;
        private LinkHashMap.Entry<E, Object> entry;

        private Itr(boolean ascending) {
            this.ascending = ascending;
            entry = ascending ? map.HEAD : map.TAIL;
        }

        @Override
        public boolean hasNext() {
            return ascending ? entry.after != map.TAIL : entry.before != map.HEAD;
        }

        @Override
        public E next() {
            return (entry = ascending ? entry.after : entry.before).key;
        }
    }
}
</script></code></pre>
