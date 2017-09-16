---
title: Java Executor框架入门
date: 2017-09-14 18:13:00
categories:
- java
tags:
- java
keywords: Java Executor框架 Java并发框架 线程池ThreadPool J.U.C
---

> 
Java Executor框架入门，线程池、阻塞队列、阻塞栈、Callable接口、互斥锁、读写锁、条件变量、信号量等 Java 5 新 API；

<!-- more -->

## J.U.C
J.U.C：即 java.util.concurrent 的缩写，该包参考自 EDU.oswego.cs.dl.util.concurrent，是 JSR 166 标准规范的一个实现；
那么，JSR 166 以及 J.U.C 包的作者是谁呢，没错，就是 Doug Lea 大神，挺牛逼的，大神级别的任务，膜拜；

Executor 框架是 Java 5 中引入的，其内部使用了线程池机制，它在 java.util.cocurrent 包下，通过该框架来控制线程的启动、执行和关闭，可以简化并发编程的操作；

合理使用线程池能够带来 3 个好处：
1) 降低资源消耗：通过重复利用已创建的线程降低线程创建和销毁造成的消耗；
2) 提高响应速度；当任务到达时，任务可以不需要等到线程创建就立即执行；
3) 提高线程的可管理性；线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以统一的分配、调优和监控；

Executor 框架包括：ThreadPoolExecutor 线程池（核心类），Executors 工具类（提供工厂函数创建线程池）、阻塞队列/栈、Callable 接口、Lock 互斥锁、ReadWriteLock 读写锁、Condition 条件变量等一系列多线程相关的 API；

> 
concurrent 包是 jdk1.5 引入的重要的包，主要代码由大牛 Doug Lea 完成，其实是在 jdk1.4 时代，由于 java 语言内置对多线程编程的支持比较基础和有限，所以他写了这个，因为实在太过于优秀，所以被加入到 jdk 之中；

通常所说的 concurrent 包基本有 3 个 package 组成：
`java.util.concurrent`：提供大部分关于并发的接口和类，如 BlockingQueue、Callable、ConcurrentHashMap、ExecutorService、Semaphore 等；
`java.util.concurrent.atomic`：提供所有原子操作的类，如 AtomicInteger、AtomicLong 等；
`java.util.concurrent.locks`：提供锁相关的类，如 Lock、ReadWriteLock、Condition 等；

concurrent 包的优点：
1) 首先，功能非常丰富，诸如`线程池(ThreadPoolExecutor)`，CountDownLatch 等并发编程中需要的类已经有现成的实现，不需要自己去实现一套；毕竟 jdk1.4 对多线程编程的主要支持几乎就只有 Thread、Runnable、synchronized 等；
2) concurrent 包里面的一些操作是基于硬件级别的`CAS(compare and swap)`，就是在 cpu 级别提供了原子操作，简单的说就可以提供无阻塞、无锁定的算法；而现代 cpu 大部分都是支持这样的算法的；

## 线程池
继承关系图：
![线程池继承关系图](/images/java-executor.jpg)

最顶层是 Executor 接口，它的定义很简单，一个用于执行任务的 execute 方法：
<pre><code class="language-bash line-numbers"><script type="text/plain">
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
</script></code></pre>



ExecutorService 接口继承自 Executor 接口，它提供了更丰富的实现多线程的方法，比如，ExecutorService 提供了 shutdown() 线程池的方法，以及可为跟踪一个或多个异步任务执行状况而生成 Future 的方法等：
<pre><code class="language-bash line-numbers"><script type="text/plain">
public interface ExecutorService extends Executor {
    void shutdown(); // 平滑关闭
    List<Runnable> shutdownNow(); // 强制关闭

    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;

    // 提交任务(Runnable、Callable对象)
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);

    // 批量提交任务(阻塞)
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
</script></code></pre>



AbstractExecutorService 抽象类实现了 ExecutorService 接口的大部分方法；
ThreadPoolExecutor 核心类，创建自定义线程池就靠它了，下面是 4 个构造方法：
<pre><code class="language-bash line-numbers"><script type="text/plain">
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         threadFactory, defaultHandler);
}

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          RejectedExecutionHandler handler) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), handler);
}

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    // ...... 实现细节
}
</script></code></pre>



**ThreadPoolExecutor 构造函数**
前三个构造函数都是调用的第四个构造函数进行初始化操作，各参数的作用：
1) `corePoolSize`：线程池中核心线程数的最大值；
2) `maximumPoolSize`：线程池中能同时拥有的最大线程数；
3) `keepAliveTime`：空闲线程的存活时间；
4) `unit`：keepAliveTime 的时间单位，`TimeUnit.NANOSECONDS`纳秒、`TimeUnit.MICROSECONDS`微秒、`TimeUnit.MILLISECONDS`毫秒、`TimeUnit.SECONDS`秒、`TimeUnit.MINUTES`分、`TimeUnit.HOURS`时、`TimeUnit.DAYS`天；
5) `workQueue`：缓存任务的阻塞队列；
6) `threadFactory`：创建线程的工厂；
7) `handler`：当 workQueue 已满，且线程数达 maximumPoolSize 时，线程池拒绝新任务采取的策略；

> 
一个线程即一个执行流，一个任务即一个 Runnable/Callable 对象；

**提交任务之后的流程**
当试图通过 execute() 方法（submit() 也是调用 execute() 实现的）将一个任务添加到线程池中时，按照如下顺序处理：
1) 如果线程池中的线程数量少于 corePoolSize，即使线程池中有空闲线程，也会创建一个新线程来执行新添加的任务；
2) 如果线程池中的线程数量为 corePoolSize，并且缓冲队列 workQueue 未满，则将新添加的任务放到 workQueue 中，等待线程池中的空闲线程按照 FIFO 原则依次从队列中取出任务并执行；
3) 如果线程池中的线程数量为 corePoolSize，并且缓冲队列 workQueue 已满，则创建新的线程（临时工）来执行新添加的任务，直到线程池中的线程数达到 maximumPoolSize；
4) 如果线程池中的线程数量达到 maximumPoolSize，则按照饱和策略进行处理，默认为丢弃任务并抛出 RejectedExecutionException 异常；
5) 当（临时工）线程在线程池中的空闲时间超过 keepAliveTime 后，该（临时工）线程将被自动结束，移出线程池，直到线程数恢复到 corePoolSize；

> 
线程池并没有标记哪个线程是核心线程，哪个是非核心线程，线程池只关心核心线程的数量；

**线程池工作流程简述**
通俗解释，如果把线程池比作一个单位的话，corePoolSize 就表示正式工，线程就可以表示一个员工；
当我们向单位委派一项工作时，如果单位发现正式工还没招满，单位就会招个正式工来完成这项工作；

随着我们向这个单位委派的工作增多，即使正式工全部满了，工作还是干不完，那么单位只能按照我们新委派的工作按先后顺序将它们找个地方搁置起来，这个地方就是 workQueue，等正式工完成了手上的工作，就到这里来取新的任务；

如果不巧，年末了，各个部门都向这个单位委派任务，导致 workQueue 已经没有空位置放新的任务，于是单位决定招点临时工吧（临时工：又是我！）；
临时工也不是想招多少就招多少，上级部门通过这个单位的 maximumPoolSize 确定了你这个单位的人数的最大值，换句话说最多招 maximumPoolSize - corePoolSize 个临时工；当然，在线程池中，谁是正式工，谁是临时工是没有区别的，完全同工同酬；

如果单位已经招了些临时工，但新任务没有继续增加，所以随着每个员工忙完手头的工作，都来 workQueue 领取新的任务；
随着各个员工齐心协力，任务越来越少，员工数没变，那么就必定有闲着没事干的员工；于是领导想了个办法，设定了 keepAliveTime，当空闲的员工在 keepAliveTime 这段时间还没有找到事情干，就被辞退啦，毕竟地主家也没有余粮啊！当然辞退到 corePoolSize 个员工时就不再辞退了，领导也不想当光杆司令啊；

如果单位招满了临时工，但新任务依然继续增加，线程池从上到下，从里到外真心忙的不可开交，阻塞队列也满了，只好拒绝上级委派下来的任务；怎么拒绝也是一门艺术哈；

