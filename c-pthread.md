---
title: c语言 - 多线程编程
date: 2017-08-19 09:15:00
categories:
- c
tags:
- c
keywords: c语言 多线程编程 pthread多线程
---

> 
c语言 - 多线程编程

<!-- more -->

## 进程与线程
`进程是程序执行时的一个实例，即它是程序已经执行到何种程度的数据结构的汇集`；从内核的观点看，进程的目的就是担当分配系统资源（CPU时间、内存等）的基本单位；

`线程是进程的一个执行流，是CPU调度和分派的基本单位，它是比进程更小的能独立运行的基本单位`；一个进程由几个线程组成（拥有很多相对独立的执行流的用户程序共享应用程序的大部分数据结构），线程与同属一个进程的其他的线程共享进程所拥有的全部资源；

`"进程——资源分配的最小单位，线程——程序执行的最小单位"`

进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响；
而线程只是一个进程中的不同执行路径线程有自己的堆栈和局部变量，但线程没有单独的地址空间，一个线程死掉就等于整个进程死掉；
所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些，但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程；

## pthread线程库
头文件：`pthread.h`，gcc链接时参数：`-lpthread`；

**线程基本函数**
`int pthread_create(pthread_t *tid, const pthread_attr_t *attr, void *(*func)(void *), void *arg);`：创建线程
- `tid`：输出参数，保存返回的线程ID（与linux系统中的线程ID不一样，这个ID应该理解为一个地址），用无符号长整型表示；
- `attr`：输入参数，线程的相关属性，如线程优先级、初始栈大小、是否为守护进程等，一般置为NULL，表示使用默认属性；
- `func`：输入参数，一个函数指针（`void *job(void *arg);`），线程执行的函数；
- `arg`：输入参数，函数的参数，如果有多个参数须将其封装为一个结构体；
- 返回值：成功返回0，失败返回errno值（正数）；

`void pthread_exit(void *status);`：退出线程
- `status`：输入参数，退出状态；

`int pthread_join(pthread_t tid, void **status);`：等待线程退出
- `tid`：输入参数，指定等待的线程ID；
- `status`：输出参数，一个二级指针，保存退出值，可为NULL；
- 返回值：成功返回0，失败返回errno值；

`pthread_t pthread_self(void);`：获取当前线程ID

`int pthread_detach (pthread_t tid);`：分离线程
- 变为分离状态的线程，如果线程退出，它的所有资源将全部释放；而如果不是分离状态，线程必须保留它的线程ID，退出状态直到其它线程对它调用了pthread_join；
- `tid`：输入参数，指定的线程ID；
- 返回值：成功返回0，失败返回errno值；

**线程ID**
主线程：每个进程至少有一个线程，即main()函数的执行线程，称之为主线程；
子线程：由主线程调用`pthread_create()`创建的线程；

线程不像进程，一个进程中的线程之间是没有父子之分的，都是`平级关系`；即线程都是一样的, 退出了一个不会影响另外一个；

但是所谓的`主线程main`，其入口代码是类似这样的方式调用main的：`exit(main(...))`；
main执行完之后, 会调用exit()，exit()会让整个进程终止，那所有线程自然都会退出；

主线程先退出，子线程继续运行的方法：
在主线程main中调用pthread_exit()，只会使主线程退出；而如果是return，编译器将使其调用进程退出的代码（如_exit()），从而导致进程及其所有线程结束运行；

按照POSIX标准定义，当主线程在子线程终止之前调用pthread_exit()时，子线程是不会退出的；

系统中的线程ID：
- `ls /proc/[PID]/task/[TID]/`：可查看一个进程下的所有线程ID、及相关信息；
- `ps -eo user,pid,ppid,lwp,nlwp,%cpu,%mem,stat,cmd`：lwp即线程ID，nlwp为进程中的线程数量；

主线程的线程ID与它所属进程的进程ID相同；

注意：这里的线程ID与`pthread_self`中的线程ID不是一个概念：
`gettid`获取的是`内核中线程ID`，而`pthread_self`获取的是`posix描述的线程ID`；

在c语言中，可以用`syscall(__NR_gettid);`（头文件`sys/syscall.h`）来获取内核中的线程ID；

**互斥锁**
就像共享内存中的信号量一样，为了防止多个线程同时使用一个共享的对象（如全局变量），pthread提供了互斥锁这种机制；

