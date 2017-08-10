---
title: c语言 - socket编程(四)
date: 2017-08-10 11:21:00
categories:
- c
tags:
- c
keywords: c语言 socket编程 Linux 5种IO模型
---

> 
c语言 - socket编程(四)，Linux中的五种网络IO模型：`阻塞IO`、`非阻塞IO`、`多路复用IO`、`信号驱动式IO`、`异步IO`

<!-- more -->

## IO模型
网络IO的本质是socket的操作，我们以recv为例：

每次调用recv，`数据会先拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间`；

所以说，当一个recv操作发生时，它会经历两个阶段：
- 第一阶段：等待数据准备
- 第二阶段：将数据从内核拷贝到进程中

对于socket流而言：
- 第一阶段：通常涉及等待网络上的数据分组到达，然后被复制到内核的某个缓冲区
- 第二阶段：把数据从内核缓冲区复制到应用进程的缓冲区

**网络IO模型**
- 同步IO(synchronous IO)
 - 阻塞IO(bloking IO)
 - 非阻塞IO(non-blocking IO)
 - 多路复用IO(multiplexing IO)
 - 信号驱动式IO(signal-driven IO)
- 异步IO(asynchronous IO)

> 
由于`信号驱动式IO`实际中并不常用，所以接下来只介绍其他四种IO模型

**阻塞IO**
应用程序调用一个IO函数，如recv：
当socket接收缓冲区中没有数据时，当前进程会被recv阻塞，一直会等待数据的到来；
当socket接收缓冲区中有数据时，recv将数据拷贝到进程空间的这个过程，也是阻塞的；

> 
所以，对于阻塞IO，在这两个阶段都被阻塞了

**非阻塞IO**
将一个套接字设置为非阻塞时，就是告诉内核，当一个请求的IO操作无法立即完成时，不要让我等待，应立即给我返回一个错误，如recv：
当socket接收缓冲区中没有数据时，recv会立即返回一个错误，errno为`EAGAIN`，表示现在没有数据，等下再来吧；
当socket接收缓冲区中有数据时，recv将数据拷贝到进程空间的这个过程，也是阻塞的；

所以对于非阻塞IO，通常采取轮询polling的方式，循环往复的主动询问内核，当前是否有数据了

> 
对于非阻塞IO，第一个阶段不会阻塞，但是第二个阶段依旧是阻塞的

**IO多路复用**
其实这种方式和第二种的非阻塞IO很相似，只不过优点是可以借助这几个特殊的系统调用(`select`、`poll`、`epoll`)，来同时轮询多个socket连接：
当调用select、poll、epoll函数时，如果所监控的socket中有部分socket可读、可写或其他事件发生时，就会返回，将其交给用户进程来处理，这个过程是阻塞的，只不过是因为select、poll、epoll系统调用而阻塞的；
当调用返回后，用户进程再调用recv，将数据从内核拷贝到进程空间中，这个过程也是阻塞的，因recv拷贝数据而阻塞；

实际上这种方式相比第二种还差一些，因为这里面包含了两个系统调用(select/poll/epoll、recv)，而第二种只有一个系统调用recv；
不过IO多路复用的优点是可以处理更多的连接，当连接数大的时候，缺点就被优点给掩盖了

IO多路复用相比`多进程/多线程 + 阻塞IO`的系统开销小，因为系统不需要创建新的进程或线程，也不需要维护多个进程、线程的执行

> 
对于多路复用IO，第一个阶段是因为select、poll、epoll而阻塞的，第二个阶段(实际IO操作)依旧是阻塞的

**信号驱动式IO**
首先我们允许socket进行信号驱动IO，并安装一个信号处理函数，进程继续运行并不阻塞；
当数据准备好时，进程会收到一个SIGIO信号，可以在信号处理函数中调用IO操作函数处理数据；

**异步IO**
对于前面四种IO模型，都是同步的，因为所有的实际IO操作(将数据从内核拷贝到进程空间的这个过程)都是阻塞的；
所谓同步IO，就是必须等待当前的IO操作完成之后，才能执行后面的指令，这个等待的过程中是不能进行其他操作的，也就是说，指令序列都是线性执行的；
而异步IO，当遇到IO操作时，直接把这个IO操作交给别人(内核、新的线程/进程等等)来做，而当前进程并不会因为这个IO操作而阻塞，可以立即执行别的操作，当IO操作完成后，进程会收到完成的通知(回调函数、信号等等)

