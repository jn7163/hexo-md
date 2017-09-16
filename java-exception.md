---
title: Java 异常处理
date: 2017-09-10 17:22:00
categories:
- java
tags:
- java
keywords: Java 异常处理
---

> 
Java 异常处理

<!-- more -->

## 异常处理基础
Java 异常处理通过 5 个关键字控制：`try`、`catch`、`finally`、`throw`、`throws`；

基本用法：
<pre><code class="language-java line-numbers"><script type="text/plain">try {
    // 可能抛出异常的语句
} catch (ExceptionType_1 e) {
    // 处理异常的语句
} catch (ExceptionType_2 e) {
    // 处理异常的语句
} finally {
    // finally块中的代码总是被执行，不管有没有发生异常
}
</script></code></pre>



1) try 不可以单独出现，可以出现 try...finally 这样的组合，catch 语句可以有多个，finally 语句只能有一个；
2) ExceptionType_1、ExceptionType_2 应该匹配的异常类型，只有相匹配的类型才会被对应的 catch 捕获；
3) 如果 ExceptionType_1 和 ExceptionType_2 存在继承关系，那么需要将子类的 catch 放在前面，否则会被其父类一并捕获；
4) 异常控制的流程和 if...else 一样，如果在 try 中检测到异常，会按照从上到下的顺序依次匹配所有的 catch 块，如果符合，则执行匹配到的 catch 语句，并停止向下匹配，执行完 catch 之后跳转到 finally 语句（如果存在的话），之后执行 finally 之后的正常语句；
5) Java SE 7 之后，一个 catch 块可以匹配多个异常，使用`|`管道符号分割，如`catch (E1 | E2 e) { statement }`；
6) Java 中的异常都是 java.lang.Exception 类的派生类，它们的继承关系如下：
![Java Exception 继承体系](/images/java-exception.png)

异常类有两个主要的子类：IOException 类和 RuntimeException 类；

**Throwable 的常用方法**
1) `String getMessage()`：异常信息的简单描述；
2) `String toString()`：异常信息的详细描述 全类名+异常信息；
3) `void printStackTrace()`：打印异常信息，打印栈追踪信息；

**Throwable 有两个子类**
1) Error 类一般表示与虚拟机有关的问题，如系统崩溃、内存溢出、方法调用栈溢出、虚拟机错误等问题，对于出现这样的错误，仅靠程序本身是无法修复的，需要终止程序，修改代码；
2) Exception 类，表示的是程序可以处理的异常，如空指针异常、数组越界异常、没有元素异常、类型转换异常等等；

**Exception 异常的分类**：
1) 运行时异常(`RuntimeException`或者是其子类)，也称非检查异常，不需要在 throws 语句中抛出；
2) 编译时异常(除了运行时异常就是编译时异常)，也称检查异常，要么使用 try...catch 处理，要么在 throws 中抛出，否则编译不通过；

**如何处理异常**
1) 对于`运行时异常`，我们不要用`try...catch`来捕获处理；
而是在程序开发调试阶段，**尽量去避免这种异常**，一旦发现该异常，正确的做法就会改进程序设计的代码和实现方式，修改程序中的错误，从而避免这种异常；
捕获并处理运行时异常是好的解决办法，因为可以通过改进代码实现来避免该种异常的发生；
2) 对于`受检查异常`，没说的，老老实实去按照异常处理的方法去处理，要么用`try...catch`捕获并解决，要么用`throws`抛出！
3) 对于`Error（运行时错误）`，不需要在程序中做任何处理，出现问题后，应该在程序在外的地方找问题，然后解决；

**原则**
1、避免过大的 try 块，不要把不会出现异常的代码放到 try 块里面，尽量保持一个 try 块对应一个或多个异常；
2、细化异常的类型，不要不管什么类型的异常都写成 Excetpion、RuntimeException、Throwable；
3、catch 块尽量保持一个块捕获一类异常，不要忽略捕获的异常，捕获到后要么处理，要么转译，要么重新抛出新类型的异常；
4、不要把自己能处理的异常抛给别人；
5、不要用`try...catch`参与控制程序流程，异常控制的根本目的是处理程序的非正常情况；

**异常转型**
异常转型实际上就是捕获到异常后，将异常以新的类型的异常再抛出，这样做一般为了异常的信息更直观！

**捕获到异常的默认处理方式**
所有未被 try...catch 处理的异常都将被 Java 运行时系统的默认处理程序捕获；
默认处理程序显示一个描述异常的字符串，打印异常发生处的堆栈轨迹并且终止程序；

**自定义异常类型**
如果 Java 中的内置异常类型不能满足你的需要，那么你可以创建一个属于你自己的异常类型；
自定义异常类型可以继承于 Exception、RuntimeException 等基类；