**初始化**
静态初始化：`static pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;`
动态初始化：`int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);`
- `mutex`：输出参数，互斥变量；
- `attr`：输入参数，锁属性，NULL值为默认属性；
- 返回值：成功返回0，失败返回errno值；

**加锁、释放锁**
`int pthread_mutex_lock(pthread_mutex_t *mutex);`：加锁（阻塞）
- `mutex`：输入参数，互斥变量；
- 返回值：成功返回0，失败返回errno值；

`int pthread_mutex_trylock(pthread_mutex_t *mutex);`：尝试加锁（非阻塞）
- `mutex`：输入参数，互斥变量；
- 返回值：成功返回0，锁繁忙返回`EBUSY`，失败返回errno值；

`int pthread_mutex_unlock(pthread_mutex_t *mutex);`：释放锁
- `mutex`：输入参数，互斥变量；
- 返回值：成功返回0，失败返回errno值；

**销毁**
`int pthread_mutex_destroy(pthread_mutex_t *mutex);`
- `mutex`：输入参数，互斥变量；
- 返回值：成功返回0，失败返回errno值；

**条件变量**
与互斥锁不同，**条件变量是用来等待而不是用来上锁的，条件变量用来自动阻塞一个线程，直到某特殊情况发生为止；通常条件变量和互斥锁同时使用**

条件变量使我们可以睡眠等待某种条件出现；条件变量是利用线程间共享的全局变量进行同步的一种机制，主要包括两个动作：一个线程等待"条件变量的条件成立"而挂起；另一个线程使"条件成立"（给出条件成立信号）；

条件的检测是在互斥锁的保护下进行的；如果一个条件为假，一个线程自动阻塞，并释放等待状态改变的互斥锁；
如果另一个线程改变了条件，它发信号给关联的条件变量，唤醒一个或多个等待它的线程，重新获得互斥锁，重新评价条件；
如果两进程共享可读写的内存，条件变量可以被用来实现这两进程间的线程同步；

**相关函数**
`int pthread_cond_init(pthread_cond_t *cond, pthread_condattr_t *cond_attr);`：初始化
`int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);`：等待条件，阻塞
`int pthread_cond_timedwait(pthread_cond_t *cond, pthread_mutex *mutex, const timespec *abstime);`：等待条件，超时
`int pthread_cond_signal(pthread_cond_t *cond);`：通知条件，只唤醒单个等待线程
`int pthread_cond_broadcast(pthread_cond_t *cond);`：通知条件，唤醒所有等待线程
`int pthread_cond_destroy(pthread_cond_t *cond);`：销毁
返回值：成功返回0，失败返回errno值；

**pthread_cond_wait执行流程**
![pthread_cond_wait执行流程](/images/pthread_cond_wait.jpg)

传入给`pthread_cond_wait`的mutex应为一把已经获取的互斥锁；
pthread_cond_wait调用相当复杂，它是如下执行序列的一个组合：
1）`释放互斥锁` 并且 `将线程挂起`（这两个操作是一个`原子操作`）；
2）线程`获得信号`，`尝试获得互斥锁后被唤醒`；

## 多线程实例
**题目**
1）有一int型全局变量flag初始值为0；
2）在主线程中启动线程1，将flag设置为1；
3）在主线程中启动线程2，将flag设置为2；
4）主线程main一直阻塞，直到1变为2，或2变为1时才会继续运行；

**解决**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/syscall.h>
#include <pthread.h>

#define gettid() syscall(__NR_gettid)

static int flag = 0;

static pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
static pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

void *job1(void *arg);
void *job2(void *arg);

