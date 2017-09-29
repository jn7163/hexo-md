---
title: Java7 Java8 新特性
date: 2017-09-28 13:00:00
categories:
- java
tags:
- java
keywords: Java7 Java8 新特性
---

> 
Java7 Java8 新特性

<!-- more -->

## Java7 新特性
**数字字面量支持二进制表示**
二进制使用前缀`0b`或`0B`，如：`0b110010`；
八进制使用前缀`0`，如：`034`；
十进制没有前缀，就是普通的字面量，如：`231`；
十六进制使用前缀`0x`或`0X`，如：`0x2A7E`；

**数字中可以添加`_`分隔符**
如果一个数值比较大，很长，那么可以使用`_`来分割，比如：`1_000_000_000`，表示`1,000,000,000`；
只能在数字中间添加`_`符号，不能在边缘添加及小数点前后添加，在编译过程中，`_`将被编译器删除。

**`switch...case`中可以使用 String 字符串**
<pre><code class="language-java"><script type="text/plain">switch (gender) {
    case "男":
        title = name + " 先生";
        break;
    case "女":
        title = name + " 女士";
        break;
    default:
        title = name;
}
</script></code></pre>



编译器在编译时先做处理： 
1) 只有一个 case，直接转成 if；
2) 只有一个 case 和 default，直接转换为 if...else；
3) 有多个 case，先将 String 转换为 hashCode，然后进行相应的处理，JavaCode 在底层兼容 Java7 以前版本。

**一个 catch 可同时捕获多个异常**
> 
如果一个`catch`语句中捕获了多个异常，那么变量`e`将自动变为`final`常量。

多个异常之间使用按位或`|`操作符连接，如：
<pre><code class="language-java"><script type="text/plain">try {
    // TODO
} catch (ExceptionType1 | ExceptionType2 e) {
    // TODO
}
</script></code></pre>



**泛型的类型推断**
在 Java7 之前，我们通常需要这么写：`ArrayList<String> list = new ArrayList<String>();`
在 Java7 之后，我们可以这么写：`ArrayList<String> list = new ArrayList<>();`

**`@SafeVarargs`注解**
`@SafeVarargs`在 JDK7 中引入，主要目的是处理可变长参数中的泛型，此注解告诉编译器：在可变长参数中的泛型是类型安全的。

可变长参数是使用数组存储的，而数组和泛型不能很好的混合使用；

简单的说，数组元素的数据类型在编译和运行时都是确定的，而泛型的数据类型只有在运行时才能确定下来，因此当把一个泛型存储到数组中时，编译器在编译阶段无法检查数据类型是否匹配，因此会给出警告信息："[unchecked] Possible heap pollution from parameterized vararg type T"；

意思就是：存在可能的堆污染(heap pollution)，即如果泛型的真实数据类型无法和参数数组的类型匹配，会导致 ClassCastException 异常，因此当在可变长参数中使用泛型时，编译器都会给出警告信息。

忽略这些异常的方法有：
1) 编译器选项：`javac -Xlint:unchecked`
2) 使用注解：`@SuppressWarnings("unchecked")`
3) 使用注解：`@SafeVarargs`（Java7 新增）

**`try-with-resources`释放资源**
通过 IO 流读取文件的通常操作：
<pre><code class="language-java line-numbers"><script type="text/plain">FileInputStream f = null;
try {
    // 可能抛出异常 FileNotFoundException
    f = new FileInputStream(args.length == 0 ? "/etc/sysctl.d/default.conf" : args[0]);
    int ndata = 0;
    byte[] data = new byte[4096];
    while ((ndata = f.read(data)) != -1) { // read() 可能抛出异常 IOException
        System.out.write(data, 0, ndata);
    }
} catch (FileNotFoundException e) {
    throw new RuntimeException("文件不存在", e); // 抛给调用者
} catch (IOException e) {
    throw new RuntimeException("文件读取失败", e); // 抛给调用者
} finally {
    if (f != null) {
        try {
            f.close(); // 可能抛出异常 IOException
        } catch (IOException e) {
            System.err.println(e + "  文件关闭失败!");
        }
    }
}
</script></code></pre>



从代码中可以看出，如果 try 块中发生异常（文件不存在、文件读取错误），那么该异常将被传递给调用者；
如果 try 块和 finally 块中同时发生异常，那么 finally 块中的异常将被抑制，不会向外传播。