**预启动线程**
在创建了线程池后，默认情况下，线程池中并没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用 prestartAllCoreThreads() 和 prestartCoreThread() 方法，从方法名字可以看出，是预创建线程的意思，即在没有任务到来之前，就创建 corePoolSize 个线程或 1 个线程；

**keepAliveTime 超时**
默认情况下，keepAliveTime 只在线程数大于 corePoolSize 时才会生效；
但是如果调用了`allowCoreThreadTimeOut(true)`方法，在线程池中的线程数不大于 corePoolSize 时，keepAliveTime 参数也会起作用，直到线程池中的线程数为 0；

**阻塞队列**
workQueue 阻塞队列，用来存储等待执行的任务；该参数也很重要，会对线程池的运行过程产生巨大影响，一般而言，有一下几种选择：
1) `ArrayBlockingQueue`：基于数组结构的有界阻塞队列，此队列按 FIFO 原则对元素进行排序；
2) `LinkedBlockingQueue`：基于链表结构的有界（默认为 Integer.MAX_VALUE）阻塞队列，此队列按 FIFO 排序元素，吞吐量通常要高于 ArrayBlockingQueue；
3) `SynchronousQueue`：不存储元素（可理解为长度为 1 的队列）的无界阻塞队列；每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，反之亦然；吞吐量通常要高于 LinkedBlockingQueue；
4) `PriorityBlockingQueue`：支持优先级的无界阻塞队列；

**饱和策略**
饱和策略，即当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务；
这个策略默认情况下是 AbortPolicy，表示无法处理新任务时抛出异常；以下是 JDK1.5 提供的四种策略（内部类）：
1) `ThreadPoolExecutor.AbortPolicy`：丢弃任务，并抛出 RejectedExecutionException 运行时异常；
2) `ThreadPoolExecutor.DiscardPolicy`：丢弃新任务，不抛出异常；
3) `ThreadPoolExecutor.DiscardOldestPolicy`：丢弃最旧任务（最先提交却未得到执行的任务），然后重新尝试执行新任务；
4) `ThreadPoolExecutor.CallerRunsPolicy`：由调用线程处理该任务；

**ThreadPoolExecutor 常用方法**
`execute()`：Executor 声明的方法，在 ThreadPoolExecutor 中得到实现，核心方法，用于提交 Runnable 任务；
`submit()`：ExecutorService 声明的方法，在 AbstractExecutorService 中得到实现，可以提交 Callable 任务，并返回一个 Future 对象，本质还是调用的 execute()；
`shutdown()`：关闭线程池（平滑关闭）；
`shutdownNow()`：关闭线程池（强制关闭）；

> 
submit() 主要用于提交 Callable 任务，返回一个 Future 对象，可以通过 get() 方法获取返回值；
如果使用 submit() 提交 Runnable 任务，也会返回一个 Future 对象，但是通过 get() 方法获取的是 null；

**线程池的状态**
线程池具有以下五种状态，当创建一个线程池时初始化状态为 RUNNING：
`RUNNING`：允许提交并处理任务；
`SHUTDOWN`：不允许提交新的任务，但是会处理完已提交的任务；
`STOP`：不允许提交新的任务，也不会处理阻塞队列中未执行的任务，并设置正在执行的线程的中断标志位；
`TIDYING`：所有任务执行完毕，池中工作的线程数为 0，等待执行 terminated() 勾子方法；
`TERMINATED`：terminated() 勾子方法执行完毕；

注意，这里说的是线程池的状态而不是池中线程的状态；

调用线程池的 shutdown() 方法，将线程池由 RUNNING（运行状态）转换为 SHUTDOWN 状态；
调用线程池的 shutdownNow() 方法，将线程池由 RUNNING 或 SHUTDOWN 状态转换为 STOP 状态；
SHUTDOWN 状态和 STOP 状态先会转变为 TIDYING 状态，最终都会变为 TERMINATED 状态；

**ThreadPoolExecutor 常用构造方法**
实际上 ThreadPoolExecutor 类中还有很多重载的构造函数，下面这个构造函数在 Executors 中经常用到：
<pre><code class="language-bash line-numbers"><script type="text/plain">
public ThreadPoolExecutor(int corePoolSize,
        int maximumPoolSize,
        long keepAliveTime,
        TimeUnit unit,
        BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
    Executors.defaultThreadFactory(), defaultHandler);
}
</script></code></pre>



注意到上述的构造方法使用 Executors 中的 defaultThreadFactory() 线程工厂和 ThreadPoolExecutor 中的 defaultHandler 抛弃策略；

使用 defaultThreadFactory 创建的线程同属于相同的线程组，具有同为 Thread.NORM_PRIORITY 的优先级，以及名为"pool-XXX-thread-xxx"的线程名，且创建的线程都是非守护进程；
defaultHandler 缺省饱和策略是 ThreadPoolExecutor.AbortPolicy()；

除了在创建线程池时指定上述参数的值外，还可在线程池创建以后通过如下方法进行设置：
`public void setCorePoolSize(int corePoolSize)`：设置 corePoolSize 大小；
`public void setMaximumPoolSize(int maximumPoolSize)`：设置 maximumPoolSize 大小；
`public void allowCoreThreadTimeOut(boolean value)`：将 keepAliveTime 超时策略应用于 core 线程；
`public void setKeepAliveTime(long time, TimeUnit unit)`：设置 keepAliveTime 时间；
`public void setThreadFactory(ThreadFactory threadFactory)`：设置 ThreadFactory；
`public void setRejectedExecutionHandler(RejectedExecutionHandler handler)`：设置 RejectedExecutionException 策略；

**线程池的例子**
<pre><code class="language-bash line-numbers"><script type="text/plain">
import static java.lang.System.*;
import java.util.*;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ThreadPoolExecutor pool = new ThreadPoolExecutor(5, 5, // nCore: 5, nMax: 5
                                                      0L, TimeUnit.SECONDS, // idle-timeout: 0s
                                                      new LinkedBlockingQueue<Runnable>(50)); // 队列长度: 50
        pool.prestartAllCoreThreads(); // 预启动所有 Core 线程
        for (int i=0; i<30; i++) {
            pool.execute(new Task(i));
        }
        pool.shutdown(); // 关闭线程池
    }
}

class Task implements Runnable {
    private int n;
    public Task(int n) { this.n = n; }
    @Override
    public void run() {
        Thread t = Thread.currentThread();
        out.printf("[tid: %d, %s] -> Task%d\n", t.getId(), t.getName(), n);
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [11:59:03]
$ javac Main.java

# root @ arch in ~/work on git:master x [11:59:15]
$ java Main
[tid: 10, pool-1-thread-2] -> Task1
[tid: 10, pool-1-thread-2] -> Task5
[tid: 10, pool-1-thread-2] -> Task6
[tid: 11, pool-1-thread-3] -> Task2
[tid: 11, pool-1-thread-3] -> Task8
[tid: 11, pool-1-thread-3] -> Task9
[tid: 11, pool-1-thread-3] -> Task10
[tid: 11, pool-1-thread-3] -> Task11
[tid: 11, pool-1-thread-3] -> Task12
[tid: 11, pool-1-thread-3] -> Task13
[tid: 11, pool-1-thread-3] -> Task14
[tid: 11, pool-1-thread-3] -> Task15
[tid: 11, pool-1-thread-3] -> Task16
[tid: 11, pool-1-thread-3] -> Task17
[tid: 11, pool-1-thread-3] -> Task18
[tid: 11, pool-1-thread-3] -> Task19
[tid: 11, pool-1-thread-3] -> Task20
[tid: 11, pool-1-thread-3] -> Task21
[tid: 11, pool-1-thread-3] -> Task22
[tid: 12, pool-1-thread-4] -> Task3
[tid: 12, pool-1-thread-4] -> Task23
[tid: 13, pool-1-thread-5] -> Task4
[tid: 9, pool-1-thread-1] -> Task0
[tid: 9, pool-1-thread-1] -> Task26
[tid: 9, pool-1-thread-1] -> Task27
[tid: 9, pool-1-thread-1] -> Task28
[tid: 9, pool-1-thread-1] -> Task29
[tid: 11, pool-1-thread-3] -> Task25
[tid: 12, pool-1-thread-4] -> Task24
[tid: 10, pool-1-thread-2] -> Task7
</script></code></pre>

> 
实际使用中，很少要自己配置线程池的参数，并且如果在不了解线程池原理的情况下，也很难准确的设置合适的参数；
所以 Executors 类出现了，Executors 提供了几个创建常用配置的线程池的 static 工厂函数，极大方便了 Java 程序的编写；
不过，如果 Executors 创建的一般线程池无法满足实际生产需求，那么还是需要创建自定义线程池，即通过 ThreadPoolExecutor；

**Executors 类**
通过 Executors 工具类可以创建各种类型的线程池，如下为常见的四种：
1) `newCachedThreadPool`
<pre><code class="language-bash line-numbers"><script type="text/plain">
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
</script></code></pre>


corePoolSize 为 0，maximumPoolSize 为 Integer.MAX_VALUE，使用 SynchronousQueue 作为阻塞队列，线程的空闲时限为 60 秒；
这种类型的线程池非常适用 IO 密集的服务；因为 IO 请求具有密集、数量巨大、不持续、服务器端 CPU 等待 IO 响应时间长的特点；服务器端为了能提高 CPU 的使用率就应该为每个 IO 请求都创建一个线程，以免 CPU 因为等待 IO 响应而空闲；

2) `newFixedThreadPool`
<pre><code class="language-bash line-numbers"><script type="text/plain">
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
</script></code></pre>