int main(void){
    printf("++++++++++ entry thread_main (pid: %d, tid: %ld) ++++++++++\n", getpid(), gettid());

    pthread_t tid1, tid2;
    errno = pthread_create(&tid1, NULL, job1, NULL);
    if(errno){
        perror("pthread_create");
        exit(EXIT_FAILURE);
    }
    errno = pthread_create(&tid2, NULL, job2, NULL);
    if(errno){
        perror("pthread_create");
        exit(EXIT_FAILURE);
    }

    printf("<thread_main> waiting for 1->2 or 2->1\n");
    errno = pthread_mutex_lock(&mutex);
    if(errno){
        perror("pthread_mutex_lock");
        exit(EXIT_FAILURE);
    }
    errno = pthread_cond_wait(&cond, &mutex);
    if(errno){
        perror("pthread_cond_wait");
        exit(EXIT_FAILURE);
    }
    errno = pthread_mutex_unlock(&mutex);
    if(errno){
        perror("pthread_mutex_unlock");
        exit(EXIT_FAILURE);
    }
    printf("<thread_main> wait finish\n");

    errno = pthread_join(tid1, NULL);
    if(errno){
        perror("pthread_join");
        exit(EXIT_FAILURE);
    }
    errno = pthread_join(tid2, NULL);
    if(errno){
        perror("pthread_join");
        exit(EXIT_FAILURE);
    }

    errno = pthread_cond_destroy(&cond);
    if(errno){
        perror("pthread_cond_destroy");
        exit(EXIT_FAILURE);
    }
    errno = pthread_mutex_destroy(&mutex);
    if(errno){
        perror("pthread_mutex_destroy");
        exit(EXIT_FAILURE);
    }

    printf("---------- leave thread_main (pid: %d, tid: %ld) ----------\n", getpid(), gettid());
    return 0;
}

void *job1(void *arg){
    printf("++++++++++ entry thread_1 (pid: %d, tid: %ld) ++++++++++\n", getpid(), gettid());

    usleep(500);

    errno = pthread_mutex_lock(&mutex);
    if(errno){
        perror("pthread_mutex_lock");
        exit(EXIT_FAILURE);
    }

    printf("<thread_1> before: %d\n", flag);
    if(flag == 2){
        errno = pthread_cond_signal(&cond);
        if(errno){
            perror("pthread_cond_signal");
            exit(EXIT_FAILURE);
        }
    }
    flag = 1;
    printf("<thread_1> after: %d\n", flag);

    errno = pthread_mutex_unlock(&mutex);
    if(errno){
        perror("pthread_mutex_unlock");
        exit(EXIT_FAILURE);
    }

    printf("---------- leave thread_1 (pid: %d, tid: %ld) ----------\n", getpid(), gettid());
    return NULL;
}

void *job2(void *arg){
    printf("++++++++++ entry thread_2 (pid: %d, tid: %ld) ++++++++++\n", getpid(), gettid());

    usleep(500);

    errno = pthread_mutex_lock(&mutex);
    if(errno){
        perror("pthread_mutex_lock");
        exit(EXIT_FAILURE);
    }

    printf("<thread_2> before: %d\n", flag);
    if(flag == 1){
        errno = pthread_cond_signal(&cond);
        if(errno){
            perror("pthread_cond_signal");
            exit(EXIT_FAILURE);
        }
    }
    flag = 2;
    printf("<thread_2> after: %d\n", flag);

    errno = pthread_mutex_unlock(&mutex);
    if(errno){
        perror("pthread_mutex_unlock");
        exit(EXIT_FAILURE);
    }

    printf("---------- leave thread_2 (pid: %d, tid: %ld) ----------\n", getpid(), gettid());
    return NULL;
}
</script></code></pre>


<pre><code class="language-c line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [13:25:47]
$ gcc a.c -lpthread
a.c: In function ‘job1’:
a.c:79:18: warning: unused parameter ‘arg’ [-Wunused-parameter]
 void *job1(void *arg){
                  ^~~
a.c: In function ‘job2’:
a.c:111:18: warning: unused parameter ‘arg’ [-Wunused-parameter]
 void *job2(void *arg){
                  ^~~

# root @ arch in ~/work on git:master x [13:25:53]
$ ./a.out
++++++++++ entry thread_main (pid: 88631, tid: 88631) ++++++++++
++++++++++ entry thread_1 (pid: 88631, tid: 88632) ++++++++++
<thread_main> waiting for 1->2 or 2->1
++++++++++ entry thread_2 (pid: 88631, tid: 88633) ++++++++++
<thread_2> before: 0
<thread_2> after: 2
---------- leave thread_2 (pid: 88631, tid: 88633) ----------
<thread_1> before: 2
<thread_1> after: 1
---------- leave thread_1 (pid: 88631, tid: 88632) ----------
<thread_main> wait finish
---------- leave thread_main (pid: 88631, tid: 88631) ----------
</script></code></pre>
