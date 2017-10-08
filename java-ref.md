---
title: Java 4种引用类型
date: 2017-10-08 14:28:00
categories:
- java
tags:
- java
keywords: Java 4种引用类型、强引用、软引用、弱引用、虚引用
---

> 
Java 4种引用类型，`强引用(StrongReference)`、`软引用(SoftReference)`、`弱引用(WeakReference)`、`虚引用(PhantomReference)`

<!-- more -->

## 四种引用
java.lang.ref 是 Java 类库中比较特殊的一个包，它提供了与 Java 垃圾回收器密切相关的引用类；
StrongReference（强引用），SoftReference（软引用），WeakReference（弱引用），PhantomReference（虚引用），这四种引用的强度按照出现的顺序依次减弱。

1) `强引用（StrongReference）`
一般情况下我们使用的都是强引用，如`Object obj = new Object();`的 obj 就是一个强引用；
**当内存不足，JVM 宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会回收强引用对象来释放内存**

2) `软引用（SoftReference）`
**GC 发现了只具有软引用的对象并不会立即进行回收，而是让它活的尽可能久一些，在内存不足前再进行回收**

3) `弱引用（WeakReference）`
**GC 一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存**

4) `虚引用（PhantomReference）`
**虚引用主要用来跟踪对象被 GC 回收的活动，虚引用必须和`引用队列（ReferenceQueue）`联合使用**
“虚引用”顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期；如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。
当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。
> 
除了强引用之外，其他三种引用都在 java.lang.ref 包中。

**引用队列（ReferenceQueue）**
如果我们在创建一个引用对象时，指定了 ReferenceQueue，那么当引用对象指向的对象达到合适的状态（根据引用类型不同而不同）时，GC 会把引用对象本身添加到这个队列中，方便我们处理它；
> 
因为引用对象指向的对象 GC 会自动清理，但是引用对象本身也是对象（是对象就占用一定资源），所以需要我们自己清理。

举个例子：`SoftReference<String> ss = new SoftReference<String>("abc", queue);`
其中，ss 是一个引用对象（一个普通的对象，本身是强引用），指向"abc"对象（该对象只有一个软引用 ss 指向它）；
"abc"对象会在一定时机被 GC 自动清理，但是 ss 对象本身的清理工作依赖于 queue，当 ss 出现在 queue 中时，说明其指向的对象已经无效，可以放心清理 ss 了。

> 
抛开虚引用，我们只需要了解常用的**软引用**、**弱引用**就足以应付一般的问题了。

**引用对象的四种状态**
每一时刻，Reference 对象都处于下面四种状态中：
1) `Active`：”活动状态”，新创建的引用对象都是这个状态，GC 会根据引用对象是否在创建时指定 ReferenceQueue 参数进行状态转移，如果指定了，那么转移到 Pending 状态，如果没指定，转移到 Inactive 状态；
2) `Pending`：”待定状态”，该状态的引用对象等着被内部线程 ReferenceHandler 处理（会调用 ReferenceQueue.enqueue 方法）
3) `Enqueued`：”入队状态”，调用 ReferenceQueue.enqueued 方法后的引用对象处于这个状态中；
4) `Inactive`：”死亡状态”，处于该状态的引用对象将被 GC 自动清理。
> 
如果构造函数中指定了 ReferenceQueue 参数，那么引用对象需要手动进行清理；
如果构造函数中没有指定 ReferenceQueue 参数，那么 GC 会自动清理引用对象。

**软引用、弱引用的应用场景**
软引用、弱引用基本上没有本质上的区别，仅仅是软引用对象比弱引用对象的命更长一些罢了。
因此，软引用、弱引用都适合用来实现**内存敏感的高速缓存**，具体使用哪种引用，这里给几条参考：
1) 如果只是想避免 OutOfMemory 的发生，则可以使用软引用；如果对于性能更在意，想尽快回收一些占用内存比较大的对象，则可以使用弱引用；
2) 如果对象可能会经常使用的，就尽量用软引用；如果对象不被使用的可能性更大些，就可以用弱引用；

**java.lang.ref 包**
1) `Reference 抽象类`：除强引用外的所有引用类型的父类；
2) `SoftReference 类`：软引用；
3) `WeakReference 类`：弱引用；
4) `PhantomReference 类`：虚引用；
5) `ReferenceQueue 类`：引用队列；

**ReferenceQueue**
<pre><code class="language-java line-numbers"><script type="text/plain">public class ReferenceQueue<T> { // 类型参数 T 表示所引用的对象类型
    public ReferenceQueue() {}; // 构造函数
    public Reference<? extends T> poll(); // 无阻塞，弹出可用的引用对象，没有则返回 null
    // 超时等待，单位为毫秒，如果参数 timeout 为 0，则永久等待，弹出可用的引用对象
    public Reference<? extends T> remove(long timeout) throws InterruptedException;
    public Reference<? extends T> remove() throws InterruptedException; // timeout = 0
}
</script></code></pre>



**Reference**
<pre><code class="language-java line-numbers"><script type="text/plain">public abstract class Reference<T> {
    public T get(); // 获取所引用的对象，如果被 gc 回收则返回 null
    public boolean isEnqueued(); // 查看当前引用对象是否已入队
    public void clear(); // [不应该直接调用] 清除所引用的对象，调用此方法并不会让当前引用对象入队
    public boolean enqueue(); // [不应该直接调用] 将当前引用对象加入指定的引用队列(如果存在的话)
}
</script></code></pre>



**SoftReference**
<pre><code class="language-java line-numbers"><script type="text/plain">public class SoftReference<T> extends Reference<T> {
    public SoftReference(T referent); // referent 为被引用对象
    public SoftReference(T referent, ReferenceQueue<? super T> q); // 指定引用队列

    public T get(); // 获取所引用的对象
}
</script></code></pre>



**WeakReference**
<pre><code class="language-java line-numbers"><script type="text/plain">public class WeakReference<T> extends Reference<T> {
    public WeakReference(T referent); // referent 为被引用对象
    public WeakReference(T referent, ReferenceQueue<? super T> q); // 指定引用队列
}
</script></code></pre>



**PhantomReference**
<pre><code class="language-java line-numbers"><script type="text/plain">public class PhantomReference<T> extends Reference<T> {
    public PhantomReference(T referent, ReferenceQueue<? super T> q); // 指定引用队列
    public T get(); // 获取所引用的对象
}
</script></code></pre>



例子：
<pre><code class="language-java line-numbers"><script type="text/plain">import java.lang.ref.SoftReference;
import java.lang.ref.WeakReference;

public class Main {
    private static final int MB = 1024 * 1024;

    public static void main(String[] args) {
        SoftReference<byte[]> sref1 = new SoftReference<>(new byte[10 * MB]);
        System.out.println(sref1.get());

        SoftReference<byte[]> sref2 = new SoftReference<>(new byte[10 * MB]);
        System.out.println(sref1.get()); // null
        System.out.println(sref2.get());

        WeakReference<Object> wref = new WeakReference<>(new Object());
        System.out.println(wref.get());
        System.gc(); // 手动调用 gc
        System.out.println(wref.get()); // null
    }
}
</script></code></pre>

<pre><code class="language-java line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:42:12]
$ javac Main.java

# root @ arch in ~/work on git:master x [16:42:14]
$ java -Xmx15m Main
[B@15db9742
null
[B@6d06d69c
java.lang.Object@7852e922
null
</script></code></pre>