corePoolSize 和 maximumPoolSize 都为传入的参数 nThreads，使用 LinkedBlockingQueue 作为阻塞队列，队列长度为 Integer.MAX_VALUE，线程空闲时间 0 毫秒；
这种类型的线程池可以适用 CPU 密集的工作，在这种工作中 CPU 忙于计算而很少空闲，由于 CPU 能真正并发的执行的线程数是一定的（比如四核八线程），所以对于那些需要 CPU 进行大量计算的线程，创建的线程数超过 CPU 能够真正并发执行的线程数就没有太大的意义；

3) `newSingleThreadExecutor`
<pre><code class="language-bash line-numbers"><script type="text/plain">
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
</script></code></pre>


corePoolSize 和 maximumPoolSize 都为 1，线程空闲时间为 0 毫秒，使用 LinkedBlockingQueue 阻塞队列；newSingleThreadExecutor 可以看作是 newFixedThreadPool 的一种；
newSingleThreadExecutor 线程池中只有一个线程工作，它能保证按照任务提交的顺序来执行任务，适合串行执行的任务；

4) `newScheduledThreadPool`
<pre><code class="language-bash line-numbers"><script type="text/plain">
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

// ScheduledThreadPoolExecutor 构造函数
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
</script></code></pre>


corePoolSize 为传入的参数，maximumPoolSize 为 Integer.MAX_VALUE，线程空闲时间为 0 纳秒，阻塞队列为 DelayedWorkQueue；
newScheduledThreadPool 线程池适合执行计划任务、周期任务；


**提交任务 execute()、submit()**
execute() 方式：
<pre><code class="language-bash line-numbers"><script type="text/plain">
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();

    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }

    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
</script></code></pre>


submit() 方式：
<pre><code class="language-bash line-numbers"><script type="text/plain">
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}

public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
</script></code></pre>



**批量任务的执行方式**
方式一：首先定义任务集合，然后定义 Future 集合用于存放执行结果，执行任务，最后遍历 Future 集合获取结果；
优点：可以依次得到有序的结果；
缺点：不能及时获取已完成任务的执行结果；

方式二：首先定义任务集合，通过 CompletionService 包装 ExecutorService，执行任务，然后调用其 take() 方法去取 Future 对象；
优点：及时得到已完成任务的执行结果；
缺点：不能依次得到结果；

在方式一中，从集合中遍历的每个 Future 对象并不一定处于完成状态，这时调用 get() 方法就会被阻塞住，所以后面的任务即使已完成也不能得到结果；
而方式二中，CompletionService 的实现是维护一个保存 Future 对象的 BlockingQueue，只有当这个 Future 对象状态是结束的时候，才会加入到这个 Queue 中，所以调用 take() 能从阻塞队列中拿到最新的已完成任务的结果；

方式一的例子：
<pre><code class="language-bash line-numbers"><script type="text/plain">
import static java.lang.System.*;
import java.util.*;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newCachedThreadPool();
        ArrayList<Future<String>> results = new ArrayList<Future<String>>(5);

        results.add(pool.submit(new Task("Task-A")));
        results.add(pool.submit(new Task("Task-B")));
        results.add(pool.submit(new Task("Task-C")));
        results.add(pool.submit(new Task("Task-D")));
        results.add(pool.submit(new Task("Task-E")));

        for (Future<String> future : results) {
            while (!future.isDone());
            try {
                out.printf("task result: %s\n", future.get());
            } catch (InterruptedException | ExecutionException e) {
                out.println(e);
            }
        }

        pool.shutdown(); // 关闭线程池
    }
}

class Task implements Callable<String> {
    private String str;

    public Task(String str) {
        this.str = str;
    }

    @Override
    public String call() {
        try {
            Thread.sleep(new Random().nextInt(3000)); // 随机睡眠, 0-2999毫秒
        } catch (InterruptedException e) {
            out.println(e);
        }
        return str;
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [10:54:23]
$ javac Main.java

# root @ arch in ~/work on git:master x [10:55:02]
$ java Main
task result: Task-A
task result: Task-B
task result: Task-C
task result: Task-D
task result: Task-E
</script></code></pre>



方式二的例子：
<pre><code class="language-bash line-numbers"><script type="text/plain">
import static java.lang.System.*;
import java.util.*;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newCachedThreadPool();
        CompletionService<String> completionService = new ExecutorCompletionService<String>(pool);

        completionService.submit(new Task("Task-A"));
        completionService.submit(new Task("Task-B"));
        completionService.submit(new Task("Task-C"));
        completionService.submit(new Task("Task-D"));
        completionService.submit(new Task("Task-E"));

        Future<String> future = null;
        for (int i=0; i<5; i++) {
            try {
                future = completionService.take();
                out.printf("task result: %s\n", future.get());
            } catch (InterruptedException | ExecutionException e) {
                out.println(e);
            }
        }

        pool.shutdown(); // 关闭线程池
    }
}

class Task implements Callable<String> {
    private String str;

    public Task(String str) {
        this.str = str;
    }

