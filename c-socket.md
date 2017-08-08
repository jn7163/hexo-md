---
title: c语言 - socket编程
date: 2017-08-08 10:42:00
categories:
- c
tags:
- c
keywords: c语言 socket编程 linux socket编程 套接字
---

> 
c语言 - socket编程，源IP地址和目的IP地址以及源端口号和目的端口号的组合称为套接字，其用于标识客户端请求的服务器和服务；
它是网络通信过程中端点的抽象表示，包含进行网络通信必需的五种信息：`连接使用的协议`，`本地主机的IP地址`，`本地进程的协议端口`，`远地主机的IP地址`，`远地进程的协议端口`

<!-- more -->

## socket套接字
**网络主机之间的应用程序如何进行通信？**
网络层的`ip地址`可以唯一标识网络中的主机；
而传输层的`协议+端口`可以唯一标识主机中的应用程序(进程)；
这样利用`三元组(ip地址 + 协议 + 端口)`就可以`标识网络的进程`了，网络中的进程通信就可以利用这个标志与其它进程进行交互

**什么是socket套接字？**
上面我们已经知道网络中的进程是通过socket来通信的，那什么是socket呢？
socket起源于Unix，而Unix/Linux基本哲学之一就是`一切皆文件`，都可以用`打开open –> 读写read/write –> 关闭close`模式来操作；
socket就是该模式的一个实现，socket即是一种特殊的文件，一些socket函数就是对其进行的操作(读/写IO、打开、关闭)；

**`SOCK_STREAM`**：流套接字
流套接字用于提供`面向连接`、`可靠的数据传输`服务；该服务将保证数据能够实现无差错、无重复发送，并按顺序接收；
流套接字之所以能够实现可靠的数据服务，原因在于其使用了传输控制协议，即`TCP协议`

**`SOCK_DGRAM`**：数据报套接字
数据报套接字提供了一种`无连接`的服务；该服务`并不能保证数据传输的可靠性`，数据有可能在传输过程中丢失或出现数据重复，且无法保证顺序地接收到数据；
数据报套接字使用`UDP协议`进行数据的传输；由于数据报套接字不能保证数据传输的可靠性，对于有可能出现的数据丢失情况，需要在程序中做相应的处理

**`SOCK_RAW`**：原始套接字
原始套接字允许`对较低层次的协议直接访问`，比如`IP`、`ICMP`协议，它常用于检验新的协议实现，或者访问现有服务中配置的新设备；
因为RAW SOCKET可以自如地控制网络底层的传输机制，所以可以应用原始套接字来操纵网络层和传输层应用；
比如，我们可以通过RAW SOCKET来接收发向本机的ICMP、IGMP协议包，或者接收TCP/IP栈不能够处理的IP包，也可以用来发送一些自定包头或自定协议的IP包；
原始套接字与标准套接字(标准套接字指的是前面介绍的流套接字和数据包套接字)的区别在于：
原始套接字可以读写内核没有处理的IP数据包，而流套接字只能读取TCP协议的数据，数据报套接字只能读取UDP协议的数据；因此，如果要访问其他协议发送数据必须使用原始套接字

## socket基本操作
`int socket(int domain, int type, int protocol);`：创建一个socket文件描述符fd
- `domain`：输入参数，协议域、地址域或`协议族`；
常用的协议族有：`AF_INET`、`AF_INET6`、`AF_LOCAL`(或称`AF_UNIX`)、`AF_ROUTE`等等
协议族决定了socket的地址类型，在通信中必须采用对应的地址，如AF_INET决定了要用ipv4地址(32位)与端口号(16位)的组合、AF_UNIX决定了要用一个绝对路径名作为地址
- `type`：输入参数，socket类型；
常用的socket类型有：`SOCK_STREAM`、`SOCK_DGRAM`、`SOCK_RAW`、`SOCK_PACKET`、`SOCK_SEQPACKET`等等
- `protocol`：输入参数，指定使用的协议；
常用的协议有：`IPPROTO_TCP`、`IPPTOTO_UDP`、`IPPROTO_SCTP`、`IPPROTO_TIPC`等等
如果该参数为0，则让系统自动选择合适的协议，一般我们也这么操作
- 返回值：成功返回一个非负整数的fd，失败则返回-1，并设置errno

当我们调用socket创建一个socket时，返回的socket描述字它存在于协议族空间中，但没有一个具体的地址；
如果想要给它赋值一个地址，就必须调用`bind()`函数，否则就当调用`connect()`、`listen()`时系统会自动随机分配一个端口