**finally 的执行顺序**
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        try {
            out.printf("try statement block\n");
            return;
        } finally {
            out.printf("finally statement block\n");
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [7:12:54] C:127
$ javac Main.java

# root @ arch in ~/work on git:master x [7:12:56]
$ java Main
try statement block
finally statement block
</script></code></pre>


<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        try {
            out.printf("try statement block\n");
            throw new RuntimeException();
        } finally {
            out.printf("finally statement block\n");
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [7:13:46]
$ javac Main.java

# root @ arch in ~/work on git:master x [7:13:50]
$ java Main
try statement block
finally statement block
Exception in thread "main" java.lang.RuntimeException
	at Main.main(Main.java:7)
</script></code></pre>


## throw抛出异常
我们除了可以处理别人抛出的异常之外，还可以自己显式的抛出一个异常，语法：`throw exceptionObj`；
exceptionObj 必须是 Throwable 类及其派生类的实例，其他的类型都是不被允许的；

所有的 Java 内置异常类型有两个构造函数：一个没有参数，一个带有一个字符串参数；
当用到第二种形式时，参数指定描述异常的字符串；
如果对象用作 print() 或 println() 的参数时，该字符串被显示；
这同样可以通过调用 getMessage() 来实现；

抛出异常的例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        try {
            func();
        } catch (RuntimeException e) {
            e.printStackTrace();
        }
    }

    public static void func() {
        try {
            throw new RuntimeException("这是一个运行时异常");
        } catch (RuntimeException e) {
            out.printf("catch exception [func()]\n");
            throw e; // 重新抛出
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:10:43]
$ javac Main.java

# root @ arch in ~/work on git:master x [20:10:52]
$ java Main
catch exception [func()]
java.lang.RuntimeException: 这是一个运行时异常
	at Main.func(Main.java:14)
	at Main.main(Main.java:6)
</script></code></pre>



## throws异常声明
1) throws 表示方法可能会抛出异常的声明；
如果添加了 throws 子句，表示该方法即将抛出异常，异常的处理交由它的调用者，至于调用者任何处理则不是它的责任范围内的了；
所以如果一个方法会有异常发生时，但是又不想处理或者没有能力处理，就可以使用 throws 来抛出这个异常给调用者去处理；
2) `throws`语句位于方法声明的后面，如：`void func() throws ClassCastException { ... }`
3) `throws`可以声明多个异常，多个异常之间使用逗号隔开；
4) 检查异常必须在 throws 中声明，非检查异常可不必声明；
5) throws 语句的目的：明确告知调用者可能抛出哪些异常；JavaDoc 也可对该异常进行说明使 API 更加友善；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        try {
            func();
        } catch (MyException e) {
            e.printStackTrace();
        }
    }

    public static void func() throws MyException {
        throw new MyException("自定义错误");
    }
}