    @Override
    public String call() {
        try {
            Thread.sleep(new Random().nextInt(3000)); // 随机睡眠, 0-2999毫秒
        } catch (InterruptedException e) {
            out.println(e);
        }
        return str;
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [11:06:17]
$ javac Main.java

# root @ arch in ~/work on git:master x [11:06:20]
$ java Main
task result: Task-D
task result: Task-C
task result: Task-E
task result: Task-B
task result: Task-A

# root @ arch in ~/work on git:master x [11:06:23]
$ java Main
task result: Task-D
task result: Task-C
task result: Task-A
task result: Task-B
task result: Task-E

# root @ arch in ~/work on git:master x [11:06:28]
$ java Main
task result: Task-A
task result: Task-D
task result: Task-C
task result: Task-E
task result: Task-B
</script></code></pre>


## Callable接口
在 Java5 之前，线程是没有返回值的，常常为了“有”返回值，破费周折，而且代码很不好写；或者干脆绕过这道坎，走别的路了；
现在 Java 终于有可返回值的任务（线程）了，可返回值的任务必须实现 Callable 泛型接口，类似的，无返回值的任务必须实现 Runnable 接口；
执行 Callable 任务后，可以获取一个 Future 对象，在该对象上调用 get() 就可以获取到 Callable 任务返回的结果了；

**Runnable、Callable 关系**
在 Java5 以后，一个可以调度执行的线程单元可以有三种方式定义：Thread、Runnable、Callable；
Runnable 实现的是 void run() 方法，Callable 实现的是 V call() 方法，并且可以返回执行结果；
其中 Runnable 可以提交给 Thread 来包装下，直接启动一个线程来执行，而 Callable 则一般都是提交给 ExecuteService 来执行；

**Callable 接口**
Callable 定义在 java.util.concurrent 包中，只有一个`<V> call() throws Exception;`方法，Callable 是一个泛型接口：
<pre><code class="language-bash line-numbers"><script type="text/plain">
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
</script></code></pre>



**Future 接口**
一个 Future 表示一个异步计算的结果；它提供了一系列方法，用来检测计算是否完成，获取计算结果，等待计算结果等；
使用 get() 方法获取计算结果时，如果计算没有完成，get() 方法会一直等待，直到任务完成；
如果你想使用 Future，但并不需要返回一个结果，则可以使用`Future<?>`并返回 null；
<pre><code class="language-bash line-numbers"><script type="text/plain">
public interface Future<V> {

    /**
     * Attempts to cancel execution of this task.  This attempt will
     * fail if the task has already completed, has already been cancelled,
     * or could not be cancelled for some other reason. If successful,
     * and this task has not started when {@code cancel} is called,
     * this task should never run.  If the task has already started,
     * then the {@code mayInterruptIfRunning} parameter determines
     * whether the thread executing this task should be interrupted in
     * an attempt to stop the task.
     *
     * <p>After this method returns, subsequent calls to {@link #isDone} will
     * always return {@code true}.  Subsequent calls to {@link #isCancelled}
     * will always return {@code true} if this method returned {@code true}.
     *
     * @param mayInterruptIfRunning {@code true} if the thread executing this
     * task should be interrupted; otherwise, in-progress tasks are allowed
     * to complete
     * @return {@code false} if the task could not be cancelled,
     * typically because it has already completed normally;
     * {@code true} otherwise
     */
    boolean cancel(boolean mayInterruptIfRunning);

    /**
     * Returns {@code true} if this task was cancelled before it completed
     * normally.
     *
     * @return {@code true} if this task was cancelled before it completed
     */
    boolean isCancelled();

    /**
     * Returns {@code true} if this task completed.
     *
     * Completion may be due to normal termination, an exception, or
     * cancellation -- in all of these cases, this method will return
     * {@code true}.
     *
     * @return {@code true} if this task completed
     */
    boolean isDone();

    /**
     * Waits if necessary for the computation to complete, and then
     * retrieves its result.
     *
     * @return the computed result
     * @throws CancellationException if the computation was cancelled
     * @throws ExecutionException if the computation threw an
     * exception
     * @throws InterruptedException if the current thread was interrupted
     * while waiting
     */
    V get() throws InterruptedException, ExecutionException;

    /**
     * Waits if necessary for at most the given time for the computation
     * to complete, and then retrieves its result, if available.
     *
     * @param timeout the maximum time to wait
     * @param unit the time unit of the timeout argument
     * @return the computed result
     * @throws CancellationException if the computation was cancelled
     * @throws ExecutionException if the computation threw an
     * exception
     * @throws InterruptedException if the current thread was interrupted
     * while waiting
     * @throws TimeoutException if the wait timed out
     */
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
</script></code></pre>



**FutureTask 类**
FutureTask 表示一个可以取消的异步计算任务；它实现了 Runnable 接口和 Future 接口；
由于 FutureTask 实现了 Runnable，因此它既可以通过 Thread 包装来直接执行，也可以提交给 ExecuteService 来执行；
并且还可以直接通过 get() 函数获取执行结果，该函数会阻塞，直到结果返回；
因此 FutureTask 既是 Future、Runnable，又是包装了 Callable(如果是 Runnable 最终也会被转换为 Callable)，它是这两者的结合体；
<pre><code class="language-bash line-numbers"><script type="text/plain">
public class FutureTask<V> implements RunnableFuture<V> {
    /**
     * Creates a {@code FutureTask} that will, upon running, execute the
     * given {@code Callable}.
     *
     * @param  callable the callable task
     * @throws NullPointerException if the callable is null
     */
    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }

    /**
     * Creates a {@code FutureTask} that will, upon running, execute the
     * given {@code Runnable}, and arrange that {@code get} will return the
     * given result on successful completion.
     *
     * @param runnable the runnable task
     * @param result the result to return on successful completion. If
     * you don't need a particular result, consider using
     * constructions of the form:
     * {@code Future<?> f = new FutureTask<Void>(runnable, null)}
     * @throws NullPointerException if the runnable is null
     */
    public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
    }
}

public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
</script></code></pre>



Callable + ExecutorService 方式：
<pre><code class="language-bash line-numbers"><script type="text/plain">
import static java.lang.System.*;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newSingleThreadExecutor();
        Future<String> future = pool.submit(new Task("This is the result of the execution\n"));
        while (!future.isDone());
        try {
            out.print(future.get());
        } catch (InterruptedException | ExecutionException e) {
            out.println(e);
        }
        pool.shutdown();
    }
}

class Task implements Callable<String> {
    private String str;
    public Task(String str) { this.str = str; }
    @Override
    public String call() {
        out.printf("[%s] -> running ... \n", Thread.currentThread().getName());
        return str;
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [13:54:30]
$ javac Main.java

# root @ arch in ~/work on git:master x [13:54:43]
$ java Main
[pool-1-thread-1] -> running ...
This is the result of the execution
</script></code></pre>


Callable + FutureTask 方式：
<pre><code class="language-bash line-numbers"><script type="text/plain">
import static java.lang.System.*;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        FutureTask<String> ft = new FutureTask<String>(new Task("This is the result of the execution\n"));
        Thread t = new Thread(ft);
        t.start();
        while (!ft.isDone());
        try {
            out.print(ft.get());
        } catch (InterruptedException | ExecutionException e) {
            out.println(e);
        }
    }
}

class Task implements Callable<String> {
    private String str;
    public Task(String str) { this.str = str; }
    @Override
    public String call() {
        out.printf("[%s] -> running ... \n", Thread.currentThread().getName());
        return str;
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [13:59:26]
$ javac Main.java

# root @ arch in ~/work on git:master x [13:59:41]
$ java Main
[Thread-0] -> running ...
This is the result of the execution
</script></code></pre>


## Lock锁
在 Java5 中，专门提供了锁对象，利用锁可以方便的实现资源的封锁，用来控制对竞争资源并发访问的控制，这些内容主要集中在 java.util.concurrent.locks 包下面，里面有三个重要的接口 Lock、ReadWriteLock、Condition；

1) `Lock`：互斥锁，Lock 提供了比 synchronized 方法和语句块更广泛、更细粒度的锁定操作；
2) `ReadWriteLock`：读写锁，ReadWriteLock 分为读锁、写锁，它们是一个整体，读锁有任意多把，写锁只有一把，读锁和写锁不能同一时间锁定；
3) `Condition`：条件变量，Condition 将 Object 监视器方法（wait、notify 和 notifyAll）分解成截然不同的对象，以便通过将这些对象与任意 Lock 实现组合使用；

Lock 可以说是 synchronized 的一个替代品，synchronized 能做的事，lock 基本都可以做，而且能做得更好；他们的一些区别是：
1) Lock 在获取锁的过程可以被中断；
2) Lock 可以尝试获取锁，如果锁被其他线程持有，则返回 false，不会使当前线程阻塞；
3) Lock 在尝试获取锁的时候，传入一个时间参数，如果在这个时间范围内，没有获得锁，那么就终止请求；
4) synchronized 会自动释放锁，Lock 则不会自动释放锁，需要调用 unlock() 进行释放；

这样可以看到，Lock 比起 synchronized 具有更细粒度的控制；但是也不是说 Lock 就完全可以取代 synchronized，因为 Lock 的学习成本，复杂度等方面要比 synchronized 高，对于初级 Java 程序员，使用 synchronized 的风险要比 Lock 低；

**Lock 锁相关的接口、类**
所在的包：java.util.concurrent.locks
接口：Lock、ReadWriteLock、Condition；
实现：ReentrantLock、ReentrantReadWriteLock、ConditionObject（AbstractQueuedSynchronizer、AbstractQueuedLongSynchronizer 中）；

**Lock 接口**
<pre><code class="language-bash line-numbers"><script type="text/plain">
public interface Lock {
    void lock(); // 不可中断
    void lockInterruptibly() throws InterruptedException; // 可中断
    boolean tryLock(); // 尝试获取锁，成功返回 true，失败返回 false
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException; // 同上，支持超时
    void unlock(); // 释放锁
    Condition newCondition(); // 返回配对的 Condition
}
</script></code></pre>