`int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);`：绑定一个固定的socket地址
- `sockfd`：输入参数，socket文件描述符，即socket()的返回值
- `addr`：输入参数，指向sock地址的指针
- `addrlen`：输入参数，sock地址的长度
- 返回值：成功返回0，失败返回-1，并设置errno

`int listen(int sockfd, int backlog);`：监听socket套接字
- `sockfd`：输入参数，被监听的套接字
- `backlog`：输入参数，最大等待队列数量，宏`SOMAXCONN`为系统设定的最大值，可通过`/etc/sysctl.conf`修改
- 返回值：成功返回0，失败返回-1，并设置errno

`int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);`：连接socket套接字
- `sockfd`：输入参数，客户端的套接字
- `addr`：输入参数，服务器的地址
- `addrlen`：输入参数，服务器的地址的长度
- 返回值：成功返回0，失败返回-1，并设置errno

`int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);`：接受socket连接请求
- `sockfd`：输入参数，被监听的套接字
- `addr`：输出参数，返回客户端的地址，可为NULL
- `addrlen`：输入参数，指定客户端的地址的长度，可为NULL
- 返回值：成功返回已连接的新套接字描述符connfd，失败返回-1，并设置errno

`int recv(int sockfd, void *buf, int len, int flags);`：从socket接收数据
- `sockfd`：输入参数，从这个socket接收数据
- `buf`：输出参数，用来保存接收到的数据
- `len`：输入参数，指定buf的长度，表示最多接收这么多个字节的数据
- `flags`：输入参数，flags，指定对应的选项，一般置为0
- 返回值：成功返回接收到的数据大小，返回0表示对方不再发送数据(可以理解为关闭了连接)，出错返回-1，并设置errno

`int send(int sockfd, void *buf, int len, int flags);`：向socket发送数据
- `sockfd`：输入参数，向这个socket发送数据
- `buf`：输入参数，发送buf指向的数据
- `len`：输入参数，指定buf的长度，指定要发送的数据大小
- `flags`：输入参数，flags，指定对应的选项，一般置为0
- 返回值：成功返回已发送的数据大小，失败则返回-1，并设置errno

`int recvfrom(int sockfd, void *buf, int len, int flags, struct sockaddr *addr, socklen_t *addrlen);`：从udp socket接收数据
- `sockfd`：输入参数，从该socket接收数据
- `buf`：输出参数，将接收的数据存放在buf上
- `len`：输入参数，指定buf的长度
- `flags`：输入参数，flags，指定对应的选项，一般置为0
- `addr`：输出参数，保存该数据的发送方地址
- `addrlen`：输入参数，指定发送方地址的长度
- 返回值：成功返回接收到的数据大小，失败则返回-1，并设置errno

`int sendto(int sockfd, void *buf, int len, int flags, struct sockaddr *addr, socklen_t addrlen);`：向udp socket发送数据
- `sockfd`：输入参数，向该socket发送数据
- `buf`：输入参数，发送buf指向的数据
- `len`：输入参数，指定buf的长度
- `flags`：输入参数，flags，指定对应的选项，一般置为0
- `addr`：输入参数，指定接收方的地址
- `addrlen`：输入参数，指定接收方的地址的长度
- 返回值：成功返回发送的数据大小，失败则返回-1，并设置errno

最后一个参数`flags`：
- `MSG_WAITALL`：用于recv，尽可能等待所有数据
- `MSG_DONTWAIT`：用于recv、send，仅本次操作不阻塞
- `MSG_DONTROUTE`：用于send，绕过路由表查找
- `MSG_OOB`：用于recv、send，发送或接收带外数据
- `MSG_PEEK`：用于recv，窥看外来数据，不将其从socket缓冲区中删除

`int shutdown(int sockfd, int howto);`：关闭tcp连接
- `sockfd`：输入参数，即将关闭连接的套接字
- `howto`：输入参数，定义如何关闭：`SHUT_RD`值为0，关闭读、`SHUT_WR`值为1，关闭写、`SHUT_RDWR`值为2，关闭读写
- 返回值：成功返回0，失败返回-1，并设置errno

`int close(int fd);`：关闭socket
- `fd`：输入参数，要关闭的文件描述符fd
- 返回值：成功返回0，失败返回-1，并设置errno

**shutdown和close的区别**
`shutdown()`函数用于tcp连接的socket套接字，对udp无效
用于更精细的控制流的读和写，可以只关闭读，也可以只关闭写，或者读写都关闭，但是关闭后，该sockfd依旧是有效的，仍需调用close进行关闭
而`close()`函数用于关闭对应的fd文件描述符，socket也是特殊的文件，注意，close只是将对应的sockfd的引用计数减1，当引用计数减到0时，系统才关闭该sockfd

