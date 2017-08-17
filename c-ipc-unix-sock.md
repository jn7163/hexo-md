---
title: c语言 - 进程间通信 Unix Domain Socket
date: 2017-08-17 10:40:00
categories:
- c
tags:
- c
keywords: c语言 进程间通信 Unix Domain Socket IPC
---

> 
c语言 - 进程间通信 Unix Domain Socket

<!-- more -->

## Unix Domain Socket
socket API原本是为网络通讯设计的，但是后来在socket的框架上发展出一种IPC机制，就是`UNIX Domain Socket`；

虽然网络socket也可用于同一台主机的进程间通讯（通过loopback地址127.0.0.1），但是UNIX Domain Socket用于IPC更有效率：
- 不需要经过网络协议栈；
- 不需要打包拆包；
- 不需要计算校验和；
- 不需要维护序号和应答；

这是因为IPC机制本质上是可靠的通讯，而网络协议是为不可靠的通讯设计的；
UNIX Domain Socket也提供`面向流`和`面向数据报`两种API接口，类似`TCP`和`UDP`，但是`面向数据报的UNIX Domain Socket`也是可靠的，消息既不会丢失也不会顺序错乱；

使用UNIX Domain Socket的过程和网络socket十分相似，也要先调用socket()创建一个socket文件描述符，address family指定为`AF_UNIX`，type可以选择`SOCK_STREAM`或`SOCK_DGRAM`，protocol参数仍然指定为`0`即可；

UNIX Domain Socket与网络socket编程最明显的不同在于`地址格式不同`，用结构体`sockaddr_un`表示；
网络编程的socket地址是IP地址加端口号，而UNIX Domain Socket的地址是一个`socket类型的文件在文件系统中的路径`，这个socket文件由bind()调用创建，如果调用bind()时该文件已经存在，则bind()错误返回；

## Unix Domain Socket 示例
`server.c`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <ctype.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/un.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <netdb.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <signal.h>
#include <sys/wait.h>

#define SOCK_PATH "/run/echo.sock"
#define BUF_SIZE 1024

int listenfd;
void handle_signal(int signo);