**ReadWriteLock 接口**
<pre><code class="language-bash line-numbers"><script type="text/plain">
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
</script></code></pre>



**Condition 接口**
<pre><code class="language-bash line-numbers"><script type="text/plain">
public interface Condition {
    void await() throws InterruptedException; // 可中断 wait
    void awaitUninterruptibly(); // 不可中断 wait
    long awaitNanos(long nanosTimeout) throws InterruptedException; // 可中断 wait，超时等待（纳秒）
    boolean await(long time, TimeUnit unit) throws InterruptedException; // 可中断 wait，超时等待
    boolean awaitUntil(Date deadline) throws InterruptedException; // 可中断 wait，直到某个时间点
    void signal(); // 唤醒某个线程
    void signalAll(); // 唤醒全部线程
}
</script></code></pre>



ReentrantLock 中的 Reentrant 是可重入的意思；也就是说 ReentrantLock 是可重入锁，synchronized 也是可重入锁，读写锁类推；

**可重入锁是什么意思**？
可重入锁，也叫做递归锁，指的是同一线程在外层函数获得锁之后，内层递归函数仍然有获取该锁的代码，但不受影响；
在 JAVA 环境下 ReentrantLock、ReentrantReadWriteLock 和 synchronized 都是可重入锁；
可重入可以简单的理解为，只要我获得了锁 A，那么无论我再调用多少次 A.lock() 都不会有问题；

`可重入锁最大的作用是避免死锁`

**synchronized、Lock 性能对比**
在 JDK1.5 中，synchronized 是性能低效的；因为这是一个重量级操作，它对性能最大的影响是阻塞的是实现，挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作给系统的并发性带来了很大的压力；

相比之下使用 Java 提供的 Lock 对象，性能更高一些；Brian Goetz 对这两种锁在 JDK1.5、单核处理器及双 Xeon 处理器环境下做了一组吞吐量对比的实验，发现多线程环境下，synchronized 的吞吐量下降的非常严重，而 ReentrantLock 则能基本保持在同一个比较稳定的水平上；

但与其说 ReetrantLock 性能好，倒不如说 synchronized 还有非常大的优化余地，于是到了 JDK1.6，发生了变化，对 synchronize 加入了很多优化措施，有自适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等；导致在 JDK1.6 上 synchronize 的性能并不比 Lock 差；

官方也表示，他们也更支持 synchronize，在未来的版本中还有优化余地，所以还是提倡在 synchronized 能实现需求的情况下，优先考虑使用 synchronized 来进行同步；

**阻塞同步、非阻塞同步**
互斥同步最主要的问题就是进行线程阻塞和唤醒所带来的性能问题，因而这种同步又称为`阻塞同步`，它属于一种`悲观`的并发策略，即线程获得的是独占锁；独占锁意味着其他线程只能依靠阻塞来等待线程释放锁；而在 CPU 转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起 CPU 频繁的上下文切换导致效率很低；synchronized 采用的便是这种并发策略；

随着指令集的发展，我们有了另一种选择：基于冲突检测的`乐观`并发策略，通俗地讲就是先进性操作，如果没有其他线程争用共享数据，那操作就成功了，如果共享数据被争用，产生了冲突，那就再进行其他的补偿措施（最常见的补偿措施就是不断地重试，直到试成功为止），这种乐观的并发策略的许多实现都不需要把线程挂起，因此这种同步被称为`非阻塞同步`；ReetrantLock 采用的便是这种并发策略；

在乐观的并发策略中，需要操作和冲突检测这两个步骤具备原子性，它靠硬件指令来保证，这里用的是 `CAS操作（Compare and Swap）`；JDK1.5 之后，Java 程序才可以使用 CAS 操作；我们可以进一步研究 ReentrantLock 的源代码，会发现其中比较重要的获得锁的一个方法是 compareAndSetState，这里其实就是调用的 CPU 提供的特殊指令；现代的 CPU 提供了指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而 compareAndSet() 就用这些代替了锁定；这个算法称作`非阻塞算法`，意思是一个线程的失败或者挂起不应该影响其他线程的失败或挂起；

Java 5 中引入了注入 AutomicInteger、AutomicLong、AutomicReference 等特殊的原子性变量类，它们提供的如：compareAndSet()、incrementAndSet() 和 getAndIncrement() 等方法都使用了 CAS 操作；因此，它们都是由硬件指令来保证的原子方法；

**Lock 对象的使用方式**
Lock 接口有 3 个实现它的类：ReentrantLock、ReetrantReadWriteLock.ReadLock 和 ReetrantReadWriteLock.WriteLock，即可重入锁、读锁和写锁；

Lock 必须被显式地创建、锁定和释放，为了可以使用更多的功能，一般用 ReentrantLock 为其实例化；

为了保证锁最终一定会被释放（可能会有异常发生），要把互斥区放在 try 语句块内，并在 finally 语句块中释放锁，尤其当有 return 语句时，return 语句必须放在 try 字句中，以确保 unlock()不会过早发生，从而将数据暴露给第二个任务；

因此，采用 Lock 加锁和释放锁的一般形式如下：
<pre><code class="language-bash line-numbers"><script type="text/plain">
Lock lock = new ReentrantLock(); // 默认使用非公平锁，如果要使用公平锁，需要传入参数 true
lock.lock(); // 获取锁
try {
    // 更新对象的状态
    // 捕获异常，必要时恢复到原来的不变约束
    // 如果有 return 语句，放在这里
} finally {
    lock.unlock(); // 在 finally 块中释放锁(在return之前会执行)
}
</script></code></pre>



**用途比较**
基本语法上，ReentrantLock 与 synchronized 很相似，它们都具备一样的`线程重入特性`，只是代码写法上有点区别而已，一个表现为 API 层面的互斥锁（Lock），一个表现为原生语法层面的互斥锁（synchronized）；ReentrantLock 相对 synchronized 而言还是增加了一些高级功能，主要有以下三项：

1) `等待可中断`：当持有锁的线程长期不释放锁时，正在等待的线程可以选择放弃等待，改为处理其他事情；而在等待由 synchronized 产生的互斥锁时，会一直阻塞，是不能被中断的；
2) `可实现公平锁`：多个线程在等待同一个锁时，必须按照申请锁的时间顺序排队等待，而非公平锁则不保证这点，在锁释放时，任何一个等待锁的线程都有机会获得锁；`synchronized`中的锁是`非公平锁`，`ReentrantLock`默认情况下也是`非公平锁`，但可以通过构造方法`ReentrantLock(ture)`来要求使用`公平锁`；
3) `锁可以绑定多个条件`：ReentrantLock 对象可以同时绑定多个 Condition 对象，而在 synchronized 中，锁对象的 wait() 和 notify()/notifyAll() 方法可以实现一个隐含条件，但如果要和多于一个的条件关联的时候，就不得不额外地添加一个锁，而 ReentrantLock 则无需这么做，只需要多次调用 newCondition() 方法即可；而且我们还可以通过绑定 Condition 对象来判断当前线程通知的是哪些线程（即与 Condition 对象绑定在一起的其他线程）；

**可中断锁**
ReetrantLock 有两种锁：忽略中断锁（不可中断）和响应中断锁（可中断）；忽略中断锁与 synchronized 实现的互斥锁一样，不能响应中断，而响应中断锁可以响应中断；

获得响应中断锁的方法：`lock.lockInterruptibly();`；

synchronized 不可中断的例子：
<pre><code class="language-bash line-numbers"><script type="text/plain">
import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        Object lock = new Object();
        Thread t1 = new Thread(new TaskA(lock));
        Thread t2 = new Thread(new TaskB(lock));

        t1.start();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            out.println(e);
        }
        t2.start();

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            out.println(e);
        }

        out.println("[Main]: B，别等了，A 已经死了，他永远的离开了！");
        t2.interrupt(); // 发送中断请求

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            out.println(e);
        }
    }
}

class TaskA implements Runnable {
    private Object lock;
    public TaskA(Object lock) { this.lock = lock; }
    @Override
    public void run() {
        synchronized (lock) {
            out.println("[A]: 我获得了锁，现在我要开始装逼了！");
            while (true) {
                try {
                    Thread.sleep(Integer.MAX_VALUE); // 先来睡个觉
                } catch (InterruptedException e) {
                    out.println("[A]: 别吵我，继续睡 ...");
                }
            }
            // out.println("[A]: 我是谁，我从哪里来，将到哪里去？！"); // 几乎不可能醒来了
        }
    }
}