class MyException extends Exception {
    public MyException() {}
    public MyException(String msg) {
        super(msg);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:33:50]
$ javac Main.java

# root @ arch in ~/work on git:master x [20:34:07]
$ java Main
MyException: 自定义错误
	at Main.func(Main.java:13)
	at Main.main(Main.java:6)
</script></code></pre>



## 异常处理的性能影响
创建一个异常对象的时空开销比创建一个普通的对象要大的多；

因为创建一个异常对象往往需要收集一个`栈跟踪（stack track）`，这个栈跟踪用于描述异常是在何处创建的；构建这些栈跟踪时需要为运行时栈做一份快照，正是这一部分开销很大；

异常处理过程中的主要开销就是`创建异常`，抛出和捕获异常都没有太大的性能影响；

那么 try...catch 和 for/while 循环一起使用呢？比如下面两个例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Test {
    public static void main(String[] args) {
        assert(args.length != 0) : "请提供运行程序的必要参数！";
        try {
            for (int i = 0; i < args.length; i++) {
                out.printf("%d, ", Integer.parseInt(args[i]));
            }
            out.println();
        } catch (NumberFormatException e) {
            out.println("bad_format");
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:59:59]
$ javac Test.java

# root @ arch in ~/work on git:master x [10:00:02]
$ java -ea Test
Exception in thread "main" java.lang.AssertionError: 请提供运行程序的必要参数！
	at Test.main(Test.java:5)

# root @ arch in ~/work on git:master x [10:00:05] C:1
$ java -ea Test 1 2 3 4 5 a b c
1, 2, 3, 4, 5, bad_format
</script></code></pre>



<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Test {
    public static void main(String[] args) {
        assert(args.length != 0) : "请提供运行程序的必要参数！";
        for (int i = 0; i < args.length; i++) {
            try {
                out.printf("%d, ", Integer.parseInt(args[i]));
            } catch (NumberFormatException e) {
                out.printf("bad_format, ");
            }
        }
        out.println();
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:02:12]
$ javac Test.java

# root @ arch in ~/work on git:master x [10:02:16]
$ java -ea Test 1 2 3 4 5
1, 2, 3, 4, 5,

# root @ arch in ~/work on git:master x [10:02:23]
$ java -ea Test 1 2 3 4 5 a b c
1, 2, 3, 4, 5, bad_format, bad_format, bad_format,
</script></code></pre>



两者在功能上有很明显的区别：
前者：只要在 for 循环中有任何一个异常，循环就会终止；
后者：如果转换失败则打印"bad_format"，并不会直接终止循环；

那它们的性能区别呢？
性能无非就是看`空间消耗`，`时间消耗`，一开始的时候我想当然得觉得 try...catch 重复执行了这么多次肯定比只执行了一次跑得肯定慢，空间消耗肯定更大；但是实际上并不是，只能说明太年轻！

[Stack Overflow 讨论帖地址](http://stackoverflow.com/questions/141560/should-try-catch-go-inside-or-outside-a-loop)
讨论的结果是：**在没有抛出异常的情况下，性能完全没区别**；

[Java 异常处理的原理](http://www.javaworld.com/javaworld/jw-01-1997/jw-01-hood.html?page=1)

大致流程：
1、类会跟随一张`异常表（exception table）`，每一个 try...catch 都会在这个表里添加行记录，每一个记录都有 4 个信息（try...catch 的开始地址，结束地址，异常的处理起始位，异常类名称）；

2、当代码在运行时抛出了异常时，首先拿着抛出位置到异常表中查找是否可以被 catch，如果可以则跑到异常处理的起始位置开始处理，如果没有找到则原地 return，并且 copy 异常的引用给父调用方，接着看父调用的异常表，以此类推；

综合上面来看可以得出结论：
1、异常如果没发生，也就不会去查表，也就是说你写不写 try...catch 也就是有没有这个异常表的问题，如果没有发生异常，写 try...catch 对性能是没有消耗的，所以不会让程序跑得更慢；
2、try 的范围大小其实就是异常表中两个值（开始地址和结束地址）的差异而已，也是不会影响性能的；

当然 try...catch 绝对不是在什么地方都可以乱加的！要在适当的场合使用适当的特性！


## assert断言
**断言的概念**
断言的目的是为了标示与验证程序开发者预期的结果，当程序运行到断言的位置时，对应的断言应该为真；若断言不为真时，程序会中止运行，并给出错误消息；
断言可以在运行时从代码中完全删除，所以对代码的运行速度没有影响；

**断言的使用**
断言有两种方法：
1) 一种是`assert bool_expression`；
2) 一种是`assert bool_expression : "assert_msg"`；

如果布尔表达式的值为 true，则表示程序的运行符合开发者预期的结果，这时候有没有断言都不影响程序的运行；
如果布尔表达式的值为 false，则表示程序的运行存在问题，不符合预期，需要进行修改，断言失败将抛出 AssertionError 错误，AssertionError 继承自 Error 类，不应该被 catch 处理；

断言是 JDK1.4 引入的一个功能，因此你的 Java 环境中的版本必须高于 1.4；
并且，在使用 java 命令运行程序的时候是默认关闭断言的，需要使用选项`-ea`显式启用断言；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">public class Main {
    public static void main(String[] args) {
        assert args.length != 0 : "必须提供最少一个参数!";
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [21:01:03]
$ javac Main.java

# root @ arch in ~/work on git:master x [21:01:08]
$ java -ea Main
Exception in thread "main" java.lang.AssertionError: 必须提供最少一个参数!
	at Main.main(Main.java:3)

# root @ arch in ~/work on git:master x [21:01:13] C:1
$ java -ea Main 1 2 3
</script></code></pre>



**断言推荐的使用方法**
用于验证方法中的内部逻辑，包括：
1) 内在不变式
2) 控制流程不变式
3) 后置条件和类不变式
注意：不推荐用于公有方法内的前置条件的检查；

> 
断言中的布尔表达式不应该改变上下文环境，因为关闭断言后会导致程序运行出现错乱；
断言的主要用途是用于开发阶段的测试，尽量的发现可能的 bug，并进行修复；
不应该将断言用于函数入口的参数检查，并且在正式运行的时候需要关闭断言；

**运行时屏蔽断言**
运行时要允许断言，可以用`java –enableassertions`或`java -ea`；
运行时要屏蔽断言，可以用`java –disableassertions`或`java -da`；