还有一点：在多进程程序中，使用shutdown会导致共享进程的连接也被关闭，读写出现错误，而close不会影响共享进程的socket

## tcp三次握手、四次挥手
![tcp三次握手](/images/tcp_handshake.png)
首先，客户端client打开SYN标志位，向服务器server发送seq为x的同步连接请求报文，client进入`SYN_SENT`状态；
server接收到client发来的连接请求后，打开SYN、ACK标志位，发送seq为y、ack为x+1的报文，server进入`SYN_RCVD`状态；
client接收到server返回的ack报文后，检查ack的值是否为x+1，如果是，则打开ACK标志位，发送seq为x+1、ack为y+1的报文，进入`ESTABLISHED`状态；
server接收到client返回的ack报文后，检查ack的值是否为y+1，如果是，则进入`ESTABLISHED`状态，至此，tcp连接建立完成

接下来就是互相发送数据、接收数据了......

![tcp四次挥手](/images/tcp_wave.png)
注意，可以是一个连接的任意一方主动close，这里假设client主动关闭连接：
首先，client打开FIN标志位，向server发送fin报文，告诉server我已经不再发送数据了，进入`FIN_WAIT_1`状态；
server接收到client发来的fin报文后，知道client已经没有数据要发送了，但是此时server依旧可以把未发送的数据给发出去，server打开ACK标志位，向client回应ack报文，告诉client我已经知道了，不过你得先等我把剩余的数据发完，server进入`CLOSE_WAIT`状态；
client接收到server返回的ack报文后，就知道server已经收到了消息，等待server发送完剩余的数据，client进入`FIN_WAIT_2`状态；
server处理完数据后，打开FIN标志位，向client发送fin报文，告诉它我也没有数据要发送了，要关闭连接了，server进入`LAST_ACK`状态；
client接收到server发来的fin报文后，知道server也没东西发送了，于是打开ACK标志位，给server回应ack报文，进入`TIME_WAIT`状态；
server接收到client返回的ack报文后，就确认client已经收到了fin报文，于是可以放心的关闭socket，进入`CLOSED`关闭状态；
而client在等待了2MSL的时间后，发现server已经没有再发送fin报文了，也就确定了server已经接收到了我的ack报文，已经关闭了连接，于是client也关闭了连接，client进入`CLOSED`关闭状态，至此，tcp连接释放完成

注：`MSL`是`报文最大生存时间`，RFC793中规定MSL为2分钟，实际应用中常用的是30秒，1分钟和2分钟等
可以修改`/etc/sysctl.conf`内核参数，来缩短`TIME_WAIT`的时间，避免不必要的资源浪费

最后附上tcp从建立连接到传输数据到释放连接的过程图：
![tcp连接](/images/tcp_handshake_wave.jpg)

## tcp_socket实例
实现一个echo回声的服务端/客户端，即client发送一条消息给server，server就把该消息原样返回给client：

`tcp_echo_server.c`
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
#define BUF_SIZE 128