class TaskB implements Runnable {
    private Object lock;
    public TaskB(Object lock) { this.lock = lock; }
    @Override
    public void run() {
        out.println("[B]: 我等等吧，到 B 醒来了就好了。。");
        synchronized (lock) {
            out.println("[B]: MD，终于轮到劳资装逼了，哈哈哈哈嗝");
        }
        out.println("[B]: (装逼被打了。。。)");
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [15:11:20] C:130
$ javac Main.java

# root @ arch in ~/work on git:master x [15:11:22]
$ java Main
[A]: 我获得了锁，现在我要开始装逼了！
[B]: 我等等吧，到 B 醒来了就好了。。
[Main]: B，别等了，A 已经死了，他永远的离开了！
^C#
</script></code></pre>



ReentrantLock 可中断锁，例子：
<pre><code class="language-bash line-numbers"><script type="text/plain">
import static java.lang.System.*;
import java.util.concurrent.locks.*;

public class Main {
    public static void main(String[] args) {
        Lock lock = new ReentrantLock();
        Thread t1 = new Thread(new TaskA(lock));
        Thread t2 = new Thread(new TaskB(lock));

        t1.start();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            out.println(e);
        }
        t2.start();

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            out.println(e);
        }

        out.println("[Main]: B，别等了，A 已经死了，他永远的离开了！");
        t2.interrupt(); // 发送中断请求
        t1.interrupt();

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            out.println(e);
        }
    }
}

class TaskA implements Runnable {
    private Lock lock;
    public TaskA(Lock lock) { this.lock = lock; }
    @Override
    public void run() {
        try {
            lock.lockInterruptibly();
            out.println("[A]: 我获得了锁，现在我要开始装逼了！");
            try {
                Thread.sleep(Integer.MAX_VALUE); // 先来睡个觉
            } catch (InterruptedException e) {
                out.println("[B]: (被 Main 埋了。。。)");
            }
            lock.unlock();
        } catch (InterruptedException e) {
            // TODO
        }
    }
}

class TaskB implements Runnable {
    private Lock lock;
    public TaskB(Lock lock) { this.lock = lock; }
    @Override
    public void run() {
        try {
            out.println("[B]: 我等等吧，到 B 醒来了就好了。。");
            lock.lockInterruptibly();
            out.println("[B]: MD，终于轮到劳资装逼了，哈哈哈哈嗝");
            out.println("[B]: (装逼被打了。。。)");
            lock.unlock();
        } catch (InterruptedException e) {
            out.println("[B]: 知道了，劳资不等了");
        }
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [15:34:17]
$ javac Main.java

# root @ arch in ~/work on git:master x [15:34:30]
$ java Main
[A]: 我获得了锁，现在我要开始装逼了！
[B]: 我等等吧，到 B 醒来了就好了。。
[Main]: B，别等了，A 已经死了，他永远的离开了！
[B]: 知道了，劳资不等了
[B]: (被 Main 埋了。。。)
</script></code></pre>



**Condition 条件变量**
Condition 对象需要通过 Lock.newCondition() 方法来获取，Condition 总是和 Lock 一起出现；
一个 Lock 可以有多个不同的 Condition 对象，它们之间互不影响，而 wait()、notify()/notifyAll() 不能；

一个简单的生产者消费者例子：
<pre><code class="language-bash line-numbers"><script type="text/plain">
import static java.lang.System.*;
import java.util.concurrent.locks.*;

public class Main {
    public static void main(String[] args) {
        Godown godown = new Godown();
        Thread t1 = new Thread(new Producer(godown));
        Thread t2 = new Thread(new Consumer(godown));
        t1.start();
        t2.start();
        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            out.println(e);
        }
    }
}

class Godown {
    private Lock lock = new ReentrantLock(); // 互斥锁mutex
    private Condition cond_produce = lock.newCondition(); // 生产者cond
    private Condition cond_consume = lock.newCondition(); // 消费者cond
    private volatile boolean isProduced = false; // 标识产品已生产
    private Product p; // 产品

    public static class Product {
        private int id; // 产品编号
        public Product(int id) {
            this.id = id;
        }
        @Override
        public String toString() {
            return "产品编号: " + id + ", X牌x产品";
        }
    }

    // 生产者 put
    public void put(Product p) {
        lock.lock();

        while (isProduced) { // 如果当前有产品未消费则等待
            try {
                cond_produce.await();
            } catch (InterruptedException e) {
                // TODO
            }
        }

        this.p = p;
        isProduced = true;
        out.printf("[生产者] -> 已生产(%s)\n", p);

        cond_consume.signalAll(); // 通知消费者
        lock.unlock();
    }

    // 消费者 get
    public void get() {
        lock.lock();

        while (!isProduced) { // 没有产品可消费则一直等待
            try {
                cond_consume.await();
            } catch (InterruptedException e) {
                // TODO
            }
        }

        out.printf("[消费者] -> 已消费(%s)\n", p);
        isProduced = false;

        cond_produce.signalAll(); // 通知生产者
        lock.unlock();
    }
}

class Producer implements Runnable {
    private Godown godown;

    public Producer(Godown godown) {
        this.godown = godown;
    }

    @Override
    public void run() {
        for (int i=1; i<=5; i++) {
            godown.put(new Godown.Product(i));
        }
    }
}

class Consumer implements Runnable {
    private Godown godown;

    public Consumer(Godown godown) {
        this.godown = godown;
    }

    @Override
    public void run() {
        for (int i=1; i<=5; i++) {
            godown.get();
        }
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [16:31:02] C:127
$ javac Main.java

# root @ arch in ~/work on git:master x [16:31:05]
$ java Main
[生产者] -> 已生产(产品编号: 1, X牌x产品)
[消费者] -> 已消费(产品编号: 1, X牌x产品)
[生产者] -> 已生产(产品编号: 2, X牌x产品)
[消费者] -> 已消费(产品编号: 2, X牌x产品)
[生产者] -> 已生产(产品编号: 3, X牌x产品)
[消费者] -> 已消费(产品编号: 3, X牌x产品)
[生产者] -> 已生产(产品编号: 4, X牌x产品)
[消费者] -> 已消费(产品编号: 4, X牌x产品)
[生产者] -> 已生产(产品编号: 5, X牌x产品)
[消费者] -> 已消费(产品编号: 5, X牌x产品)
</script></code></pre>



从这个例子中并不能看出用条件变量的 await()、signal()、signalAll() 方法比用 Object 对象的 wait()、notify()、notifyAll() 方法实现线程间协作有多少优点，但它在处理更复杂的多线程问题时，会有明显的优势；所以，Lock 和 Condition 对象只有在更加困难的多线程问题中才是必须的；

**ReadWriteLock 读写锁**
读写锁实际是一种特殊的自旋锁，它把对共享资源的访问者划分成读者和写者，读者只对共享资源进行读访问，写者则需要对共享资源进行写操作；

这种锁相对于自旋锁而言，能提高并发性，因为在多处理器系统中，它允许同时有多个读者来访问共享资源，最大可能的读者数为实际的逻辑 CPU 数；写者是排他性的，一个读写锁同时只能有一个写者或多个读者，但不能同时既有读者又有写者；

如果读写锁当前没有读者，也没有写者，那么写者可以立刻获得读写锁，否则它必须自旋在那里，直到没有任何写者或读者；
如果读写锁没有写者，那么读者可以立即获得该读写锁，否则读者必须自旋在那里，直到写者释放该读写锁；

在 J.U.C 中，java.util.concurrent.locks.ReadWriteLock 接口定义了读写锁：
<pre><code class="language-bash line-numbers"><script type="text/plain">
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
</script></code></pre>



关于读写锁的 Condition，尝试调用通过 readLock() 获取的读锁的 newCondition() 方法将会抛出 UnsupportedOperationException 运行时异常，而 writeLock() 获取的写锁则不会；


## Synchronizers同步器
J.U.C 中的同步器主要用于协助线程同步，有以下四种：
1) 闭锁 CountDownLatch
2) 栅栏 CyclicBarrier
3) 信号量 Semaphore
4) 交换器 Exchanger

