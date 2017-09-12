---
title: Java 多线程编程
date: 2017-09-11 18:57:00
categories:
- java
tags:
- java
keywords: Java 多线程编程
---

> 
Java 多线程编程

<!-- more -->

## 线程
`进程是程序执行时的一个实例，即它是程序已经执行到何种程度的数据结构的汇集`；从内核的观点看，进程的目的就是担当分配系统资源（CPU时间、内存等）的基本单位；

`线程是进程的一个执行流，是CPU调度和分派的基本单位，它是比进程更小的能独立运行的基本单位`；一个进程由几个线程组成（拥有很多相对独立的执行流的用户程序共享应用程序的大部分数据结构），线程与同属一个进程的其他的线程共享进程所拥有的全部资源；

`"进程——资源分配的最小单位，线程——程序执行的最小单位"`

进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响；
而线程只是一个进程中的不同执行路径线程有自己的堆栈和局部变量，但线程没有单独的地址空间，一个线程死掉就等于整个进程死掉；
所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些，但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程；

**Java 中的多线程**
和 C/C++ 不同，Java 内置支持多线程编程，不需要类似 pthread 这样的第三方库；
Java 运行系统在很多方面依赖于线程，所有的类库设计都考虑到多线程；

**线程的 5 种状态**
![Java 中线程的五种状态](/images/java-multi-thread.png)

1) `New`新建状态：
当程序使用 new 关键字创建了一个线程后，该线程就处于新建状态，此时线程还未启动；

2) `Runnable`就绪状态：
一个新创建的线程并不自动开始运行，要执行线程，必须调用线程的 start() 方法；
当线程对象调用 start() 方法即启动了线程，start() 方法创建线程运行的系统资源，并调度线程运行 run() 方法；当 start() 方法返回后，线程就处于就绪状态；

处于就绪状态的线程并不一定立即运行 run() 方法，线程还必须同其他线程竞争 CPU 时间，只有获得 CPU 时间才可以运行线程；
因为在单 CPU 的计算机系统中，不可能同时运行多个线程，一个时刻仅有一个线程处于运行状态；因此此时可能有多个线程处于就绪状态；
对多个处于就绪状态的线程是由 Java 运行时系统的线程调度程序来调度的；

3) `Running`运行状态：
当线程获得 CPU 时间后，它才进入运行状态，真正开始执行 run() 方法；

4) `Block`阻塞状态：
线程运行过程中，可能由于各种原因进入阻塞状态：
- 线程通过调用 sleep() 方法进入睡眠状态；
- 线程调用一个在 I/O 上被阻塞的操作，即该操作在输入输出操作完成之前不会返回到它的调用者；
- 线程试图得到一个锁，而该锁正被其他线程持有；
- 线程在等待某个触发条件；


> 
所谓阻塞状态是正在运行的线程没有运行结束，暂时让出 CPU，这时其他处于就绪状态的线程就可以获得 CPU 时间，进入运行状态；

5) `Dead`死亡状态：
有两个原因会导致线程死亡：
- run() 方法正常退出而自然死亡；
- 一个未捕获的异常终止了 run() 方法而使线程猝死；


为了确定线程在当前是否存活着（就是要么是可运行的，要么是被阻塞了），需要使用 isAlive() 方法，如果是可运行或被阻塞，这个方法返回 true；如果线程仍旧是 new 状态且不是可运行的，或者线程死亡了，则返回 false；

需要注意的是，如果试图对一个已经死亡的线程调用 start() 方法，线程死亡后将可能再次作为线程执行，系统会抛出 IllegalThreadStateException 异常；

**挂起、阻塞、睡眠**
`挂起`和`睡眠`是**主动**的，挂起恢复需要主动完成，睡眠恢复则是自动完成的，因为睡眠有一个睡眠时间，睡眠时间到则恢复到就绪态；
而`阻塞`是**被动**的，是在等待某种事件或者资源的表现，一旦获得所需资源或者事件信息就自动回到就绪态；

`睡眠`和`挂起`是两种**行为**，`阻塞`则是一种**状态**；

挂起：一般是主动的，由系统或程序发出，甚至于辅存中去；（不释放 CPU，可能释放内存，放在外存）；
阻塞：一般是被动的，在抢占资源中得不到资源，被动的挂起在内存，等待某种资源或信号量将他唤醒（释放 CPU，不释放内存）；