在 Java7 之前，我们只能用这样的复杂且重复的代码来处理这种情况，没有什么更好的办法了；
但是在 Java7 之后，我们可以使用`try-with-resources`语句简化上面这段代码：
<pre><code class="language-java line-numbers"><script type="text/plain">try (FileInputStream f = new FileInputStream(args.length == 0 ? "/etc/sysctl.d/default.conf" : args[0])) {
    int ndata = 0;
    byte[] data = new byte[4096];
    while ((ndata = f.read(data)) != -1) {
        System.out.write(data, 0, ndata);
    }
} catch (FileNotFoundException e) {
    throw new RuntimeException("文件不存在", e);
} catch (IOException e) {
    throw new RuntimeException("文件读取失败", e);
}
</script></code></pre>



这段代码的运行逻辑和之前的是一样的，在调用 f.close() 过程中如果发生异常，那么该异常将被抑制，只有 try 块中的异常才会往外传播。

但是有一点不同，之前的代码中，如果 f.close() 发生异常，那么该异常仅仅就是被打印到屏幕而已，并没有记录下来；
而在使用了`try-with-resources`的代码中，如果 f.close() 发生异常，那么它将被隐式的添加至 catch 块中 e 变量的"被抑制异常数组"，可以通过 Throwable.getSuppressed() 方法获得"被抑制异常数组"。

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        try (Test t = new Test()) {
            t.work();
        } catch (RuntimeException e) {
            System.err.println("--------- t.work() ---------");
            System.err.println(e);
            System.err.println("--------- t.close() ---------");
            System.err.println(e.getSuppressed()[0]);
        }
    }

    private static class Test implements AutoCloseable {
        public Test() {
            System.out.println("Test::constructor");
        }

        public void work() {
            System.out.println("Test running ...");
            throw new RuntimeException("Test.work() occurred RuntimeException");
        }

        @Override
        public void close() {
            System.out.println("Test::close()");
            throw new RuntimeException("Test.close() occurred RuntimeException");
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:37:17]
$ javac Main.java

# root @ arch in ~/work on git:master x [16:37:39]
$ java Main
Test::constructor
Test running ...
Test::close()
--------- t.work() ---------
java.lang.RuntimeException: Test.work() occurred RuntimeException
--------- t.close() ---------
java.lang.RuntimeException: Test.close() occurred RuntimeException
</script></code></pre>



**哪些资源可以使用`try-with-resources`管理**？
只要一个类实现了 java.lang.AutoCloseable 接口就可以使用`try-with-resources`进行资源的自动释放。

java.lang.AutoCloseable 接口只有一个方法：`void close() throws Exception;`；
同时，java.io.Closeable 接口因继承自 java.lang.AutoCloseable 接口，因此也可以使用`try-with-resources`释放资源。

try-with-resources 语句可以有 catch、finally 语句块，但是 try-with-resources 语句后面的 catch、finally 语句块将在资源关闭后执行。

try 块中可以注册多个资源，在离开 try 块时，它们将被按照相反的顺序调用 close() 方法。比如：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        try (Test t1 = new Test(1);
             Test t2 = new Test(2)) {
            t1.work();
            t2.work();
        }
    }

    private static class Test implements AutoCloseable {
        private int id;

        public Test(int id) {
            this.id = id;
            System.out.println(id + " -> open()");
        }

        public void work() {
            System.out.println(id + " -> running ...");
        }

        @Override
        public void close() {
            System.out.println(id + " -> close()");
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:50:17]
$ javac Main.java

# root @ arch in ~/work on git:master x [16:50:33]
$ java Main
1 -> open()
2 -> open()
1 -> running ...
2 -> running ...
2 -> close()
1 -> close()
</script></code></pre>



## Java8 新特性
Java8 是 Java 语言开发的一个主要版本。Oracle 公司于 2014 年 3 月 18 日发布 Java8 ，它支持**函数式编程**，新的 JavaScript 引擎，新的日期 API，新的 Stream API 等。

Java8 新增了非常多的特性，主要的几个如下：
- Lambda 表达式：对匿名内部类的一个改进，基本可以取代内部匿名类，并且性能比匿名类好得多；
- 方法引用：方法引用其实是随 Lambda 产生的语法糖，与 Lambda 联合使用，可以使语言的构造更紧凑简洁，减少冗余代码；
- 默认方法：默认方法就是一个在接口里面有了一个实现的方法，使用 default 关键字；
- Stream API：不同于 java.io 中的 Stream 流，新添加的 Stream API（java.util.stream）把真正的函数式编程风格引入到 Java 中；
- Date Time API：加强对日期与时间的处理；
- Optional 类：Optional 类已经成为 Java8 类库的一部分，用来解决空指针异常；



目前先介绍 Lambda 表达式、方法引用、默认方法。

**Lambda 表达式**
Lambda 表达式实质上是一个内部匿名类的对象；
对于匿名类来说，必须继承一个父类或者实现一个接口，在编译之后会生成一个名为`父类$n.class`（n 从 1 开始）的独立字节码文件；
对于 Lambda，也会实现一个接口，这个接口只有一个抽象方法（称为"函数式接口"），但是在编译之后并不会生成单独的字节码文件。

因此，Lambda 不能单独存在，它必定和一个函数式接口`@FunctionalInterface`相关；
函数式接口就是只有一个抽象方法的普通接口，为了明确语义，通常使用`@FunctionalInterface`注解一个函数式接口。

**内部类和 Lambda 表达式的命名规则**
1) 成员内部类，包括普通成员内部类、静态成员内部类，`外部类名$内部类名`
2) 局部内部类，`外部类名$n内部类名`，n 从 1 开始，每个函数都有不同的 n 值
3) 匿名内部类，`外部类名$n`，n 从 1 开始
4) Lambda 表达式，`外部类名$$Lambda$n`，n 从 1 开始