int main(void){
    signal(SIGINT, handle_signal);
    signal(SIGHUP, handle_signal);
    signal(SIGTERM, handle_signal);

    if((listenfd = socket(AF_UNIX, SOCK_STREAM, 0)) < 0){
        perror("socket");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_un servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sun_family = AF_UNIX;
    strcpy(servaddr.sun_path, SOCK_PATH);

    unlink(SOCK_PATH);
    if(bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0){
        perror("bind");
        exit(EXIT_FAILURE);
    }
    chmod(SOCK_PATH, 00640);

    if(listen(listenfd, SOMAXCONN) < 0){
        perror("listen");
        exit(EXIT_FAILURE);
    }

    int connfd, nbuf;
    char buf[BUF_SIZE + 1];
    for(;;){
        if((connfd = accept(listenfd, NULL, NULL)) < 0){
            perror("accept");
            continue;
        }

        nbuf = recv(connfd, buf, BUF_SIZE, 0);
        buf[nbuf] = 0;
        printf("new msg: \"%s\"\n", buf);
        send(connfd, buf, nbuf, 0);

        close(connfd);
    }

    return 0;
}

void handle_signal(int signo){
    if(signo == SIGINT){
        fprintf(stderr, "received signal: SIGINT(%d)\n", signo);
    }else if(signo == SIGHUP){
        fprintf(stderr, "received signal: SIGHUP(%d)\n", signo);
    }else if(signo == SIGTERM){
        fprintf(stderr, "received signal: SIGTERM(%d)\n", signo);
    }

    close(listenfd);
    unlink(SOCK_PATH);
    exit(EXIT_SUCCESS);
}
</script></code></pre>


`client.c`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <ctype.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/un.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <netdb.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <signal.h>
#include <sys/wait.h>

#define SOCK_PATH "/run/echo.sock"
#define BUF_SIZE 1024

int main(int argc, char *argv[]){
    if(argc < 2){
        fprintf(stderr, "usage: %s msg\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int sockfd;
    if((sockfd = socket(AF_UNIX, SOCK_STREAM, 0)) < 0){
        perror("socket");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_un servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sun_family = AF_UNIX;
    strcpy(servaddr.sun_path, SOCK_PATH);

    if(connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0){
        perror("connect");
        exit(EXIT_FAILURE);
    }

    char buf[BUF_SIZE + 1];
    int nbuf;

    nbuf = strlen(argv[1]);
    send(sockfd, argv[1], nbuf, 0);
    nbuf = recv(sockfd, buf, BUF_SIZE, 0);
    buf[nbuf] = 0;
    printf("echo msg: \"%s\"\n", buf);

    close(sockfd);
    return 0;
}
</script></code></pre>


<pre><code class="language-c line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:50:42]
$ gcc -o server server.c

# root @ arch in ~/work on git:master x [10:50:46]
$ gcc -o client client.c

# root @ arch in ~/work on git:master x [10:50:47]
$ ./server

# root @ arch in ~/work on git:master x [10:36:51]
$ for ((i=0; i<10; i++)); do ./client 'www.zfl9.com'; done
echo msg: "www.zfl9.com"
echo msg: "www.zfl9.com"
echo msg: "www.zfl9.com"
echo msg: "www.zfl9.com"
echo msg: "www.zfl9.com"
echo msg: "www.zfl9.com"
echo msg: "www.zfl9.com"
echo msg: "www.zfl9.com"
echo msg: "www.zfl9.com"
echo msg: "www.zfl9.com"

# root @ arch in ~/work on git:master x [10:50:47]
$ ./server
new msg: "www.zfl9.com"
new msg: "www.zfl9.com"
new msg: "www.zfl9.com"
new msg: "www.zfl9.com"
new msg: "www.zfl9.com"
new msg: "www.zfl9.com"
new msg: "www.zfl9.com"
new msg: "www.zfl9.com"
new msg: "www.zfl9.com"
new msg: "www.zfl9.com"
^Creceived signal: SIGINT(2)
</script></code></pre>

## socketpair函数
`socketpair()`函数用于创建一个`全双工的流管道`；

`int socketpair(int domain, int type, int protocol, int sv[2]);`
- 头文件：`sys/types.h`、`sys/socket.h`；
- `domain`：输入参数，协议族（AF_INET、AF_INET6、AF_UNIX），这里我们选择AF_UNIX；
- `type`：输入参数，套接字类型（SOCK_STREAM、SOCK_DGRAM、SOCK_RAW），这里我们选择SOCK_STREAM；
- `protocol`：输入参数，协议类型，一般置为0，让系统选择默认的协议；
- `sv[2]`：输出参数，拥有2个元素的整型数组，返回一个套接字对；
- 返回值：成功返回0，失败返回-1，并设置errno

实际上`socketpair()`跟`pipe()`函数是类似的，也只能在同个主机上具有亲缘关系的进程间通信；
但`pipe()`创建的匿名管道是`半双工`的，而`socketpair()`可以认为是创建一个全双工的管道；
可以使用`socketpair()`创建返回的套接字对进行父子进程通信：

**示例**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <ctype.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/un.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <netdb.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <signal.h>
#include <sys/wait.h>

#define BUF_SIZE 1024

int main(void){
    signal(SIGCHLD, SIG_IGN);

    int sockfds[2];
    if(socketpair(AF_UNIX, SOCK_STREAM, 0, sockfds) < 0){
        perror("socketpair");
        exit(EXIT_FAILURE);
    }
    int parent_fd = sockfds[0], child_fd = sockfds[1];

    pid_t pid = fork();
    if(pid < 0){
        perror("fork");
        exit(EXIT_FAILURE);
    }else if(pid > 0){
        close(child_fd);

        char buf[BUF_SIZE + 1] = "from parent_proc";
        int nbuf = strlen(buf);
        send(parent_fd, buf, nbuf, 0);

        nbuf = recv(parent_fd, buf, BUF_SIZE, 0);
        buf[nbuf] = 0;
        printf("parent_proc(%d) recv_msg: %s\n", getpid(), buf);

        close(parent_fd);
    }else if(pid == 0){
        close(parent_fd);

        char buf[BUF_SIZE + 1] = "from child_proc";
        int nbuf = strlen(buf);
        send(child_fd, buf, nbuf, 0);

        nbuf = recv(child_fd, buf, BUF_SIZE, 0);
        buf[nbuf] = 0;
        printf("child_proc(%d) recv_msg: %s\n", getpid(), buf);

        close(child_fd);
    }

    return 0;
}
</script></code></pre>


<pre><code class="language-c line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [11:02:22]
$ gcc sockpair.c

# root @ arch in ~/work on git:master x [11:02:24]
$ ./a.out
parent_proc(38817) recv_msg: from child_proc
child_proc(38818) recv_msg: from parent_proc
</script></code></pre>
