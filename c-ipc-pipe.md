---
title: c语言 - 进程间通信 管道
date: 2017-08-13 14:00:00
categories:
- c
tags:
- c
keywords: c语言 进程间通信 管道 匿名管道 命名管道
---

> 
c语言 - 进程间通信 管道

<!-- more -->

## 匿名管道pipe
如果你使用过Linux的命令，那么对于管道这个名词你一定不会感觉到陌生，因为我们通常通过符号"|"来使用管道；

但是管道的真正定义是什么呢？
管道是一个进程连接数据流到另一个进程的通道，它通常是用作把一个进程的输出通过管道连接到另一个进程的输入；

举个例子，在shell中输入命令：`ls -l | grep string`
我们知道ls命令（其实也是一个进程）会把当前目录中的文件都列出来，但是它不会直接输出，而是把本来要输出到屏幕上的数据通过管道输出到grep这个进程中，作为grep这个进程的输入，然后这个进程对输入的信息进行筛选，把存在string的信息的字符串（以行为单位）打印在屏幕上；

**匿名管道pipe**
`int pipe(filedes[2]);`：创建一个匿名管道
- 头文件：`unistd.h`
- `filedes[2]`：输出参数，用于接收pipe返回的两个文件描述符；`filedes[0]`读管道、`filedes[1]`写管道
- 返回值：成功返回0，失败返回-1，并设置errno

匿名管道实质上是一个`先进先出（FIFO）的队列`：
`filedes[0]`是队头（front），`filedes[1]`是队尾（rear）；

数据从队尾进，从队头出，遵循先进先出的原则；

pipe()创建的管道，其实是一个在内核中的缓冲区，该缓冲区的大小一般为一页，即4K字节；

先来看一个简单的例子：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>