**线程优先级**
Java 中的线程优先级的范围是 1～10，默认的优先级是 5；“高优先级线程”会优先于“低优先级线程”执行；

JVM 线程调度程序是基于优先级的抢先调度机制；在大多数情况下，当前运行的线程优先级将大于或等于线程池中任何线程的优先级；但这仅仅是大多数情况；

> 
注意：当设计多线程应用程序的时候，一定不要依赖于线程的优先级；因为线程调度优先级操作是没有保障的，只能把线程优先级作用作为一种提高程序效率的方法，但是要保证程序不依赖这种操作；

当线程池中线程都具有相同的优先级，调度程序的 JVM 实现自由选择它喜欢的线程；这时候调度程序的操作有两种可能：一是选择一个线程运行，直到它阻塞或者运行完成为止；二是时间分片，为池内的每个线程提供均等的运行机会；

设置线程的优先级：线程默认的优先级是创建它的执行线程的优先级；
可以通过`setPriority(int newPriority)`更改线程的优先级；通过`getPriority()`获取线程实例的优先级；

线程优先级为 1~10 之间的正整数，JVM 从不会改变一个线程的优先级；
然而，1~10 之间的值是没有保证的；一些 JVM 可能不能识别 10 个不同的值，而将这些优先级进行每两个或多个合并，变成少于 10 个的优先级，则两个或多个优先级的线程可能被映射为一个优先级；

线程默认优先级是 5，Thread 类中有三个常量，定义线程优先级范围：`MAX_PRIORITY`最高优先级、`MIN_PRIORITY`最低优先级、`NORM_PRIORITY`默认优先级；

**线程同步机制**
对于 C/C++，线程之间的同步也是需要借助 pthread 库的，需要预先创建一个同步锁，用于同步，比如互斥锁；
一个线程必须先调用 lock() 获得互斥锁之后才能进入临界区，同时执行完毕后还需要调用 unlock() 释放互斥锁；

但是对于 Java 来说，没有所谓的锁变量；相反，每个对象都拥有自己的隐式锁，当对象的同步方法被调用时调用线程自动获得该对象的锁；
一旦该对象的锁被某个线程持有，那么其他线程都不能再调用该对象的其他非静态 synchronized 方法，除非持有锁的线程退出同步方法或同步代码块；

这种隐式的对象锁使得我们可以编写非常清晰和简洁的多线程代码，因为线程同步支持是 Java 语言内置的；

## 主线程
当 Java 程序启动时，一个线程立刻运行，该线程通常叫做程序的`主线程（main thread）`，因为它是程序开始时就执行的；主线程的重要性体现在两方面：
1) 它是产生其他子线程的线程；
2) 通常它必须最后完成执行，因为它执行各种关闭动作；

无论是 C/C++ 还是 Java，程序的执行都是从 main() 函数开始的；而 main() 函数所在的线程就称之为主线程；
主线程和由主线程创建的新线程一样，没有所谓的父子之分，都是`平级关系`，即线程都是一样的，退出了一个并不会影响其他线程的运行；

但是 main 线程有点特殊，因为 main 线程正常返回之后会隐式的调用 System.exit(status) 来结束整个进程；
因此如果 main 线程先于其他线程退出了，那么所有的未完成的线程都将会挂掉，因为整个进程空间都被 exit() 了；

有没有办法正常结束 main 线程并且不让它调用 exit() 结束当前进程呢？
对于 C/C++，可以使用 pthread 库的 pthread_exit(status) 方法，正常结束 main 线程；
但是对于 Java，我还没有找到合适的方法来结束 main 线程（如果你知道，请告诉我！）；

**获取当前线程的 Thread 对象引用**
尽管主线程在程序启动时自动创建，但它可以由一个 Thread 对象控制；
为此，你必须调用方法 currentThread() 获得它的一个引用，currentThread() 是 Thread 类的公有的静态成员；
它的通常形式如下：`static Thread currentThread()`，该方法返回一个调用它的线程的引用；一旦你获得主线程的引用，你就可以像控制其他线程那样控制主线程；

例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Thread t = Thread.currentThread();  // 获取当前线程的 Thread 引用

        out.printf("currentThread_name: %s\n", t.getName()); // 获取线程名称
        out.printf("currentThread_Priority: %d\n", t.getPriority()); // 获取线程优先级
        out.printf("currentThread: %s\n", t.toString()); // 获取线程描述字符串

        out.printf("sleep ... \n");
        try {
            t.sleep(2000); // ms毫秒
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        out.printf("sleep ... done\n");
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:18:43]
$ javac Main.java