int main(void){
    int listenfd;
    if((listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        perror("create_listenfd error");
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

    if(listen(listenfd, SOMAXCONN) < 0){
        perror("listen_listenfd error");
        exit(EXIT_FAILURE);
    }

    int connfd;
    struct sockaddr_in peeraddr;
    socklen_t peerlen = sizeof(peeraddr);
    char buf[BUF_SIZE];
    int nbuf;

    for(;;){
        if((connfd = accept(listenfd, (struct sockaddr *)&peeraddr, &peerlen)) < 0){
            perror("accept_listenfd error");
            continue;
        }

        nbuf = recv(connfd, buf, BUF_SIZE, 0);
        buf[nbuf] = 0;
        if(!strcmp(buf, "exit")){
            printf("exit_server\n");
            close(connfd);
            break;
        }
        printf("new conn(%s:%d); msg: %s\n", inet_ntoa(peeraddr.sin_addr), ntohs(peeraddr.sin_port), buf);
        send(connfd, buf, nbuf, 0);
        close(connfd);
    }

    close(listenfd);
    return 0;
}
</script></code></pre>

`tcp_echo_client.c`
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

#define SERV_PORT 8080
#define BUF_SIZE 128

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
    if(inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr) <= 0){
        perror("convert_servaddr error");
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
    printf("echo msg: %s\n", buf);

    close(sockfd);
    return 0;
}
</script></code></pre>

测试echo回声程序：
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/tmp [14:59:29]
$ gcc -o server tcp_echo_server.c

# root @ localhost in ~/tmp [14:59:36]
$ gcc -o client tcp_echo_client.c

# root @ localhost in ~/tmp [14:59:43]
$ ./server

# root @ localhost in ~/tmp [14:57:04]
$ for ((i=0; i<10; i++)); do ./client 'www.zfl9.com'; done && ./client 'exit'
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg:

# root @ localhost in ~/tmp [14:59:43]
$ ./server
new conn(127.0.0.1:55806); msg: www.zfl9.com
new conn(127.0.0.1:55808); msg: www.zfl9.com
new conn(127.0.0.1:55810); msg: www.zfl9.com
new conn(127.0.0.1:55812); msg: www.zfl9.com
new conn(127.0.0.1:55814); msg: www.zfl9.com
new conn(127.0.0.1:55816); msg: www.zfl9.com
new conn(127.0.0.1:55818); msg: www.zfl9.com
new conn(127.0.0.1:55820); msg: www.zfl9.com
new conn(127.0.0.1:55822); msg: www.zfl9.com
new conn(127.0.0.1:55824); msg: www.zfl9.com
exit_server
</script></code></pre>

## udp_socket实例
我们也实现一个echo回声：

`udp_echo_server.c`
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

#define BIND_PORT 8080
#define BUF_SIZE 128

int main(void){
    int sockfd;
    if((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0){
        perror("create_sockfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in bindaddr;
    memset(&bindaddr, 0, sizeof(bindaddr));
    bindaddr.sin_family = AF_INET;
    bindaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    bindaddr.sin_port = htons(BIND_PORT);

    if(bind(sockfd, (struct sockaddr *)&bindaddr, sizeof(bindaddr)) < 0){
        perror("bind_sockfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in peeraddr;
    socklen_t peerlen = sizeof(peeraddr);
    char buf[BUF_SIZE];
    int nbuf;

    for(;;){
        nbuf = recvfrom(sockfd, buf, BUF_SIZE, 0, (struct sockaddr *)&peeraddr, &peerlen);
        buf[nbuf] = 0;
        if(!strcmp(buf, "exit")){
            printf("exit_server\n");
            break;
        }
        printf("new msg(%s:%d): %s\n", inet_ntoa(peeraddr.sin_addr), ntohs(peeraddr.sin_port), buf);
        sendto(sockfd, buf, nbuf, 0, (struct sockaddr *)&peeraddr, peerlen);
    }

    close(sockfd);
    return 0;
}
</script></code></pre>

`udp_echo_client.c`
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

#define SERV_PORT 8080
#define BUF_SIZE 128

int main(int argc, char *argv[]){
    if(argc < 2){
        fprintf(stderr, "usage: %s <MSG>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int sockfd;
    if((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0){
        perror("create_sockfd error");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    if(inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr) <= 0){
        perror("convert_servaddr error");
        exit(EXIT_FAILURE);
    }
    servaddr.sin_port = htons(SERV_PORT);

    char buf[BUF_SIZE];
    int nbuf = strlen(argv[1]);
    sendto(sockfd, argv[1], nbuf, 0, (struct sockaddr *)&servaddr, sizeof(servaddr));
    nbuf = recvfrom(sockfd, buf, BUF_SIZE, 0, NULL, NULL);
    buf[nbuf] = 0;
    printf("echo msg: %s\n", buf);

    close(sockfd);
    return 0;
}
</script></code></pre>

测试echo回声程序：
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/tmp [15:36:35]
$ gcc -o server udp_echo_server.c

# root @ localhost in ~/tmp [15:36:40]
$ gcc -o client udp_echo_client.c

# root @ localhost in ~/tmp [15:36:41]
$ ./server

# root @ localhost in ~/tmp [15:36:46]
$ ./client 'test'
^C

# root @ localhost in ~/tmp [15:36:58] C:130
$ for ((i=0; i<10; i++)); do ./client 'www.zfl9.com'; done && ./client 'exit'
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
echo msg: www.zfl9.com
^C

# root @ localhost in ~/tmp [15:36:41]
$ ./server
new msg(127.0.0.1:45884): test
new msg(127.0.0.1:47339): www.zfl9.com
new msg(127.0.0.1:49160): www.zfl9.com
new msg(127.0.0.1:34802): www.zfl9.com
new msg(127.0.0.1:45306): www.zfl9.com
new msg(127.0.0.1:59937): www.zfl9.com
new msg(127.0.0.1:53763): www.zfl9.com
new msg(127.0.0.1:51414): www.zfl9.com
new msg(127.0.0.1:58480): www.zfl9.com
new msg(127.0.0.1:35993): www.zfl9.com
new msg(127.0.0.1:33064): www.zfl9.com
exit_server
</script></code></pre>