不同于 C/C++，在 Java 中，`$`美元符也是一个合法的标识符，可以位于标识符的任意位置。

**Lambda 的语法**
`(parameters) -> { statements }`
- `parameters`：函数的形参，即当前 Lambda 所实现的函数式接口中的函数的形参；
可以省略形参的数据类型，只写形参变量也是可以的，编译器可以自行推断；如`(int n)`与`(n)`是一样的；
当函数只有一个参数，并且没有显式指明形参类型，则可以省略小括号，如`n -> { ... }`；
- `statements`：函数体，和普通的函数体一样，可以有任意多条语句；
如果只有一条语句，那么可以省略花括号`{}`，且该语句的返回值类型必须和所实现函数的返回值类型相兼容；
如果有多条语句，那么就和普通的匿名类一样，需要有花括号`{}`，返回值类型也必须和所实现函数的返回值类型相兼容；



对比一下匿名类、Lambda 表达式：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        new Thread(() -> System.out.println("Lambda expression")).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Anonymous Class");
            }
        }).start();
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [11:28:14]
$ javac Main.java

# root @ arch in ~/work on git:master x [11:28:25]
$ java Main
Lambda expression
Anonymous Class
</script></code></pre>



Lambda 与匿名类的不同之处：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        new Main() {}; // 正确，new 一个普通的对象

        new Main() { // 正确，new 一个普通的对象并调用其方法
            public void func() {}
        }.func();

        Main m = new Main() {}; // 正确，向上转型

        // () -> {}; // 错误
        // () -> {}.func(); // 错误

        VoidToVoid v = () -> {}; // 正确，向上转型
        v.func(); // 正确，调用 func() 方法
    }
}

@FunctionalInterface
interface VoidToVoid {
    void func();
}
</script></code></pre>



**Java 中有哪些 FunctionalInterface 接口**？
JDK1.8 之前已有的函数式接口：
`java.lang.Runnable`
`java.util.concurrent.Callable`
`java.security.PrivilegedAction`
`java.util.Comparator`
`java.io.FileFilter`
`java.nio.file.PathMatcher`
`java.lang.reflect.InvocationHandler`
`java.beans.PropertyChangeListener`
`java.awt.event.ActionListener`
`javax.swing.event.ChangeListener`

JDK1.8 新增加的函数接口：
`java.util.function.*`

java.util.function 它包含了很多函数式接口，用来支持 Java 的函数式编程，该包中的函数式接口有：
`Supplier<T>`：get()，不接受参数，返回 T 类型结果；
`BooleanSupplier`：getAsBoolean()，不接受参数，返回 boolean 结果；
`IntSupplier`：getAsInt()，不接受参数，返回 int 结果；
`LongSupplier`：getAsLong()，不接受参数，返回 long 结果；
`DoubleSupplier`：getAsDouble()，不接受参数，返回 double 结果；