linux2.6之后的内核提供了AIO库，提供异步IO操作，但是实际上用的比较少，目前有很多流行的开源异步IO库：libevent、libev、libuv等

## 实例
### 迭代模式 + 阻塞IO
所谓的迭代模式，就是用while、for循环来不断接受新连接

server.c
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <stdarg.h>
#include <string.h>
#include <unistd.h>
#include <ctype.h>
#include <errno.h>
#include <signal.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <sys/ioctl.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <fcntl.h>

#define LISTEN_PORT 8080
#define MAX_CONN 1024
#define BUF_SIZE 512

static int listenfd;

void handle_signal(int sig);
void do_service(int connfd);

int main(void){
    signal(SIGHUP, handle_signal);
    signal(SIGINT, handle_signal);
    signal(SIGTERM, handle_signal);

    if((listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        perror("create_listenfd error");
        exit(EXIT_FAILURE);
    }

    int reuseaddr = 1;
    if(setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &reuseaddr, sizeof(reuseaddr)) < 0){
        perror("setsockopt_listenfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(LISTEN_PORT);

    if(bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0){
        perror("bind_listenfd error");
        exit(EXIT_FAILURE);
    }

    if(listen(listenfd, MAX_CONN) < 0){
        perror("listen_listenfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in peeraddr;
    socklen_t peerlen = sizeof(peeraddr);
    int connfd;

    for(;;){
        if((connfd = accept(listenfd, (struct sockaddr *)&peeraddr, &peerlen)) < 0){
            perror("accept_listenfd error");
            continue;
        }

        printf("new conn(%s:%d)\n", inet_ntoa(peeraddr.sin_addr), ntohs(peeraddr.sin_port));

        do_service(connfd);
    }

    return 0;
}

void handle_signal(int sig){
    if(sig == SIGHUP){
        fprintf(stderr, "signal: SIGHUP(%d)", sig);
    }else if(sig == SIGINT){
        fprintf(stderr, "signal: SIGINT(%d)", sig);
    }else if(sig == SIGTERM){
        fprintf(stderr, "signal: SIGTERM(%d)", sig);
    }

    fprintf(stderr, "   close server ... ");
    close(listenfd);
    fprintf(stderr, "done\n");

    exit(EXIT_SUCCESS);
}

void do_service(int connfd){
    char buf[BUF_SIZE];
    int nbuf;

    nbuf = recv(connfd, buf, BUF_SIZE, 0);
    buf[nbuf] = 0;

    printf("recv_msg: %s\n", buf);

    send(connfd, buf, nbuf, 0);

    shutdown(connfd, SHUT_WR);
    close(connfd);
}
</script></code></pre>

client.c
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <stdarg.h>
#include <string.h>
#include <unistd.h>
#include <ctype.h>
#include <errno.h>
#include <signal.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <sys/ioctl.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <fcntl.h>

#define BUF_SIZE 512
#define SERV_ADDR "127.0.0.1"
#define SERV_PORT 8080

int main(int argc, char *argv[]){
    if(argc < 2){
        fprintf(stderr, "usage: %s <MSG>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int sockfd;
    if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        perror("create_sockfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    int ret = inet_pton(AF_INET, SERV_ADDR, &servaddr.sin_addr);
    if(ret < 0){
        perror("inet_pton_servaddr error");
        exit(EXIT_FAILURE);
    }else if(ret == 0){
        fprintf(stderr, "inet_pton_servaddr error: The address format is wrong\n");
        exit(EXIT_FAILURE);
    }
    servaddr.sin_port = htons(SERV_PORT);

    if(connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0){
        perror("connect_sockfd error");
        exit(EXIT_FAILURE);
    }

    char buf[BUF_SIZE];
    int nbuf = strlen(argv[1]);

    send(sockfd, argv[1], nbuf, 0);

    nbuf = recv(sockfd, buf, BUF_SIZE, 0);
    buf[nbuf] = 0;

    printf("echo_msg: %s\n", buf);

    close(sockfd);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/tmp [14:24:56]
$ gcc -o server server.c

# root @ localhost in ~/tmp [14:25:00]
$ gcc -o client client.c

# root @ localhost in ~/tmp [14:25:01]
$ ./server

# root @ localhost in ~/tmp [14:22:58]
$ for ((i=0; i<10; i++)); do ./client 'www.zfl9.com'; done
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com

# root @ localhost in ~/tmp [14:25:01]
$ ./server
new conn(127.0.0.1:58278)
recv_msg: www.zfl9.com
new conn(127.0.0.1:58280)
recv_msg: www.zfl9.com
new conn(127.0.0.1:58282)
recv_msg: www.zfl9.com
new conn(127.0.0.1:58284)
recv_msg: www.zfl9.com
new conn(127.0.0.1:58286)
recv_msg: www.zfl9.com
new conn(127.0.0.1:58288)
recv_msg: www.zfl9.com
new conn(127.0.0.1:58290)
recv_msg: www.zfl9.com
new conn(127.0.0.1:58292)
recv_msg: www.zfl9.com
new conn(127.0.0.1:58294)
recv_msg: www.zfl9.com
new conn(127.0.0.1:58296)
recv_msg: www.zfl9.com
^Csignal: SIGINT(2)   close server ... done
</script></code></pre>

### 多进程 + 阻塞IO
主进程调用accept()不断接收新连接，然后调用fork()创建一个子进程来处理新连接，即：来一个新连接就启动一个新进程

server.c
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <stdarg.h>
#include <string.h>
#include <unistd.h>
#include <ctype.h>
#include <errno.h>
#include <signal.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <sys/ioctl.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <fcntl.h>

#define LISTEN_PORT 8080
#define MAX_CONN 1024
#define BUF_SIZE 512

static int listenfd;

void handle_signal(int sig);
void do_service(int connfd);

int main(void){
    signal(SIGHUP, handle_signal);
    signal(SIGINT, handle_signal);
    signal(SIGTERM, handle_signal);
    signal(SIGCHLD, SIG_IGN);

    if((listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        perror("create_listenfd error");
        exit(EXIT_FAILURE);
    }

    int reuseaddr = 1;
    if(setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &reuseaddr, sizeof(reuseaddr)) < 0){
        perror("setsockopt_listenfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(LISTEN_PORT);

    if(bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0){
        perror("bind_listenfd error");
        exit(EXIT_FAILURE);
    }

    if(listen(listenfd, MAX_CONN) < 0){
        perror("listen_listenfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in peeraddr;
    socklen_t peerlen = sizeof(peeraddr);
    int connfd;
    pid_t pid;

    for(;;){
        if((connfd = accept(listenfd, (struct sockaddr *)&peeraddr, &peerlen)) < 0){
            perror("accept_listenfd error");
            continue;
        }

        printf("new conn(%s:%d)\n", inet_ntoa(peeraddr.sin_addr), ntohs(peeraddr.sin_port));

        pid = fork();
        if(pid < 0){
            perror("fork error");
            close(connfd);
            continue;
        }else if(pid == 0){
            close(listenfd);
            do_service(connfd);
            exit(EXIT_SUCCESS);
        }else{
            close(connfd);
        }
    }

    return 0;
}

void handle_signal(int sig){
    if(sig == SIGHUP){
        fprintf(stderr, "signal: SIGHUP(%d)", sig);
    }else if(sig == SIGINT){
        fprintf(stderr, "signal: SIGINT(%d)", sig);
    }else if(sig == SIGTERM){
        fprintf(stderr, "signal: SIGTERM(%d)", sig);
    }

    fprintf(stderr, "   close server ... ");
    close(listenfd);
    fprintf(stderr, "done\n");

    exit(EXIT_SUCCESS);
}

void do_service(int connfd){
    char buf[BUF_SIZE];
    int nbuf;

    nbuf = recv(connfd, buf, BUF_SIZE, 0);
    buf[nbuf] = 0;

    printf("recv_msg: %s\n", buf);

    send(connfd, buf, nbuf, 0);

    shutdown(connfd, SHUT_WR);
    close(connfd);
}
</script></code></pre>

client.c
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <stdarg.h>
#include <string.h>
#include <unistd.h>
#include <ctype.h>
#include <errno.h>
#include <signal.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <sys/ioctl.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <fcntl.h>

#define BUF_SIZE 512
#define SERV_ADDR "127.0.0.1"
#define SERV_PORT 8080

int main(int argc, char *argv[]){
    if(argc < 2){
        fprintf(stderr, "usage: %s <MSG>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int sockfd;
    if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        perror("create_sockfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    int ret = inet_pton(AF_INET, SERV_ADDR, &servaddr.sin_addr);
    if(ret < 0){
        perror("inet_pton_servaddr error");
        exit(EXIT_FAILURE);
    }else if(ret == 0){
        fprintf(stderr, "inet_pton_servaddr error: The address format is wrong\n");
        exit(EXIT_FAILURE);
    }
    servaddr.sin_port = htons(SERV_PORT);

    if(connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0){
        perror("connect_sockfd error");
        exit(EXIT_FAILURE);
    }

    char buf[BUF_SIZE];
    int nbuf = strlen(argv[1]);

    send(sockfd, argv[1], nbuf, 0);

    nbuf = recv(sockfd, buf, BUF_SIZE, 0);
    buf[nbuf] = 0;

    printf("echo_msg: %s\n", buf);

    close(sockfd);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/tmp [14:46:20]
$ gcc -o server server.c

# root @ localhost in ~/tmp [14:46:41]
$ gcc -o client client.c

# root @ localhost in ~/tmp [14:46:45]
$ ./server

# root @ localhost in ~/tmp [14:44:22]
$ for ((i=0; i<10; i++)); do ./client 'www.zfl9.com'; done
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com
echo_msg: www.zfl9.com

# root @ localhost in ~/tmp [14:46:45]
$ ./server
new conn(127.0.0.1:50474)
recv_msg: www.zfl9.com
new conn(127.0.0.1:50476)
recv_msg: www.zfl9.com
new conn(127.0.0.1:50478)
recv_msg: www.zfl9.com
new conn(127.0.0.1:50480)
recv_msg: www.zfl9.com
new conn(127.0.0.1:50482)
recv_msg: www.zfl9.com
new conn(127.0.0.1:50484)
recv_msg: www.zfl9.com
new conn(127.0.0.1:50486)
recv_msg: www.zfl9.com
new conn(127.0.0.1:50488)
recv_msg: www.zfl9.com
new conn(127.0.0.1:50490)
recv_msg: www.zfl9.com
new conn(127.0.0.1:50492)
recv_msg: www.zfl9.com
^Csignal: SIGINT(2)   close server ... done
</script></code></pre>

### epoll多路复用IO
**基本知识**
epoll是在Linux 2.6内核中提出的，是之前的select和poll的增强版本；
相对于select和poll来说，epoll更加灵活，没有描述符限制；
epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中；

**epoll接口**
epoll操作过程需要三个接口，这三个函数都在头文件`sys/epoll.h`中：

`int epoll_create(int size);`：创建一个epoll文件描述符
- `size`：输入参数，在内核版本 2.6.8 之后，这个参数被弃用了，不过传入的值必须大于0
- 返回值：成功返回epoll实例的文件描述符fd，失败返回-1，并设置errno

`int epoll_create1(int flags);`：创建一个epoll文件描述符(新)
- `flags`：输入参数，如果flags为0，则等价于epoll_create()；
- 返回值：成功返回epoll实例的文件描述符fd，失败返回-1，并设置errno

`int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);`：epoll事件注册、修改、删除
- `epfd`：输入参数，epoll文件描述符
- `op`：输入参数，动作：`EPOLL_CTL_ADD`添加、`EPOLL_CTL_MOD`修改、`EPOLL_CTL_DEL`移除
- `fd`：输入参数，被监听的文件描述符
- `event`：输入参数，监听的epoll事件，events可以是以下几个宏的集合：
`EPOLLIN`：表示对应的文件描述符可以读(包括对端SOCKET正常关闭)；
`EPOLLOUT`：表示对应的文件描述符可以写；
`EPOLLPRI`：表示对应的文件描述符有紧急的数据可读(这里应该表示有带外数据到来)；
`EPOLLERR`：表示对应的文件描述符发生错误；
`EPOLLHUP`：表示对应的文件描述符被挂断；
`EPOLLET`：将EPOLL设为`边缘触发(Edge Triggered)`模式，这是相对于`水平触发(Level Triggered)`来说的；
`EPOLLONESHOT`：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里；
- 返回值：成功返回0，失败返回-1，并设置errno

`int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);`：等待epoll事件的发生
- `epfd`：输入参数，epoll文件描述符
- `events`：输出参数，一个数组，用来保存发生的事件的集合
- `maxevents`：输入参数，events数组的长度
- `timeout`：输入参数，超时时间(单位:毫秒)，-1永久阻塞，0立即返回
- 返回值：成功返回实际发生的事件的数量，返回0表示超时，失败返回-1，并设置errno

**工作模式**
epoll有两种工作模式：`LT(level trigger)`水平触发、`ET(edge trigger)`边缘触发；

默认工作在LT模式，LT模式与ET模式的区别：
- LT模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件；下次调用epoll_wait时，会再次响应应用程序并通知此事件；
LT模式同时支持阻塞、非阻塞的socket套接字；
- ET模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件；如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件；只能等待该描述符的下次事件发生(也就是状态改变的时候)才会通知应用程序；
ET模式只支持非阻塞的socket套接字；

ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高；
epoll工作在ET模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/写操作把处理多个文件描述符的任务饿死；

所以，对于ET模式的epoll，必须在事件触发后，一次性把当前socket缓冲区的数据全部读完，把要发送的数据全部发完才能继续处理下一个事件，同时，对于connect和accept也要进行相应的处理

**LT模式 - 实例**
server.c
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <stdarg.h>
#include <string.h>
#include <unistd.h>
#include <ctype.h>
#include <errno.h>
#include <signal.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <sys/ioctl.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <fcntl.h>

#define LISTEN_PORT 8080
#define MAX_CONN 1024
#define MAX_EVENT 1024
#define BUF_SIZE 512

static int listenfd;
static int epollfd;

void handle_signal(int sig);

int main(void){
    signal(SIGHUP, handle_signal);
    signal(SIGINT, handle_signal);
    signal(SIGTERM, handle_signal);

    if((listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        perror("create_listenfd error");
        exit(EXIT_FAILURE);
    }

    int reuseaddr = 1;
    if(setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &reuseaddr, sizeof(reuseaddr)) < 0){
        perror("setsockopt_listenfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(LISTEN_PORT);

    if(bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0){
        perror("bind_listenfd error");
        exit(EXIT_FAILURE);
    }

    if((epollfd = epoll_create1(0)) < 0){
        perror("create_epollfd error");
        exit(EXIT_FAILURE);
    }

    struct epoll_event event, events[MAX_EVENT];
    event.data.fd = listenfd;
    event.events = EPOLLIN;
    if(epoll_ctl(epollfd, EPOLL_CTL_ADD, listenfd, &event) < 0){
        perror("addevent_epollfd error");
        exit(EXIT_FAILURE);
    }

    if(listen(listenfd, MAX_CONN) < 0){
        perror("listen_listenfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in peeraddr;
    socklen_t peerlen = sizeof(peeraddr);
    char buf[BUF_SIZE];
    int nfds, fd, connfd, nbuf;
    uint32_t ev;

    for(;;){
        nfds = epoll_wait(epollfd, events, MAX_EVENT, -1);
        if(nfds < 0){
            perror("wait_epollfd error");
            continue;
        }

        for(int i=0; i<nfds; i++){
            fd = events[i].data.fd;
            ev = events[i].events;

            if(ev & EPOLLERR || ev & EPOLLHUP || !(ev & EPOLLIN)){
                continue;
            }else if(fd == listenfd && (ev & EPOLLIN)){
                connfd = accept(fd, (struct sockaddr *)&peeraddr, &peerlen);
                if(connfd < 0){
                    perror("accept_listenfd error");
                    continue;
                }

                printf("new conn %s:%d\n", inet_ntoa(peeraddr.sin_addr), ntohs(peeraddr.sin_port));

                struct epoll_event ev;
                ev.data.fd = connfd;
                ev.events = EPOLLIN;
                if(epoll_ctl(epollfd, EPOLL_CTL_ADD, connfd, &ev) < 0){
                    perror("addevent_epollfd error");
                    close(connfd);
                    continue;
                }
            }else if(ev & EPOLLIN){
                nbuf = recv(fd, buf, BUF_SIZE, 0);
                buf[nbuf] = 0;
                printf("recv_msg: %s\n", buf);
                send(fd, buf, nbuf, 0);
                shutdown(fd, SHUT_WR);
                close(fd);
            }
        }
    }

    return 0;
}

void handle_signal(int sig){
    if(sig == SIGHUP){
        fprintf(stderr, "signal: SIGHUP(%d)", sig);
    }else if(sig == SIGINT){
        fprintf(stderr, "signal: SIGINT(%d)", sig);
    }else if(sig == SIGTERM){
        fprintf(stderr, "signal: SIGTERM(%d)", sig);
    }

    fprintf(stderr, "   close server ... ");
    close(listenfd);
    close(epollfd);
    fprintf(stderr, "done\n");

    exit(EXIT_SUCCESS);
}
</script></code></pre>

client.c
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <stdarg.h>
#include <string.h>
#include <unistd.h>
#include <ctype.h>
#include <errno.h>
#include <signal.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <sys/ioctl.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <fcntl.h>

#define BUF_SIZE 512
#define SERV_ADDR "127.0.0.1"
#define SERV_PORT 8080

int main(int argc, char *argv[]){
    if(argc < 2){
        fprintf(stderr, "usage: %s <MSG>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int sockfd;
    if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        perror("create_sockfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    int ret = inet_pton(AF_INET, SERV_ADDR, &servaddr.sin_addr);
    if(ret < 0){
        perror("inet_pton_servaddr error");
        exit(EXIT_FAILURE);
    }else if(ret == 0){
        fprintf(stderr, "inet_pton_servaddr error: The address format is wrong\n");
        exit(EXIT_FAILURE);
    }
    servaddr.sin_port = htons(SERV_PORT);

    if(connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0){
        perror("connect_sockfd error");
        exit(EXIT_FAILURE);
    }

    char buf[BUF_SIZE];
    int nbuf = strlen(argv[1]);

    send(sockfd, argv[1], nbuf, 0);

    nbuf = recv(sockfd, buf, BUF_SIZE, 0);
    buf[nbuf] = 0;

    printf("echo_msg: %s\n", buf);

    close(sockfd);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/tmp [17:29:39]
$ gcc -o server server.c

# root @ localhost in ~/tmp [17:29:59]
$ gcc -o client client.c

# root @ localhost in ~/tmp [17:30:02]
$ ./server

# root @ localhost in ~/tmp [17:28:26]
$ time bash -c 'for((i=0; i<1000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.05s user 1.10s system 103% cpu 1.105 total

# root @ localhost in ~/tmp [17:30:32]
$ time bash -c 'for((i=0; i<1000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.07s user 1.08s system 103% cpu 1.107 total

# root @ localhost in ~/tmp [17:30:34]
$ time bash -c 'for((i=0; i<1000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.05s user 1.10s system 104% cpu 1.105 total

# root @ localhost in ~/tmp [17:30:36]
$ time bash -c 'for((i=0; i<5000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.31s user 5.67s system 105% cpu 5.640 total

# root @ localhost in ~/tmp [17:30:47]
$ time bash -c 'for((i=0; i<5000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.28s user 5.73s system 104% cpu 5.756 total

# root @ localhost in ~/tmp [17:30:54]
$ time bash -c 'for((i=0; i<5000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.31s user 5.80s system 105% cpu 5.783 total

# root @ localhost in ~/tmp [17:30:02]
$ ./server
new conn 127.0.0.1:49288
recv_msg: www.zfl9.com
new conn 127.0.0.1:49290
recv_msg: www.zfl9.com
new conn 127.0.0.1:49292
recv_msg: www.zfl9.com
new conn 127.0.0.1:49294
recv_msg: www.zfl9.com
new conn 127.0.0.1:49296
recv_msg: www.zfl9.com
new conn 127.0.0.1:49298
recv_msg: www.zfl9.com
new conn 127.0.0.1:49300
recv_msg: www.zfl9.com
......
</script></code></pre>

**ET模式 - 实例**
server.c
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <stdarg.h>
#include <string.h>
#include <unistd.h>
#include <ctype.h>
#include <errno.h>
#include <signal.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <sys/ioctl.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <fcntl.h>

#define LISTEN_PORT 8080
#define MAX_CONN 1024
#define MAX_EVENT 1024

static int listenfd;
static int epollfd;

void handle_signal(int sig);

int main(void){
    signal(SIGHUP, handle_signal);
    signal(SIGINT, handle_signal);
    signal(SIGTERM, handle_signal);

    if((listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        perror("create_listenfd error");
        exit(EXIT_FAILURE);
    }

    int reuseaddr = 1;
    if(setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &reuseaddr, sizeof(reuseaddr)) < 0){
        perror("setsockopt_listenfd error");
        exit(EXIT_FAILURE);
    }

    int flgs = fcntl(listenfd, F_GETFL, 0);
    if(fcntl(listenfd, F_SETFL, flgs|O_NONBLOCK) < 0){
        perror("setnonblock_listenfd error");
        exit(EXIT_FAILURE);
    }

    int buf_size;
    socklen_t optlen = sizeof(buf_size);
    if(getsockopt(listenfd, SOL_SOCKET, SO_RCVBUF, &buf_size, &optlen) < 0){
        perror("getsockopt_listenfd error");
        exit(EXIT_FAILURE);
    }
    const int BUF_SIZE = buf_size;

    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(LISTEN_PORT);

    if(bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0){
        perror("bind_listenfd error");
        exit(EXIT_FAILURE);
    }

    if((epollfd = epoll_create1(0)) < 0){
        perror("create_epollfd error");
        exit(EXIT_FAILURE);
    }

    struct epoll_event event, events[MAX_EVENT];
    event.data.fd = listenfd;
    event.events = EPOLLIN | EPOLLET;
    if(epoll_ctl(epollfd, EPOLL_CTL_ADD, listenfd, &event) < 0){
        perror("addevent_epollfd error");
        exit(EXIT_FAILURE);
    }

    if(listen(listenfd, MAX_CONN) < 0){
        perror("listen_listenfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in peeraddr;
    socklen_t peerlen = sizeof(peeraddr);
    char buf[BUF_SIZE];
    int nfds, fd, connfd, nbuf, nbytes;
    uint32_t ev;

    for(;;){
        nfds = epoll_wait(epollfd, events, MAX_EVENT, -1);
        if(nfds < 0){
            perror("wait_epollfd error");
            continue;
        }

        for(int i=0; i<nfds; i++){
            fd = events[i].data.fd;
            ev = events[i].events;

            if(ev & EPOLLERR || ev & EPOLLHUP || !(ev & EPOLLIN)){
                continue;
            }else if(fd == listenfd && (ev & EPOLLIN)){
                for(;;){
                    connfd = accept(fd, (struct sockaddr *)&peeraddr, &peerlen);
                    if(connfd < 0 && errno == EAGAIN){
                        break;
                    }else if(connfd < 0 && (errno == EINTR || errno == ECONNABORTED || errno == EPROTO)){
                        continue;
                    }else if(connfd < 0){
                        perror("accept_listenfd error");
                        continue;
                    }

                    printf("new conn %s:%d\n", inet_ntoa(peeraddr.sin_addr), ntohs(peeraddr.sin_port));

                    flgs = fcntl(connfd, F_GETFL, 0);
                    if(fcntl(connfd, F_SETFL, flgs|O_NONBLOCK) < 0){
                        perror("setnonblock_connfd error");
                        close(connfd);
                        continue;
                    }

                    struct epoll_event ev;
                    ev.data.fd = connfd;
                    ev.events = EPOLLIN | EPOLLET;
                    if(epoll_ctl(epollfd, EPOLL_CTL_ADD, connfd, &ev) < 0){
                        perror("addevent_epollfd error");
                        close(connfd);
                        continue;
                    }
                }
                continue;
            }else if(ev & EPOLLIN){
                nbuf = 0;
                bool can_send = true;

                for(; nbuf < BUF_SIZE;){
                    nbytes = recv(fd, buf+nbuf, BUF_SIZE-nbuf, MSG_DONTWAIT);
                    if(nbytes < 0 && errno == EAGAIN){
                        break;
                    }else if(nbytes < 0 && errno == EINTR){
                        continue;
                    }else if(nbytes < 0){
                        perror("recv_connfd error");
                        shutdown(fd, SHUT_WR);
                        close(fd);
                        can_send = false;
                        break;
                    }else if(nbytes == 0){
                        fprintf(stderr, "peer close conn...\n");
                        shutdown(fd, SHUT_WR);
                        close(fd);
                        can_send = false;
                        break;
                    }else{
                        nbuf += nbytes;
                    }
                }

                if(can_send){
                    int nsent = 0;
                    for(; nsent < nbuf;){
                        nbytes = send(fd, buf+nsent, nbuf-nsent, MSG_DONTWAIT);
                        if(nbytes < 0 && (errno == EAGAIN || errno == EINTR)){
                            continue;
                        }else if(nbytes < 0){
                            perror("send_connfd error");
                            close(fd);
                            break;
                        }else{
                            nsent += nbytes;
                        }
                    }
                }
            }
        }
    }

    return 0;
}

void handle_signal(int sig){
    if(sig == SIGHUP){
        fprintf(stderr, "signal: SIGHUP(%d)", sig);
    }else if(sig == SIGINT){
        fprintf(stderr, "signal: SIGINT(%d)", sig);
    }else if(sig == SIGTERM){
        fprintf(stderr, "signal: SIGTERM(%d)", sig);
    }

    fprintf(stderr, "   close server ... ");
    close(listenfd);
    close(epollfd);
    fprintf(stderr, "done\n");

    exit(EXIT_SUCCESS);
}
</script></code></pre>

client.c
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <stdarg.h>
#include <string.h>
#include <unistd.h>
#include <ctype.h>
#include <errno.h>
#include <signal.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <sys/ioctl.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <fcntl.h>

#define BUF_SIZE 512
#define SERV_ADDR "127.0.0.1"
#define SERV_PORT 8080

int main(int argc, char *argv[]){
    if(argc < 2){
        fprintf(stderr, "usage: %s <MSG>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int sockfd;
    if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        perror("create_sockfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    int ret = inet_pton(AF_INET, SERV_ADDR, &servaddr.sin_addr);
    if(ret < 0){
        perror("inet_pton_servaddr error");
        exit(EXIT_FAILURE);
    }else if(ret == 0){
        fprintf(stderr, "inet_pton_servaddr error: The address format is wrong\n");
        exit(EXIT_FAILURE);
    }
    servaddr.sin_port = htons(SERV_PORT);

    if(connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0){
        perror("connect_sockfd error");
        exit(EXIT_FAILURE);
    }

    char buf[BUF_SIZE];
    int nbuf = strlen(argv[1]);

    send(sockfd, argv[1], nbuf, 0);

    nbuf = recv(sockfd, buf, BUF_SIZE, 0);
    buf[nbuf] = 0;

    printf("echo_msg: %s\n", buf);

    close(sockfd);
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/tmp [19:50:45]
$ gcc -o server server.c

# root @ localhost in ~/tmp [19:51:06]
$ gcc -o client client.c

# root @ localhost in ~/tmp [19:51:09]
$ ./server

# root @ localhost in ~/tmp [19:50:23]
$ time bash -c 'for((i=0; i<1000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.06s user 1.21s system 101% cpu 1.249 total

# root @ localhost in ~/tmp [19:51:22]
$ time bash -c 'for((i=0; i<1000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.09s user 1.20s system 101% cpu 1.275 total

# root @ localhost in ~/tmp [19:51:24]
$ time bash -c 'for((i=0; i<1000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.09s user 1.19s system 102% cpu 1.248 total

# root @ localhost in ~/tmp [19:51:26]
$ time bash -c 'for((i=0; i<5000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.48s user 6.04s system 102% cpu 6.335 total

# root @ localhost in ~/tmp [19:51:37]
$ time bash -c 'for((i=0; i<5000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.47s user 6.16s system 102% cpu 6.502 total

# root @ localhost in ~/tmp [19:51:44]
$ time bash -c 'for((i=0; i<5000; i++)); do ./client 'www.zfl9.com' > /dev/null ; done'
bash -c   0.47s user 6.39s system 102% cpu 6.727 total

# root @ localhost in ~/tmp [19:51:09]
$ ./server
new conn 127.0.0.1:50354
peer close conn...
new conn 127.0.0.1:50356
peer close conn...
new conn 127.0.0.1:50358
peer close conn...
new conn 127.0.0.1:50360
peer close conn...
new conn 127.0.0.1:50362
peer close conn...
new conn 127.0.0.1:50364
peer close conn...
new conn 127.0.0.1:50366
peer close conn...
</script></code></pre>

> 
可以发现，ET模式所耗的时间比LT模式更长一些，不过这是因为echo回声程序的特性导致的；
因为ET模式中的recv/send结果判断、循环，无疑加重了cpu的负担，适得其反；
不过也可能是我的程序逻辑太辣鸡了(这是肯定的了(눈_눈))；
所以说选对模型很重要，并不是所有的情况都适合用ET模式；