# root @ arch in ~/work on git:master x [20:18:52]
$ java Main
currentThread_name: main
currentThread_Priority: 5
currentThread: Thread[main,5,main]
sleep ...
sleep ... done
</script></code></pre>



线程类 Thread 的 toString() 方法默认返回的格式为：`Thread[线程名, 线程优先级, 线程组]`
从本例中可以得知，main 线程的名字默认为 main，并且线程优先级默认为 5，main 线程所在的线程组是 main；

这里解释一下线程组：
一个线程总是属于某个线程组；默认情况下，如果没有指定线程组，那么自动归到当前线程所属的线程组中；
Java 程序中的线程组由 java.lang.ThreadGroup 类的一个对象表示；Thread 类中的 getThreadGroup() 方法返回一个线程的 ThreadGroup 的引用；

您还可以创建线程组，并在该线程组中放置一个新线程；
要在你的线程组中放置一个新线程，我们必须使用 Thread 类的一个构造函数来接受一个 ThreadGroup 对象作为参数：
<pre><code class="language-java line-numbers"><script type="text/plain">ThreadGroup myGroup = new ThreadGroup("My Thread  Group");
Thread t = new Thread(myGroup, "myThreadName");
</script></code></pre>



线程组以树状结构布置；线程组可以包含另一个线程组；
ThreadGroup 类中的 getParent() 方法返回线程组的父线程组；顶层线程组的父级为 null；

ThreadGroup 类中的某些方法，可以对线程组中的线程产生作用；
例如，setMaxPriority() 方法可以设定线程组中的所有线程拥有最大的优先权；

在创建之初，线程被限制到一个组里，而且不能改变到一个不同的组；每个应用都至少有一个线程从属于系统线程组；若创建多个线程而不指定一个组，它们就会自动归属于系统线程组；

之所以要提出“线程组”的概念，一般认为，是由于“安全”或者“保密”方面的理由；根据 Arnold 和 Gosling 的说法：“线程组中的线程可以修改组内的其他线程，包括那些位于分层结构最深处的；一个线程不能修改位于自己所在组或者下属组之外的任何线程”

**创建线程组**
`public ThreadGroup(String name)`、`public ThreadGroup(ThreadGroup parent, String name)`

**获取线程组信息**
`public int activeCount()`：获得当前线程组中线程数目，包括可运行和不可运行的
`public int activeGroupCount()`：获得当前线程组中活动的子线程组的数目
`public int enumerate(Thread list[])`：列举当前线程组中的线程
`public int enumerate(ThreadGroup list[])`：列举当前线程组中的子线程组
`public final int getMaxPriority()`：获得当前线程组中最大优先级
`public final String getName()`：获得当前线程组的名字
`public final ThreadGroup getParent()`：获得当前线程组的父线程组
`public boolean parentOf(ThreadGroup g)`：判断当前线程组是否为指定线程的父线程
`public boolean isDaemon()`：判断当前线程组中是否有守护线程
`public void list()`：列出当前线程组中所有线程和子线程名

**操作线程组**
`public final void setMaxPriority(int pri)`：设置当前线程组允许的最大优先级
`public final void setDaemon(boolean daemon)`：指定一个线程为当前线程组的守护线程
`public final void suspend()`：挂起当前线程组中所有线程
`public final void resume()`：使被挂起的当前组内的线程恢复到可运行状态
`public final void stop()`：终止当前线程组中所有线程
`public String toStrinng()`：将当前线程组转换为 String 类的对象


**Thread.sleep() 方法**
`public static native void sleep(long millis) throws InterruptedException`：毫秒为单位
`public static void sleep(long millis, int nanos) throws InterruptedException`：毫秒为基本单位，精确到纳秒；

> 
1秒(s) = 1000毫秒(ms)、1秒(s) = 1000,000微秒(μs)、1秒 = 1000,000,000纳秒(ns)

## 创建线程
大多数情况，通过实例化一个 Thread 对象来创建一个线程；Java 定义了两种方式：
1) 实现 Runnable 接口（通常情况）；
2) 继承 Thread 类（除非需要重写 Thread 类的某些方法）；