`Consumer<T>`：accept()，接受 1 个参数，不返回结果；
`BiConsumer<T, U>`：accept()，接受 2 个参数，不返回结果；
`IntConsumer`：accept()，接受 1 个 int 参数，不返回结果；
`LongConsumer`：accept()，接受 1 个 long 参数，不返回结果；
`DoubleConsumer`：accept()，接受 1 个 double 参数，不返回结果；
`ObjIntConsumer<T>`：accept()，接受 1 个 T 类型参数和 1 个 int 参数，不返回结果；
`ObjLongConsumer<T>`：accept()，接受 1 个 T 类型参数和 1 个 long 参数，不返回结果；
`ObjDoubleConsumer<T>`：accept()，接受 1 个 T 类型参数和 1 个 double 参数，不返回结果；

`Predicate<T>`：test()，接受 1 个参数，返回 boolean 结果；
`BiPredicate<T, U>`：test()，接受 2 个参数，返回 boolean 结果；
`IntPredicate`：test()，接受 1 个 int 参数，返回 boolean 结果；
`LongPredicate`：test()，接受 1 个 long 参数，返回 boolean 结果；
`DoublePredicate`：test()，接受 1 个 double 参数，返回 boolean 结果；

`UnaryOperator<T>`：apply()，接受 1 个 T 类型参数，返回 T 类型结果；
`IntUnaryOperator`：applyAsInt()，接受 1 个 int 参数，返回 int 结果；
`LongUnaryOperator`：applyAsLong()，接受 1 个 long 参数，返回 long 结果；
`DoubleUnaryOperator`：applyAsDouble()，接受 1 个 double 参数，返回 double 结果；

`BinaryOperator<T>`：apply()，接受 2 个 T 类型参数，返回 T 类型结果；
`IntBinaryOperator`：applyAsInt()，接受 2 个 int 参数，返回 int 结果；
`LongBinaryOperator`：applyAsLong()，接受 2 个 long 参数，返回 long 结果；
`DoubleBinaryOperator`：applyAsDouble()，接受 2 个 double 参数，返回 double 结果；

`Function<T, R>`：apply()，接受 1 个 T 类型参数，返回 R 类型结果；
`BiFunction<T, U, R>`：apply()，接受 2 个参数，返回 1 个结果；
`IntFunction<R>`：apply()，接受 1 个 int 参数，返回 R 类型结果；
`IntToLongFunction`：applyAsLong()，接受 1 个 int 参数，返回 long 结果；
`IntToDoubleFunction`：applyAsDouble()，接受 1 个 int 参数，返回 double 结果；
`LongFunction<R>`：apply()，接受 1 个 long 参数，返回 R 类型结果；
`LongToIntFunction`：applyAsInt()，接受 1 个 long 参数，返回 int 结果；
`LongToDoubleFunction`：applyAsDouble()，接受 1 个 long 参数，返回 double 结果；
`DoubleFunction<R>`：apply()，接受 1 个 double 参数，返回指定类型的结果；
`DoubleToIntFunction`：applyAsInt()，接受 1 个 double 参数，返回 int 结果；
`DoubleToLongFunction`：applyAsLong()，接受 1 个 double 参数，返回 long 结果；
`ToIntFunction<T>`：applyAsInt()，接受 1 个 T 类型参数，返回 int 结果；
`ToIntBiFunction<T, U>`：applyAsInt()，接受 1 个 T 类型参数和 1 个 U 类型参数，返回 int 结果；
`ToLongFunction<T>`：applyAsLong()，接受 1 个 T 类型参数，返回 long 结果；
`ToLongBiFunction<T, U>`：applyAsLong()，接受 1 个 T 类型参数和 1 个 U 类型参数，返回 long 结果；
`ToDoubleFunction<T>`：applyAsDouble()，接受 1 个 T 类型参数，返回 double 结果；
`ToDoubleBiFunction<T, U>`：applyAsDouble()，接受 1 个 T 类型参数和 1 个 U 类型参数，返回 double 结果；


例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.List;
import java.util.Arrays;
import java.util.function.Predicate;

public class Main {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        System.out.println(list);

        System.out.println("-------- 奇数 --------");
        print(list, n -> n % 2 != 0);

        System.out.println("-------- 偶数 --------");
        print(list, n -> n % 2 == 0);

        System.out.println("-------- 大于3 --------");
        print(list, n -> n > 3);
    }

    private static <T> void print(List<T> list, Predicate<T> p) {
        for (T elem : list) {
            if (p.test(elem)) {
                System.out.print(elem + ", ");
            }
        }
        System.out.println();
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:12:13]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:12:31]
$ java Main
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
-------- 奇数 --------
1, 3, 5, 7, 9,
-------- 偶数 --------
2, 4, 6, 8, 10,
-------- 大于3 --------
4, 5, 6, 7, 8, 9, 10,
</script></code></pre>



