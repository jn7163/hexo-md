---
title: Java Executor框架入门
date: 2017-09-14 18:13:00
categories:
- java
tags:
- java
keywords: Java Executor框架 Java并发框架 线程池ThreadPool
---

> 
Java Executor框架入门，线程池、阻塞队列、阻塞栈、Callable接口、互斥锁、读写锁、条件变量、信号量等 Java 5 新 API；

<!-- more -->

## Executor框架
在 Java 5 之后，并发编程引入了一堆新的启动、调度和管理线程的 API；

Executor 框架便是 Java 5 中引入的，其内部使用了线程池机制，它在 java.util.cocurrent 包下，通过该框架来控制线程的启动、执行和关闭，可以简化并发编程的操作；

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



前三个构造函数都是调用的第四个构造函数进行初始化操作，各参数的作用：
1) `corePoolSize`：线程池中核心线程数的最大值；
2) `maximumPoolSize`：线程池中能同时拥有的最大线程数；
3) `keepAliveTime`：空闲线程的存活时间；
4) `unit`：keepAliveTime 的时间单位，`TimeUnit.NANOSECONDS`纳秒、`TimeUnit.MICROSECONDS`微秒、`TimeUnit.MILLISECONDS`毫秒、`TimeUnit.SECONDS`秒、`TimeUnit.MINUTES`分、`TimeUnit.HOURS`时、`TimeUnit.DAYS`天；
5) `workQueue`：缓存任务的阻塞队列；
6) `threadFactory`：创建线程的工厂；
7) `handler`：当 workQueue 已满，且线程数达 maximumPoolSize 时，线程池拒绝新任务采取的策略；