### Runnable 接口
先看一下 Runnable 接口的定义，很简单，就一个 void run() 方法：
<pre><code class="language-java line-numbers"><script type="text/plain">public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
</script></code></pre>



因此我们仅需要实现 run() 方法，注意，run() 方法和其他的方法没有任何区别，本质都是一个函数，可以被任意调用；
只不过 Thread 类中的 start() 方法会默认将 run() 方法作为新线程的入口函数，仅此而已；

如果你的类实现了 Runnable 接口的 run() 方法，却不作为 Thread 的参数去创建线程，那么这个 run() 方法就和普通的成员函数一样；比如这个例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        MyThread t = new MyThread();
        t.run();
    }
}

class MyThread implements Runnable {
    @Override
    public void run() {
        out.printf("这并不是一个新线程，而是一个普通的成员函数！\n");
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [8:43:23]
$ javac Main.java

# root @ arch in ~/work on git:master x [8:43:31]
$ java Main
这并不是一个新线程，而是一个普通的成员函数！
</script></code></pre>



当一个类实现了 Runnable 接口后，就可以使用 Thread 类的构造函数创建一个线程实例了，构造函数有：
<pre><code class="language-java line-numbers"><script type="text/plain">private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals)

public Thread() {
    init(null, null, "Thread-" + nextThreadNum(), 0);
}

/**
 * @param target    Runnable 对象
 */
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}

/**
 * @param group     线程组
 * @param target    Runnable 对象
 */
public Thread(ThreadGroup group, Runnable target) {
    init(group, target, "Thread-" + nextThreadNum(), 0);
}

/**
 * @param name      线程名称
 */
public Thread(String name) {
    init(null, null, name, 0);
}

/**
 * @param group     线程组
 * @param name      线程名称
 */
public Thread(ThreadGroup group, String name) {
    init(group, null, name, 0);
}

/**
 * @param target    Runnable 对象
 * @param name      线程名称
 */
public Thread(Runnable target, String name) {
    init(null, target, name, 0);
}

/**
 * @param group     线程组
 * @param target    Runnable 对象
 * @param name      线程名称
 */
public Thread(ThreadGroup group, Runnable target, String name) {
    init(group, target, name, 0);
}

/**
 * @param group     线程组
 * @param target    Runnable 对象
 * @param name      线程名称
 * @param stackSize 栈大小
 */
public Thread(ThreadGroup group, Runnable target, String name,
              long stackSize) {
    init(group, target, name, stackSize);
}
</script></code></pre>