> 
这块内容待跟进，笔者现在只了解 Semaphore 信号量，惭愧惭愧！

信号量可以用来限制对某个共享资源进行访问的线程的数量；在对资源进行访问之前，线程必须从得到信号量的许可（调用 Semaphore 对象的 acquire() 方法）；在完成对资源的访问后，线程必须向信号量归还许可（调用 Semaphore 对象的 release() 方法）

> 
在 Linux 的进程间通信（C 语言）里面，我们通常使用共享内存和信号量进行进程间的同步访问；

在 Java 中，Semaphore 也可以作为线程之间同步的手段，不过这并不是信号量的初衷（先不管这么多了，其他例子真的举不出了）
<pre><code class="language-bash line-numbers"><script type="text/plain">
import static java.lang.System.*;
import java.util.concurrent.Semaphore;

public class Main {
    public static void main(String[] args) {
        Semaphore sem = new Semaphore(1); // 一个许可证
        Thread t1 = new Thread(new Task("www.zfl9.com\n", sem));
        Thread t2 = new Thread(new Task("www.baidu.com\n", sem));
        Thread t3 = new Thread(new Task("www.google.com\n", sem));
        t1.start();
        t2.start();
        t3.start();
        try {
            t1.join();
            t2.join();
            t3.join();
        } catch (InterruptedException e) {
            out.println(e);
        }
    }
}

class Task implements Runnable {
    private String str;
    private Semaphore sem;
    public Task(String str, Semaphore sem) {
       this.str = str;
       this.sem = sem;
    }
    @Override
    public void run() {
        try {
            sem.acquire(); // 进入临界区
            for (int i=0; i<str.length(); i++) {
                out.print(str.charAt(i));
            }
            sem.release(); // 离开临界区
        } catch (InterruptedException e) {
            out.println(e);
        }
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [17:48:04]
$ javac Main.java

# root @ arch in ~/work on git:master x [17:48:18]
$ java Main
www.zfl9.com
www.baidu.com
www.google.com
</script></code></pre>


## Atomic原子量
并发中，锁主要提供两种特性：`互斥性`与`可见性`

并发情况下，一个线程先到来，给对象上锁，其他线程只能在后面排队，这种互斥带给我们的是操作的`原子性`，即在并发的情况下，一个操作不会受其他线程的影响；

另外，每个线程在读取变量时，自己都会缓存一份，也就是说，可能下次读取操作便不会经过内存，而是直接取缓存，当然，这在并发条件下，线程共享的变量会有问题是理所当然的，如果每个线程在访问一个变量时，每次都从内存中重新读取值，那么我们认为这个变量对于每个线程来说具有`可见性`；

当然，若你不需要互斥，只需要保证可见性，Java 提供了一种更轻量的语法 volitale，经过 volitale 修饰的变量，可以保证：
1) `可见性`
2) `防止指令重排`

在执行代码时，代码顺序为 A -> B -> C，CPU 为了执行效率，可能会将其顺序打乱，当然，为什么我们在单线程条件下执行却是有序的，原因是指令重排遵循 as-if-serial 语义，即无论如何重排，都不会影响结果，就像如果 C 依赖于 A 和 B，那么总是会在 A 和 B 都执行完后，才会执行 A；

显而易见的是 volitale 并不支持原子性，就像 i++ 的执行过程：
1) 从内存中取出 i
2) 将 i 加 1
3) 再将结果赋值给 i

也就是说，**volatile 可以保证可见性，但是不能保证原子性**；

**Unsafe 类**
正如它的名字一样，Unsafe 是一个非常不安全的类；并发包下 Unsafe 提供以下功能：
1) 直接操作内存地址
2) 恢复与挂起线程
3) 开放汇编、CPU 级别的 CAS 操作

> 
`比较并交换(compare and swap, CAS)`，是原子操作的一种，可用于在多线程编程中实现不被打断的数据交换操作，从而避免多线程同时改写某一数据时由于执行顺序不确定性以及中断的不可预知性产生的数据不一致问题；该操作通过将内存中的值与指定数据进行比较，当数值一样时将内存中的数据替换为新的值；
CAS 操作基于 CPU 提供的原子操作指令实现；对于 Intel X86 处理器，可通过在汇编指令前增加 LOCK 前缀来锁定系统总线，使系统总线在汇编指令执行时无法访问相应的内存地址；而各个编译器根据这个特点实现了各自的原子操作函数；

我们可以认为，JDK 并发包是在 Unsafe 下建立的，它的构造函数是被类加载器封死的，外包下可以通过反射获取，但使用 Unsafe 类，请务必对它十分清楚；

**Atomic 系列类**
Java 从 JDK1.5 开始提供了 java.util.concurrent.atomic 包，方便程序员在多线程环境下，无锁的进行原子操作；
原子变量的底层使用了处理器提供的原子指令（CAS），但是不同的 CPU 架构可能提供的原子指令不一样，也有可能需要某种形式的内部锁，所以该方法不能绝对保证线程不被阻塞；

在 Atomic 包里一共有 12 个类，四种原子更新方式，分别是`原子更新基本类型`，`原子更新数组`，`原子更新引用`和`原子更新字段`；Atomic 包里的类基本都是使用 Unsafe 实现的包装类；

**原子更新基本类型**
用于通过原子的方式更新基本类型，Atomic 包提供了以下三个类：
`AtomicBoolean`：原子更新布尔类型；
`AtomicInteger`：原子更新整型；
`AtomicLong`：原子更新长整型；

AtomicInteger 的常用方法如下：
`int get()`：返回 value 值；
`void set(int newValue)`：设置 newValue 值；
`int intValue()`：返回 value 值；
`long longValue()`：返回 value 值（强制类型转换），下同；
`float floatValue()`：返回 value 值；
`double doubleValue()`：返回 value 值；
`String toString()`：转换为字符串；

`int getAndSet(int newValue)`：返回旧值，并以原子方式设置为 newValue 值；
`boolean compareAndSet(int expect, int update)`：如果 value 等于 expect，则将 value 以原子方式设置为 update 值；

`int incrementAndGet()`：原子方式前自增；
`int decrementAndGet()`：原子方式前自减；
`int getAndIncrement()`：原子方式后自增；
`int getAndDecrement()`：原子方式后自减；

`int getAndAdd(int delta)`：返回 value，并以原子方式将 value 和 delta 相加；
`int addAndGet(int delta)`：以原子方式将 value 和 delta 相加，并返回计算结果；

例子：
<pre><code class="language-bash line-numbers"><script type="text/plain">
import static java.lang.System.*;
import java.util.concurrent.atomic.*;