**方法引用**
方法引用其实就是 Lambda 表达式的一种简写方式。

在 Java8 中，我们会使用 Lambda 表达式创建匿名方法，但是有时候，我们的 Lambda 表达式可能仅仅调用一个已存在的方法，而不做任何其它事，对于这种情况，通过一个方法名字来引用这个已存在的方法会更加清晰，Java8 的方法引用允许我们这样做。

方法引用是一个更加紧凑，易读的 Lambda 表达式，方法引用的操作符是双冒号"::"。

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.util.Arrays;
import java.util.ArrayList;
import java.util.function.Consumer;

public class Main {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));

        // 使用匿名类
        list.forEach(new Consumer<Integer>() {
            @Override
            public void accept(Integer elem) {
                System.out.println(elem);
            }
        });

        // 使用 Lambda
        System.out.println("------------");
        list.forEach(elem -> System.out.println(elem));

        // 使用方法引用
        System.out.println("------------");
        list.forEach(System.out::println);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:33:41]
$ javac Main.java

# root @ arch in ~/work on git:master x [14:33:51]
$ java Main
1
2
3
4
5
------------
1
2
3
4
5
------------
1
2
3
4
5
</script></code></pre>



**四种方法引用类型**
1) **引用静态方法**：`ClassName::staticMethodName`；
2) **引用实例方法**：`objectName::methodName`；
3) **引用构造方法**：`ClassName::new`；
4) **引用任意方法**：`ClassName::methodName`

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        // 静态方法引用
        StaticRefer<String> f1 = Test::staticMethod;
        f1.func("www.zfl9.com");

        // 实例方法引用
        // InstanceRefer<String> f2 = new Test()::instanceMethod;
        Test t = new Test();
        InstanceRefer<String> f2 = t::instanceMethod;
        f2.func("www.zfl9.com");

        // 构造方法引用，带参数
        ConstructorReferWithParam<String, Test> f3 = Test::new;
        f3.func("www.zfl9.com");

        // 构造方法引用，不带参数
        ConstructorRefer<Test> f4 = Test::new;
        f4.func();

        // 使用类名引用实例方法
        SpecialRefer<Test, String> f5 = Test::instanceMethod;
        f5.func(new Test(), "www.zfl9.com");
    }

    private static class Test {
        public Test() {}
        public Test(String s) {
            System.out.println(s);
        }

        public static void staticMethod(String s) {
            System.out.println(s);
        }

        public void instanceMethod(String s) {
            System.out.println(s);
        }
    }
}

@FunctionalInterface
interface ConstructorRefer<T> {
    T func();
}

@FunctionalInterface
interface ConstructorReferWithParam<T, R> {
    R func(T t);
}

@FunctionalInterface
interface StaticRefer<T> {
    void func(T t);
}

@FunctionalInterface
interface InstanceRefer<T> {
    void func(T t);
}

@FunctionalInterface
interface SpecialRefer<T, U> {
    void func(T t, U u);
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:25:54]
$ javac Main.java

# root @ arch in ~/work on git:master x [15:26:08]
$ java Main
www.zfl9.com
www.zfl9.com
www.zfl9.com
www.zfl9.com
</script></code></pre>



简要说明：
1) 实例方法引用：this 指针由编译器隐式传递；
2) 静态方法引用：没有 this 指针，静态方法是与实例无关的方法；
3) 构造方法引用：需要指定类名、构造函数参数；
4) 任意方法引用：引用静态方法没有区别，引用实例方法需要显式传入 this 指针；

**什么时候适合使用方法引用**？
Lambda 表达式的目的仅仅为了调用另一个已存在方法时，适合使用方法引用。

**什么时候不适合使用方法引用**？
当我们需要向 Lambda 表达式传入其它参数时，不适合使用方法引用。

**接口默认方法、静态方法**
Java8 对接口做了进一步的增强：
1、接口中可以定义 default 关键字修饰的非抽象方法，即：默认方法（或扩展方法）；
2、接口中可以定义 static 静态方法。

Java8 允许给接口添加一个非抽象的方法实现，只需要使用 default 关键字即可，这个特征又叫做**扩展方法**（也称为**默认方法**或**虚拟扩展方法**或**防护方法**）

在实现该接口时，该默认扩展方法在子类上可以直接使用，它的使用方式类似于抽象类中非抽象成员方法。
但是请注意，接口中的默认方法不能够重载 Object 中的定义的方法。eg：toString、equals、hashCode；