实现 Runnable 接口，例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(new Task("A"));
        Thread t2 = new Thread(new Task("B"));

        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Task implements Runnable {
    private String m_s;

    public Task(String s) {
        m_s = s;
    }

    @Override
    public void run() {
        for (int i=1; i<=5; i++) {
            out.printf("[%s] -> %d\n", m_s, i);
            for (long j=0; j<500000000L; j++);
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:23:46]
$ javac Main.java

# root @ arch in ~/work on git:master x [9:24:07]
$ java Main
[A] -> 1
[B] -> 1
[A] -> 2
[B] -> 2
[A] -> 3
[B] -> 3
[A] -> 4
[B] -> 4
[A] -> 5
[B] -> 5
</script></code></pre>



因为默认的线程优先级为 5，所以 A、B 的执行顺序貌似一样，现在我们改变一下优先级：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(new Task("A"));
        Thread t2 = new Thread(new Task("B"));

        // 改变优先级
        t1.setPriority(10);
        t2.setPriority(1);

        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Task implements Runnable {
    private String m_s;

    public Task(String s) {
        m_s = s;
    }

    @Override
    public void run() {
        for (int i=1; i<=5; i++) {
            out.printf("[%s] -> %d\n", m_s, i);
            for (long j=0; j<500000000L; j++);
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:29:56]
$ javac Main.java

# root @ arch in ~/work on git:master x [9:29:58]
$ java Main
[A] -> 1
[B] -> 1
[B] -> 2
[A] -> 2
[B] -> 3
[A] -> 3
[B] -> 4
[A] -> 4
[B] -> 5
[A] -> 5
</script></code></pre>



可以发现，A、B 两个线程还是轮询执行的，有平等的权利获取 CPU 时间片，所以说线程优先级并不能决定线程的真正运行顺序；

### Thread 类
继承 Thread 基类，例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Task t1 = new Task("A");
        Task t2 = new Task("B");

        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Task extends Thread {
    private String m_s;

    public Task(String s) {
        super();
        m_s = s;
    }

    @Override
    public void run() {
        for (int i=1; i<=5; i++) {
            out.printf("[%s] -> %d\n", m_s, i);
            for (long j=0; j<500000000L; j++);
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:44:26] C:1
$ javac Main.java

# root @ arch in ~/work on git:master x [9:44:29]
$ java Main
[A] -> 1
[B] -> 1
[B] -> 2
[A] -> 2
[B] -> 3
[A] -> 3
[B] -> 4
[A] -> 4
[B] -> 5
[A] -> 5
</script></code></pre>



**Thread 类常用方法**
<pre><code class="language-java line-numbers"><script type="text/plain">// 获取当前线程的 Thread 类的引用
public static native Thread currentThread();

/**
 * 线程让步，让当前线程由 Running 状态进入 Runnable 状态，让具有相同优先级的其他线程获得执行机会
 * 但是 yield() 并不保证其他线程一定能够获得执行权，也有可能当前线程再次进入 Running 状态
 */
public static native void yield();

/**
 * 启动线程，由 New 新建状态进入 Runnable 就绪状态
 */
public synchronized void start();

/**
 * [不建议使用]
 * 强行停止线程
 */
public final void stop();
public final synchronized void stop(Throwable obj);

/**
 * [不建议使用]
 * 销毁线程
 */
public void destroy();

/**
 * [不建议使用]
 * 挂起/恢复线程
 */
public final void suspend();
public final void resume();

/**
 * 线程中断 Interrupt
 * 每个 Java 线程都有一个标志位，即中断状态 interrupt status，中断状态有两个值，true、false
 * 默认情况下，interrupt status 值为 false，即本线程没有任何中断
 * 线程收到中断时，通常表示应该停止当前的工作
 * 比如退出循环、停止监听、从 wait/sleep 状态立即结束等
 * 具体处理方式完全由程序自己决定
 * 1. interrupt()方法，给目标线程发送中断信号，即设置中断标志位为 true
 * 2. interrupted()方法，获取当前中断标志位的值，同时清除中断标志位为 false
 * 3. isInterrupted()方法，获取当前中断标志位的值，但是并不清除中断标志位
 */
public void interrupt();
public static boolean interrupted();
public boolean isInterrupted();

/**
 * 判断线程是否处于 Active 状态
 */
public final native boolean isAlive();

/**
 * 线程优先级 [1, 10]
 * 值越大优先级越高，但是不能依赖线程优先级
 */
public final void setPriority(int newPriority);
public final int getPriority();

/**
 * 线程名称
 */
public final synchronized void setName(String name);
public final String getName();

/**
 * 线程组
 */
public final ThreadGroup getThreadGroup();

/**
 * t.join() 方法阻塞调用此方法的线程，直到线程 t 完成，此线程再继续运行
 * 通常用于在 main() 主线程内，等待其它线程完成再结束 main() 主线程
 */
public final synchronized void join(long millis) throws InterruptedException;
public final synchronized void join(long millis, int nanos) throws InterruptedException;
public final void join() throws InterruptedException;

/**
 * 守护线程，Java 中有两种线程：用户线程、守护线程
 * 守护线程和守护进程是差不多的概念，都是用来提供某种后台服务的进程/线程
 * 守护线程使用的情况较少，但并非无用，举例来说，JVM 的垃圾回收、内存管理等线程都是守护线程
 * 但是有一个问题需要注意：
 * JVM 判断一个 Java 程序是否执行完毕的依据是：所有的用户线程执行完毕
 * 当所有的用户线程执行完毕，JVM 自动退出，而程序的守护线程将挂掉！
 * 但是如果在主线程中使用 join() 等待某个守护线程，那么程序将等到该守护线程执行完毕再结束
 */
public final void setDaemon(boolean on);
public final boolean isDaemon();

/**
 * 获取 tid，即线程 ID
 */
public long getId();

/**
 * 获取线程状态
 * public enum State {
 *      NEW,        新建状态
 *      RUNNABLE,   就绪状态
 *      BLOCKED,    阻塞状态
 *      WAITING,    等待状态
 *      TIMED_WAITING, 等待状态(超时)
 *      TERMINATED; 终止状态
 * }
 */
public State getState();
</script></code></pre>



**关于线程中断**
1、一个正常运行的线程，收到中断后，不会有任何问题，可以不需要做任何处理，当然程序可以通过显示的判断，来决定下一步动作，比如：`while (!Thread.currentThread().isInterrupted())`；

2、当线程被 wait()、join()、sleep() 阻塞期间，如果收到中断请求，则会抛出 InterruptedException 异常并清除中断标志；这是中断非常有用的一点，可以提前从阻塞状态返回；

3、做为一种约定，Java API 里任何声明为抛出 InterruptedException 的方法，在抛出异常之前，都会先清除掉中断标志；

4、一般说来，当可能阻塞的方法声明中有抛出 InterruptedException 则暗示该方法是可中断的；

例一：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(new Task("A", 10));
        Thread t2 = new Thread(new Task("B", 20));

        t1.start();
        t2.start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        out.printf("sendto t1: interrupt\n");
        t1.interrupt();

        try {
            Thread.sleep(4000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        out.printf("sendto t2: interrupt\n");
        t2.interrupt();

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Task implements Runnable {
    private String m_s;
    private int m_i;

    public Task(String s, int i) {
        m_s = s;
        m_i = i;
    }

    @Override
    public void run() {
        for (int i=1; i<=m_i; i++) {
            if (Thread.interrupted()) {
                break;
            }

            out.printf("[%s] -> %d\n", m_s, i);
            for (long j=0; j<1000000000L; j++);
        }

        out.printf("[%s] -> interrupted\n", m_s);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:29:21]
$ javac Main.java

# root @ arch in ~/work on git:master x [12:29:32]
$ java Main
[A] -> 1
[B] -> 1
[A] -> 2
[B] -> 2
[B] -> 3
[A] -> 3
sendto t1: interrupt
[B] -> 4
[A] -> interrupted
[B] -> 5
[B] -> 6
[B] -> 7
[B] -> 8
[B] -> 9
[B] -> 10
[B] -> 11
[B] -> 12
[B] -> 13
[B] -> 14
sendto t2: interrupt
[B] -> interrupted
</script></code></pre>



例二：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(new Task("A", 3000));
        t1.start();

        Thread t2 = new Thread(new Task("B", 10000));
        t2.start();

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        t2.interrupt(); // 发送中断请求

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Task implements Runnable {
    private String m_s;
    private int m_millis;

    public Task(String s, int millis) {
        m_s = s;
        m_millis = millis;
    }

    @Override
    public void run() {
        out.printf("[%s] -> will sleep %d ms >>> [%3$tT.%3$tL]\n", m_s, m_millis, new Date());
        try {
            Thread.sleep(m_millis);
            out.printf("[%s] -> sleep timeout >>> [%2$tT.%2$tL]\n", m_s, new Date());
        } catch (InterruptedException e) {
            out.printf("[%s] -> receive interrupt >>> [%2$tT.%2$tL]\n", m_s, new Date());
        }
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [12:51:24]
$ javac Main.java

# root @ arch in ~/work on git:master x [12:51:38]
$ java Main
[B] -> will sleep 10000 ms >>> [12:51:39.814]
[A] -> will sleep 3000 ms >>> [12:51:39.814]
[A] -> sleep timeout >>> [12:51:42.842]
[B] -> receive interrupt >>> [12:51:44.815]
</script></code></pre>


## 线程同步
我们先来看不同步的例子，在 main 线程中启动 3 个新线程，分别打印一个字符串：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Print p = new Print();
        Thread t1 = new Thread(new Task(p, "www.zfl9.com\n"));
        Thread t2 = new Thread(new Task(p, "www.baidu.com\n"));
        Thread t3 = new Thread(new Task(p, "www.google.com\n"));

        t1.start();
        t2.start();
        t3.start();

        try {
            t1.join();
            t2.join();
            t3.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Print {
    public void print(String s) {
        for (int i=0; i<s.length(); i++) {
            out.print(s.charAt(i));
            for (long j=0; j<500000000L; j++);
        }
    }
}

class Task implements Runnable {
    private Print m_p;
    private String m_s;

    public Task(Print p, String s) {
        m_p = p;
        m_s = s;
    }

    @Override
    public void run() {
        m_p.print(m_s);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:06:42]
$ javac Main.java

# root @ arch in ~/work on git:master x [13:06:56]
$ java Main
wwwwwwwww...zgbfoaolig9dl.uec..occmoo
mm


# root @ arch in ~/work on git:master x [13:07:01]
$ java Main
wwwwwwwww...zbgfoalio9dg.ulc.eoc.moc
mo
m
</script></code></pre>



从运行结果中也可以看到，没有同步的线程之间打印的都是乱的，完全没有按我们的预期来运行；

现在我们再来看一下加了同步的例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Print p = new Print();
        Thread t1 = new Thread(new Task(p, "www.zfl9.com\n"));
        Thread t2 = new Thread(new Task(p, "www.baidu.com\n"));
        Thread t3 = new Thread(new Task(p, "www.google.com\n"));

        t1.start();
        t2.start();
        t3.start();

        try {
            t1.join();
            t2.join();
            t3.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Print {
    public synchronized void print(String s) {
        for (int i=0; i<s.length(); i++) {
            out.print(s.charAt(i));
            for (long j=0; j<500000000L; j++);
        }
    }
}

class Task implements Runnable {
    private Print m_p;
    private String m_s;

    public Task(Print p, String s) {
        m_p = p;
        m_s = s;
    }

    @Override
    public void run() {
        m_p.print(m_s);
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:10:11] C:127
$ javac Main.java

# root @ arch in ~/work on git:master x [13:10:14]
$ java Main
www.zfl9.com
www.google.com
www.baidu.com
</script></code></pre>



对比未同步的代码，我们只是在 Print 类的 print() 方法的返回值前面添加了关键字`synchronized`而已；

**对象锁、类锁**
首先，需要明确的一点是，Java 语言默认为每个类和每个对象内置了一个互斥锁；
注意是每个类、每个对象，也就是说，互斥锁分为类级别、对象级别；

类锁：作用于`静态`同步方法；
对象锁：作用于`非静态`同步方法；

1) 如果一个线程成功进入了一个对象的非静态同步方法，那么该线程将获得该对象锁，在锁未释放期间，其他任何线程将不能再访问该对象的其他非静态同步方法；
2) 如果一个线程成功进入了一个类的静态同步方法，那么该线程将获得该类锁，在锁未释放期间，其他任何线程将不能再访问该类的其他静态同步方法；

有一点需要强调，C/C++ 中需要显式的调用 mutex.lock()、mutex.unlock() 方法来获得、释放互斥锁；
但是在 Java 中，这些操作都是隐含的，当一个线程进入一个 synchronized 方法/块，那么它将自动获得互斥锁，当线程离开 synchronized 方法/块，则自动释放互斥锁，这一切都是由 Java 自动完成的；

**synchronized 关键字**
1) synchronized 方法：将 synchronized 关键字放在方法的返回值类型之前，表示该方法是 synchronized 的；
2) synchronized 代码块：`synchronized (obj) { statements }`获取对象 obj 的对象锁，`synchronized (xxx.class) { statements }`获取类 xxx 的类锁；

> 
对于 synchronized 方法：`synchronized void func() { statements }`，等同于：`void func() { synchronized (this) { statements } }`

关于同步锁，需要注意的几点：

1) 如果线程不能获得锁会怎么样？
如果线程试图进入同步方法，而其锁已经被占用，则线程在该对象上被阻塞，对于类锁也是一样的；
实质上，线程进入该对象（或类）的`锁池`中，必须在那里等待，直到其锁再次被释放，在锁池中的线程将再次竞争锁，竞争成功的线程进入同步方法，锁池中其他线程将继续等待锁的再次释放，然后一直持续这样的过程；

2) 当考虑阻塞时，一定要注意哪个对象正被用于锁定：
1、调用同一个对象中非静态同步方法的线程将彼此阻塞；如果是不同对象，则每个线程有自己的对象的锁，线程间彼此互不干预；
2、调用同一个类中的静态同步方法的线程将彼此阻塞；它们都是锁定在相同的 Class 对象上；
3、静态同步方法和非静态同步方法将永远不会彼此阻塞，因为静态方法锁定在 Class 对象上，非静态方法锁定在该类的对象上；
4、对于同步代码块，要看清楚什么对象已经用于锁定（synchronized 后面括号的内容）；在同一个对象上进行同步的线程将彼此阻塞，在不同对象上锁定的线程将永远不会彼此阻塞；