public class Main {
    public static void main(String[] args) {
        AtomicInteger aInt = new AtomicInteger(10);
        out.printf("get() -> %d\n", aInt.get());
        out.printf("getAndSet(20) -> %d, get() -> %d\n", aInt.getAndSet(20), aInt.get());
        out.printf("compareAndSet(20, 10) -> %b, get() -> %d\n", aInt.compareAndSet(20, 10), aInt.get());
        out.printf("incrementAndGet() -> %d\n", aInt.incrementAndGet()); // 前自增
        out.printf("decrementAndGet() -> %d\n", aInt.decrementAndGet()); // 前自减
        out.printf("getAndIncrement() -> %d\n", aInt.getAndIncrement()); // 后自增
        out.printf("getAndDecrement() -> %d\n", aInt.getAndDecrement()); // 后自减
        out.printf("getAndAdd(10) -> %d\n", aInt.getAndAdd(10));
        out.printf("addAndGet(10) -> %d\n", aInt.addAndGet(10));
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [20:03:28]
$ javac Main.java

# root @ arch in ~/work on git:master x [20:03:39]
$ java Main
get() -> 10
getAndSet(20) -> 10, get() -> 20
compareAndSet(20, 10) -> true, get() -> 10
incrementAndGet() -> 11
decrementAndGet() -> 10
getAndIncrement() -> 10
getAndDecrement() -> 11
getAndAdd(10) -> 10
addAndGet(10) -> 30
</script></code></pre>



**原子更新数组**
通过原子的方式更新数组里的某个元素，Atomic 包提供了以下三个类：
`AtomicIntegerArray`：原子更新整型数组里的元素；
`AtomicLongArray`：原子更新长整型数组里的元素；
`AtomicReferenceArray`：原子更新引用类型数组里的元素；

AtomicIntegerArray 类主要是提供原子的方式更新数组里的整型，其常用方法如下：
`int addAndGet(int i, int delta)`：将索引为 i 的 value 和 delta 相加，并返回结果；
`boolean compareAndSet(int i, int expect, int update)`：如果索引为 i 的 value 等于 expect，则更新其为 update 值；
更多方法请查看`JAVA_HOME/src.zip`中的 Java 源码；

**原子更新引用**
原子更新基本类型的 AtomicInteger，只能更新一个变量，如果要原子的更新多个变量，就需要使用这个原子更新引用类型提供的类；Atomic 包提供了以下三个类：
`AtomicReference`：原子更新引用类型；
`AtomicReferenceFieldUpdater`：原子更新引用类型里的字段；
`AtomicMarkableReference`：原子更新带有标记位的引用类型；可以原子的更新一个布尔类型的标记位和引用类型；构造方法是 AtomicMarkableReference(V initialRef, boolean initialMark)；

**原子更新字段**
如果我们只需要某个类里的某个字段，那么就需要使用原子更新字段类，Atomic 包提供了以下三个类：
`AtomicIntegerFieldUpdater`：原子更新整型的字段的更新器；
`AtomicLongFieldUpdater`：原子更新长整型字段的更新器；
`AtomicStampedReference`：原子更新带有版本号的引用类型；该类将整数值与引用关联起来，可用于原子的更数据和数据的版本号，可以解决使用 CAS 进行原子更新时，可能出现的 ABA 问题；

原子更新字段类都是抽象类，每次使用都时候必须使用静态方法 newUpdater 创建一个更新器；原子更新类的字段的必须使用 public volatile 修饰符；

## 阻塞队列/栈
阻塞队列提供了可阻塞的入队和出对操作，如果队列满了，入队操作将阻塞直到有空间可用，如果队列空了，出队操作将阻塞直到有元素可用；

如果你想避免使用错综复杂的 wait–notify 的语句，BlockingQueue 非常有用；BlockingQueue 可用于解决生产者-消费者问题；

任何有效的生产者-消费者问题解决方案都是通过控制生产者 put() 方法（生产资源）和消费者 take() 方法（消费资源）的调用来实现的，一旦你实现了对方法的阻塞控制，那么你将解决该问题；

Java 通过 BlockingQueue 提供了开箱即用的支持来控制这些方法的调用（一个线程创建资源，另一个消费资源）；
java.util.concurrent 包下的 BlockingQueue 接口是一个线程安全的可用于存取对象的队列；

在 Java 中，主要有以下类型的阻塞队列：
`ArrayBlockingQueue`：一个由`数组结构`组成的`有界`阻塞队列；
`LinkedBlockingQueue`：一个由`链表结构`组成的`有界`阻塞队列；
`LinkedTransferQueue`：一个由`链表结构`组成的`无界`阻塞队列；
`LinkedBlockingDeque`：一个由`链表结构`组成的`有界`阻塞双端队列（通常作为stack使用）；
`SynchronousQueue`：一个不存储元素的`无界`阻塞队列；
`PriorityBlockingQueue`：一个支持`优先级`排序的`无界`阻塞队列；
`DelayQueue`：一个支持`延时`获取元素的`无界`阻塞队列；

> 
在 JDK1.2 之后，Vector、Stack 是不被推荐的；Vector 已经被 ArrayList 替代，Stack 应该选择 ArrayDeque、Deque；

**BlockingQueue 接口**
<pre><code class="language-bash line-numbers"><script type="text/plain">
public interface BlockingQueue<E> extends Queue<E> {
    /**
     * 元素 e 入队
     * 成功返回 true，失败返回 false，并抛出 IllegalStateException 异常
     */
    boolean add(E e);

    /**
     * 元素 e 入队
     * 成功返回 true，失败返回 false
     */
    boolean offer(E e);

    /**
     * 元素 e 入队
     * 若空间不足则阻塞
     */
    void put(E e) throws InterruptedException;

    /**
     * 元素 e 入队
     * 若空间不足则阻塞指定时间
     * 成功返回 true，失败返回 false
     */
    boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;

    /**
     * 元素出队
     * 若队列为空则阻塞
     * 返回出队的元素
     */
    E take() throws InterruptedException;

    /**
     * 元素出队
     * 若队列为空则阻塞指定时间
     * 返回出队的元素
     */
    E poll(long timeout, TimeUnit unit) throws InterruptedException;

    int remainingCapacity();
    boolean remove(Object o);
    public boolean contains(Object o);
    int drainTo(Collection<? super E> c);
    int drainTo(Collection<? super E> c, int maxElements);
}
</script></code></pre>



**BlockingDeque 接口**
<pre><code class="language-bash line-numbers"><script type="text/plain">

</script></code></pre>



例子：
<pre><code class="language-bash line-numbers"><script type="text/plain">
import static java.lang.System.*;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        BlockingQueue<Object> godown = new LinkedBlockingQueue<Object>(20); // 队列长度20
        new Thread(new Producer(godown)).start();
        new Thread(new Producer(godown)).start();
        new Thread(new Producer(godown)).start();
        new Thread(new Consumer(godown)).start();
        new Thread(new Consumer(godown)).start();
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            out.println(e);
        }
        exit(0);
    }
}

class Producer implements Runnable {
    private BlockingQueue<Object> godown; // 仓库

    public Producer(BlockingQueue<Object> godown) {
        this.godown = godown;
    }

    @Override
    public void run() {
        while (true) {
            try {
                godown.put(getResource());
            } catch (InterruptedException e) {
                out.println(e);
            }
            out.printf("[Producer] godown-size: %d\n", godown.size());
        }
    }

    private Object getResource() {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            out.println(e);
        }
        return new Object();
    }
}

class Consumer implements Runnable {
    private BlockingQueue<Object> godown;

    public Consumer(BlockingQueue<Object> godown) {
        this.godown = godown;
    }

    @Override
    public void run() {
        Object obj = null;
        while (true) {
            try {
                obj = godown.take();
            } catch (InterruptedException e) {
                out.println(e);
            }
            useResource(obj);
        }
    }

    private void useResource(Object obj) {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            out.println(e);
        }
        out.printf("Consuming Resource -> %s\n", obj.toString());
    }
}
</script></code></pre>

<pre><code class="language-bash line-numbers"><script type="text/plain">
# root @ arch in ~/work on git:master x [21:01:35]
$ javac Main.java

# root @ arch in ~/work on git:master x [21:01:56]
$ java Main
[Producer] godown-size: 1
[Producer] godown-size: 1
[Producer] godown-size: 1
[Producer] godown-size: 2
[Producer] godown-size: 4
[Producer] godown-size: 3
[Producer] godown-size: 6
[Producer] godown-size: 7
[Producer] godown-size: 6
[Producer] godown-size: 8
[Producer] godown-size: 9
[Producer] godown-size: 10
[Producer] godown-size: 12
[Producer] godown-size: 13
[Producer] godown-size: 12
[Producer] godown-size: 14
[Producer] godown-size: 16
[Producer] godown-size: 15
[Producer] godown-size: 18
[Producer] godown-size: 19
[Producer] godown-size: 18
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@32d7fc0
Consuming Resource -> java.lang.Object@1a000bc0
[Producer] godown-size: 19
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@dea91d5
Consuming Resource -> java.lang.Object@1e3f9c58
[Producer] godown-size: 20
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@9d514b0
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@5b3d01fb
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@4cd79bee
Consuming Resource -> java.lang.Object@7068d303
[Producer] godown-size: 20
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@24ec3ecb
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@6aac98c5
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@7dd177ba
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@8131494
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@5c008c24
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@3b12feb4
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@2cf864a1
Consuming Resource -> java.lang.Object@d54c21e
[Producer] godown-size: 19
[Producer] godown-size: 20
Consuming Resource -> java.lang.Object@69270d93
Consuming Resource -> java.lang.Object@33009c1
[Producer] godown-size: 19
[Producer] godown-size: 20
</script></code></pre>