int main(int argc, char *argv[]){
    if(argc < 3){
        fprintf(stderr, "usage: %s parent_sendmsg child_sendmsg\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int pipes[2];
    if(pipe(pipes) < 0){
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    pid_t pid = fork();
    if(pid < 0){
        perror("fork");
        exit(EXIT_FAILURE);
    }else if(pid > 0){
        char buf[BUFSIZ + 1];
        int nbuf;
        strcpy(buf, argv[1]);
        write(pipes[1], buf, strlen(buf));

        sleep(1);

        nbuf = read(pipes[0], buf, BUFSIZ);
        buf[nbuf] = 0;
        printf("parent_proc(%d) recv_from_child: %s\n", getpid(), buf);

        close(pipes[0]);
        close(pipes[1]);
    }else if(pid == 0){
        char buf[BUFSIZ + 1];
        int nbuf = read(pipes[0], buf, BUFSIZ);
        buf[nbuf] = 0;
        printf("child_proc(%d) recv_from_parent: %s\n", getpid(), buf);

        strcpy(buf, argv[2]);
        write(pipes[1], buf, strlen(buf));

        close(pipes[0]);
        close(pipes[1]);
    }

    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [14:44:49]
$ gcc a.c

# root @ arch in ~/work on git:master x [14:44:52]
$ ./a.out from_parent from_child
child_proc(4335) recv_from_parent: from_parent
parent_proc(4334) recv_from_child: from_child
</script></code></pre>

注意到父进程的`sleep(1);`语句：
fork调用之前，父进程创建了一个匿名管道，假设文件描述符为`filedes[] = {3, 4}`，即3为队头，4为队尾；
fork调用之后，创建了一个子进程，子进程也拥有了这两个文件描述符，引用计数都分别加1；

因为实质上在内核中只存在一个管道缓冲区，是父进程创建的，只不过子进程通过fork也拥有了它的引用；
所以，如果父进程发送msg之后，子进程没有及时的读取走数据，那么会被父进程后面的read读取，违背了我们的目的；

所以，一般是不建议上面这种做法的，通常做法是：
一个进程要么往管道里写数据，要么从管道里读数据；
如果既需要读又需要写，那么需要创建两个匿名管道，一个专门读取数据，一个专门写入数据；

比如这样：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>

int main(int argc, char *argv[]){
    if(argc < 3){
        fprintf(stderr, "usage: %s parent_sendmsg child_sendmsg\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int pipes1[2], pipes2[2];
    if(pipe(pipes1) < 0 || pipe(pipes2) < 0){
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    pid_t pid = fork();
    if(pid < 0){
        perror("fork");
        exit(EXIT_FAILURE);
    }else if(pid > 0){
        close(pipes1[0]);
        close(pipes2[1]);

        char buf[BUFSIZ + 1];
        strcpy(buf, argv[1]);
        write(pipes1[1], buf, strlen(buf));

        int nbuf = read(pipes2[0], buf, BUFSIZ);
        buf[nbuf] = 0;
        printf("parent_proc(%d) recv_msg: %s\n", getpid(), buf);

        close(pipes1[1]);
        close(pipes2[0]);
    }else if(pid == 0){
        close(pipes1[1]);
        close(pipes2[0]);

        char buf[BUFSIZ + 1];
        int nbuf = read(pipes1[0], buf, BUFSIZ);
        buf[nbuf] = 0;
        printf("child_proc(%d) recv_msg: %s\n", getpid(), buf);

        strcpy(buf, argv[2]);
        write(pipes2[1], buf, strlen(buf));

        close(pipes1[0]);
        close(pipes2[1]);
    }

    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [15:17:04] C:130
$ gcc a.c

# root @ arch in ~/work on git:master x [15:17:07]
$ ./a.out parent child
child_proc(4622) recv_msg: parent
parent_proc(4621) recv_msg: child
</script></code></pre>

**默认的阻塞模式**
pipe()创建的管道默认是阻塞模式的，阻塞和非阻塞的区别与socket的阻塞、非阻塞很相似：

**管道读写规则**
当没有数据可读时
- `O_NONBLOCK`关闭：read调用阻塞，即进程暂停执行，一直等到有数据来到为止；
- `O_NONBLOCK`打开：read调用返回-1，errno值为EAGAIN；

当管道满的时候
- `O_NONBLOCK`关闭：write调用阻塞，直到有进程读走数据；
- `O_NONBLOCK`打开：调用返回-1，errno值为EAGAIN；

如果所有管道写端对应的文件描述符被关闭，则read返回0；
如果所有管道读端对应的文件描述符被关闭，则write操作会产生信号SIGPIPE；

当要写入的数据量不大于PIPE_BUF时，linux将保证写入的原子性；
当要写入的数据量大于PIPE_BUF时，linux将不再保证写入的原子性；

PIPE_BUF的大小为4096字节，注意，这不是管道的缓冲区大小，这个大小和写入的原子性有关；
所谓原子性：
- 阻塞模式时且`n<PIPE_BUF`：写入具有原子性，如果没有足够的空间供n个字节全部写入，则阻塞直到有足够空间将n个字节全部写入管道；
- 非阻塞模式时且`n<PIPE_BUF`：写入具有原子性，立即全部成功写入，否则一个都不写入，返回错误；
- 阻塞模式时且`n>PIPE_BUF`：不具有原子性，可能中间有其他进程穿插写入，直到将n字节全部写入才返回，否则阻塞等待写入；
- 非阻塞模式时且`n>PIPE_BUF`：不具有原子性，如果管道满的，则立即失败，一个都不写入，返回错误，如果不满，则返回写入的字节数，即部分写入，写入时可能有其他进程穿插写入；

**设置为非阻塞模式**
获取fd的flags值：`int flags = fcntl(sockfd, F_GETFL, 0);`
设置为非阻塞fd：`fcntl(sockfd, F_SETFL, flags|O_NONBLOCK);`
设置为阻塞fd：`fcntl(sockfd, F_SETFL, flags&~O_NONBLOCK);`

## 命名管道fifo
前面介绍的匿名管道中，我们看到了如何使用匿名管道来在进程之间传递数据，同时也看到了这个方式的一个缺陷，就是这些进程都由一个共同的祖先进程启动，这给我们在不相关的的进程之间交换数据带来了不方便；这里将会介绍进程的另一种通信方式：`命名管道`，来解决不相关进程间的通信问题；

**什么是命名管道**
命名管道也被称为`FIFO文件`，它是一种`特殊类型的文件`，它在文件系统中以文件名的形式存在，但是它的行为却和之前所讲的没有名字的管道（匿名管道）类似；

由于Linux中所有的事物都可被视为文件，所以对命名管道的使用也就变得与文件操作非常的统一，也使它的使用非常方便，同时我们也可以像平常的文件名一样在命令中使用；

**创建命名管道**
我们可以使用以下两个函数之一来创建一个命名管道，原型如下：

头文件：`sys/types.h`、`sys/stat.h`
`int mkfifo(const char *filename, mode_t mode);`
`int mknod(const char *filename, mode_t mode | S_IFIFO, (dev_t)0);`
返回值：执行成功返回0，失败返回-1，并设置errno

这两个函数都能创建一个FIFO文件，注意是创建一个真实存在于文件系统中的文件，filename指定了文件名，而mode则指定了文件的读写权限；

或者，可以直接在shell中使用命令`mkfifo`、`mknod`来创建一个FIFO文件；
`mkfifo fifo_file`或`mknod fifo_file p`；

例子：
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/stat.h>

int main(int argc, char *argv[]){
    if(argc < 2){
        fprintf(stderr, "usage: %s fifo_filename\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    if(mkfifo(argv[1], 0644) < 0){
        perror("mkfifo");
        exit(EXIT_FAILURE);
    }

    printf("create file \"%s\" success!\n", argv[1]);

    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:17:33]
$ gcc a.c

# root @ arch in ~/work on git:master x [17:17:34]
$ ./a.out fifo
create file "fifo" success!

# root @ arch in ~/work on git:master x [17:17:38]
$ ll
total 16K
-rw-r--r-- 1 root root  461 Aug 13 17:15 a.c
-rwxr-xr-x 1 root root 8.5K Aug 13 17:17 a.out
prw-r--r-- 1 root root    0 Aug 13 17:17 fifo
</script></code></pre>


**访问命名管道**

**打开FIFO文件**
与打开其他文件一样，FIFO文件也可以使用open调用来打开；注意，mkfifo函数只是创建一个FIFO文件，要使用命名管道还是将其打开；

但是有一点要注意：
不能以`O_RDWR`模式打开FIFO文件进行读写操作，而其行为也未明确定义，因为如一个管道以读/写方式打开，进程就会读回自己的输出，同时我们通常使用FIFO只是为了单向的数据传递；

打开FIFO文件通常有四种方式，
- 头文件：`sys/types.h`、`sys/stat.h`、`fcntl.h`；
- `open(const char *path, O_RDONLY);`：阻塞模式打开，只读模式；
- `open(const char *path, O_RDONLY | O_NONBLOCK);`：非阻塞模式打开，只读模式；
- `open(const char *path, O_WRONLY);`：阻塞模式打开，只写模式；
- `open(const char *path, O_WRONLY | O_NONBLOCK);`：非阻塞模式打开，只写模式；
- 返回值：执行成功返回打开的文件描述符fd，执行失败返回-1，并设置errno

对于以只读方式（O_RDONLY）打开的FIFO文件，如果open调用是阻塞的（即第二个参数为O_RDONLY），除非有一个进程以写方式打开同一个FIFO，否则它不会返回；如果open调用是非阻塞的的（即第二个参数为O_RDONLY | O_NONBLOCK），则即使没有其他进程以写方式打开同一个FIFO文件，open调用将成功并立即返回；

对于以只写方式（O_WRONLY）打开的FIFO文件，如果open调用是阻塞的（即第二个参数为O_WRONLY），open调用将被阻塞，直到有一个进程以只读方式打开同一个FIFO文件为止；如果open调用是非阻塞的（即第二个参数为O_WRONLY | O_NONBLOCK），open总会立即返回，但如果没有其他进程以只读方式打开同一个FIFO文件，open调用将返回-1，并且FIFO也不会被打开；

例子：利用FIFO来在两个非亲缘关系的进程之间传输文件：
`send.c`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main(int argc, char *argv[]){
    if(argc < 3){
        fprintf(stderr, "usage: %s fifo_file filename\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int fifo = open(argv[1], O_WRONLY);
    if(fifo < 0){
        perror("open");
        exit(EXIT_FAILURE);
    }

    FILE *fp = fopen(argv[2], "rb");
    if(fp == NULL){
        perror("fopen");
        exit(EXIT_FAILURE);
    }

    char buf[BUFSIZ];
    int nbuf;
    while((nbuf = fread(buf, 1, BUFSIZ, fp)) > 0){
        write(fifo, buf, nbuf);
    }

    fclose(fp);
    close(fifo);
    return 0;
}
</script></code></pre>

`recv.c`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <errno.h>

int main(int argc, char *argv[]){
    if(argc < 3){
        fprintf(stderr, "usage: %s fifo_file filename\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int fifo = open(argv[1], O_RDONLY);
    if(fifo < 0){
        perror("fifo");
        exit(EXIT_FAILURE);
    }

    FILE *fp = fopen(argv[2], "wb");
    if(fp == NULL){
        perror("fopen");
        exit(EXIT_FAILURE);
    }

    char buf[BUFSIZ];
    int nbuf;
    while((nbuf = read(fifo, buf, BUFSIZ)) > 0){
        fwrite(buf, nbuf, 1, fp);
    }

    close(fifo);
    fclose(fp);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [18:36:19]
$ gcc -o send send.c

# root @ arch in ~/work on git:master x [18:36:22]
$ gcc -o recv recv.c

# root @ arch in ~/work on git:master x [18:36:26]
$ ll
total 32K
pr--r--r-- 1 root root    0 Aug 13 18:34 fifo
-rwxr-xr-x 1 root root 8.8K Aug 13 18:36 recv
-rw-r--r-- 1 root root  727 Aug 13 18:34 recv.c
-rwxr-xr-x 1 root root 8.8K Aug 13 18:36 send
-rw-r--r-- 1 root root  727 Aug 13 18:32 send.c

# root @ arch in ~/work on git:master x [18:36:30]
$ ./send fifo /etc/sysctl.conf

# root @ arch in ~/work on git:master x [18:35:08]
$ ./recv fifo sysctl.conf

# root @ arch in ~/work on git:master x [18:36:48]
$ ll
total 36K
pr--r--r-- 1 root root    0 Aug 13 18:36 fifo
-rwxr-xr-x 1 root root 8.8K Aug 13 18:36 recv
-rw-r--r-- 1 root root  727 Aug 13 18:34 recv.c
-rwxr-xr-x 1 root root 8.8K Aug 13 18:36 send
-rw-r--r-- 1 root root  727 Aug 13 18:32 send.c
-rw-r--r-- 1 root root  803 Aug 13 18:36 sysctl.conf

# root @ arch in ~/work on git:master x [18:36:50]
$ cat sysctl.conf
net.ipv4.ip_forward = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
#net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 3
net.ipv4.ip_local_port_range = 10000 65535
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_max_syn_backlog = 10240
net.core.netdev_max_backlog = 10240
net.core.somaxconn = 10240
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_keepalive_time = 120
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_probes = 3
net.core.rmem_default = 8388608
net.core.wmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 32768 436600 873200
net.ipv4.tcp_wmem = 8192 436600 873200
net.ipv4.tcp_mem = 398458 448266 498073
net.ipv4.tcp_fastopen = 3
fs.file-max = 500000000
</script></code></pre>


**命名管道的安全问题**
前面的例子是两个进程之间的通信问题，也就是说，一个进程向FIFO文件写数据，而另一个进程则在FIFO文件中读取数据；
试想这样一个问题，只使用一个FIFO文件，如果有多个进程同时向同一个FIFO文件写数据，而只有一个读FIFO进程在同一个FIFO文件中读取数据时，会发生怎么样的情况呢？
会发生数据块的相互交错是很正常的，而且个人认为多个不同进程向一个FIFO读进程发送数据是很普通的情况；

为了解决这一问题，就是让写操作的原子化：
FIFO写操作的原子化同pipe()匿名管道，即：每次写入的数据小于等于`PIPE_BUF`的大小，即可保证要么一次性全部写入，要么一个字节也不写入；

**匿名管道与命名管道**
使用匿名管道，通信的进程之间需要一个父子关系，通信的两个进程一定是由一个共同的祖先进程启动；但是匿名管道没有上面说到的数据交叉的问题；

与使用匿名管道相比，我们可以看到send和recv这两个进程是没有什么必然的联系的，如果硬要说他们具有某种联系，就只能说是它们都访问同一个FIFO文件；
它解决了之前在匿名管道中出现的通信的两个进程一定是由一个共同的祖先进程启动的问题；但是为了数据的安全，我们很多时候要采用阻塞的FIFO，让写操作变成原子操作；