默认方法允许我们在接口里添加新的方法，而不会破坏实现这个接口的已有类的兼容性，也就是说不会强迫实现接口的类实现默认方法。
默认方法和抽象方法的区别是抽象方法必须要被实现，默认方法不是。作为替代方式，接口可以提供一个默认的方法实现，所有这个接口的实现类都会通过继承得到这个方法（如果有需要也可以重写这个方法）；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        new B().func();
        new C().func();
    }
}

interface A {
    default void func() {
        System.out.println("default method");
    }
}

class B implements A {} // 不重写默认方法

class C implements A { // 重写默认方法
    @Override
    public void func() {
        System.out.println("override method");
    }
}
</script></code></pre>



虽然默认方法很强大，但是使用之前一定要仔细考虑是不是真的需要使用默认方法；
因为在层级很复杂的情况下很容易导致逻辑模糊不清甚至产生编译错误！

**引入默认方法带来的优缺点**
优点：默认方法可以在不破坏代码的前提下扩展原有库的功能，避免了代码冗余；
缺点：从另一个方面来说，这使得接口作为协议，类作为具体实现的界限开始变得有点模糊。

引入默认方法之后，感觉接口和抽象类的界限变得有点模糊，接口和抽象类之间有很多相似的地方：
1、都是抽象类型；
2、都不能被实例化；
3、都可以提供具体的方法实现；
4、都可以不需要子类去实现所有方法；

但是接口和抽象类也有一些不同的地方：
1、抽象类不可以多继承，接口可以多继承；
2、设计理念不同，抽象类表示的是"is-a"关系，接口表示的是"like-a"关系；
3、接口中的变量默认被 public static final 修饰，不能在实现类中修改，而抽象类中的变量则没有此限制；

**默认方法带来的多继承命名冲突**
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        new C().func();
        new D().func();
        new E().func();
    }
}

interface A {
    default void func() {
        System.out.println("A::func()");
    }
}

interface B {
    default void func() {
        System.out.println("B::func()");
    }
}

class C implements A, B {
    public void func() { // 优先级最高，不产生命名冲突
        System.out.println("C::func()");
    }
}

class D implements A, B {
    public void func() {
        A.super.func(); // 指明调用 A.func()
    }
}

class E implements A, B {
    public void func() {
        B.super.func(); // 指明调用 B.func()
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:31:50]
$ javac Main.java

# root @ arch in ~/work on git:master x [17:32:04]
$ java Main
C::func()
A::func()
B::func()
</script></code></pre>



接口中的默认方法也可以被子接口重写，如：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        new C().func(); // B.func()
    }
}

interface A {
    default void func() {
        System.out.println("A::func()");
    }
}

interface B extends A {
    @Override
    default void func() {
        System.out.println("B::func()");
    }
}

class C implements A, B {}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:36:47] C:130
$ javac Main.java

# root @ arch in ~/work on git:master x [17:36:49]
$ java Main
B::func()
</script></code></pre>



**接口的静态方法**
在 Java8 中，接口除了可以有默认方法，还可以定义静态方法，和普通类的静态方法一样。
接口静态方法需要通过`接口名.方法名`方式来调用，否则发生编译错误。
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        A a = new B();
        // a.func(); 错误
        A.func();
    }
}

interface A {
    static void func() {
        System.out.println("A::func()");
    }
}

class B implements A {}
</script></code></pre>



接口中的静态方法不能被实现类、子接口所继承，比如：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        A.func(); // 正确
        // B.func(); // 错误
        // C.func(); // 错误
        // new C().func(); // 错误

        X.func(); // 正确
        Y.func(); // 正确
        new X().func(); // 警告
        new Y().func(); // 警告
    }
}

interface A {
    static void func() {
        System.out.println("interface A");
    }
}

interface B extends A {} // 不能继承

class C implements A {} // 不能继承

class X {
    public static void func() {
        System.out.println("class X");
    }
}

class Y extends X {} // 可以继承
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [18:04:31]
$ javac Main.java
Main.java:10: warning: [static] static method should be qualified by type name, X, instead of by an expression
        new X().func(); // 警告
               ^
Main.java:11: warning: [static] static method should be qualified by type name, X, instead of by an expression
        new Y().func(); // 警告
               ^
2 warnings

# root @ arch in ~/work on git:master x [18:04:47]
$ java Main
interface A
class X
class X
class X
class X
</script></code></pre>
