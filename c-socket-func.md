---
title: c语言 - socket编程(二)
date: 2017-08-08 18:44:00
categories:
- c
tags:
- c
keywords: c语言 socket编程
---

> 
c语言 - socket编程(二)，socket缓冲区、阻塞与非阻塞模式、tcp粘包问题、网络字节序与主机字节序、dns域名解析等等

<!-- more -->

## socket缓冲区
对于tcp_socket，在创建socket的时候同时分配了两个缓冲区：发送缓冲区、接收缓冲区，每个tcp_socket之间的缓冲区的互相独立的
对于udp_socket，只有一个缓冲区：接收缓冲区，没有发送缓冲区，因为udp的不可靠，所以也没必要在内核中保存发送的副本

滑动窗口：
tcp实现流量控制的核心就是滑动窗口，所谓滑动窗口，可以理解为接收缓冲区buffer的剩余空间大小，也就是告诉对方，我还能接受多大的数据
接收到ACK报文后，从ACK报文中可以获得对方的滑动窗口win的大小，表示本次最多只能发送这么大的数据，如果无视改值，发送过多的数据，则对方会直接将溢出部分丢弃，这就保证了接收buffer不会溢出，保证了数据的可靠性

但是udp并没有滑动窗口机制，没有任何流控机制，所以，如果对方不断发送udp包，而应用程序若不及时从内核拷走数据，会导致先前接收到的包被淹没，导致丢包
但是tcp却不会，因为如果对方不断发送数据，而应用程序没有及时的处理buffer中的数据，滑动窗口会逐渐缩小，一直到0，这时，对方也就不能再发送数据了，只能等待应用程序将buffer中的数据拷走之后，窗口才会打开，才会恢复数据的接收，保证了先前的数据不会被淹没

查看内核当前的socket缓冲区大小：
`net.ipv4.tcp_rmem = 32768 436600 873200`：tcp_socket接收缓冲区大小，min、default、max值(byte字节)
`net.ipv4.tcp_wmem = 8192 436600 873200`：tcp_socket发送缓冲区大小，min、default、max值(byte字节)
`net.core.rmem_default = 8388608`：内核socket接收缓冲区默认大小(byte字节)
`net.core.wmem_default = 8388608`：内核socket发送缓冲区默认大小(byte字节)
`net.core.rmem_max = 16777216`：内核socket接收缓冲区最大大小(byte字节)
`net.core.wmem_max = 16777216`：内核socket发送缓冲区最大大小(byte字节)

对于recv、send函数，其实都是在操作这个buffer缓冲区：
对于recv，该函数只是从socket的缓冲区中拷贝数据，并不是从网络中直接获取的数据
对于send，该函数只是往socket的缓冲区中写入数据，写入完毕就返回，并不知道数据什么时候发送给对方，这些都是tcp/ip协议栈做的事情

## tcp粘包问题
现在假设这么一个场景：
应用程序A和应用程序B通过tcp_socket建立了一条tcp连接，然后A将`1`、`2`、`3`内容的数据分三次发送给B；
但是如果B不及时处理buffer中的数据，在等待A发送完后，调用recv从buffer中拷贝数据后，会一次性把三个数据一下拷进来，内容也就变成了`123`，完全变了意义，这就是tcp的粘包问题，也反映了tcp数据的无边界性

对于文件传输这类的应用，这可能并没有什么不妥，因为，接收方只管收就行，管他粘不粘包；
而对于某些应用，如长连接的http，需要在http头部加上CONTENT-LENGTH字段标明内容的长度，以便对方分清界限

但是对于udp，并不会出现粘包问题，因为udp包之间本身是有界限的，也就不存在所谓的粘包问题了

## 网络字节序、主机字节序
我们知道，对于不同的cpu，有不同的字节序：
常见的Big-Endian：PowerPC、IBM、Sun
常见的Little-Endian：X86、DEC
ARM既可以是大端模式、也可以是小端模式，是可以调节的

这就导致了在网络通信时的一个问题，我该怎么知道对方发来的数据是大端序还是小端序？
为了防止此类问题的发生，规定在网络传输中，统一使用大端序

而在recv、send、recvfrom、sendto函数中，已经自动帮我们进行了相应的字节序的转换；
而在处理sockaddr的时候就要注意了，需要我们自己手动的转换相应的字节序

主要用到这几个函数：`htons()`、`ntohs()`、`htonl()`、`ntohl()`
h就是host，也就是主机序，n就是network，也就是网络序，s表示16bit的无符号数，l表示32bit的无符号数

对于ipv4：ip地址就是32bit的无符号数，端口号就是16bit的无符号数，就可以用上面的函数进行相应的转换

不过对于ip地址，我们其实一般都是用点分十进制记法的字符串来表示的，如`"114.114.114.114"`，而并不像端口号直接用整数表示

对于这种形式的ip地址，可以用`inet_addr()`、`inet_aton()`、`inet_ntoa()`、`inet_pton()`、`inet_ntop()`函数进行转换

`in_addr_t inet_addr(const char *string);`：将点分十进制的ip转换为网络序
- `string`：输入参数，点分十进制的ip地址
- 返回值：成功返回网络序的ip地址，失败则返回INADDR_NONE

`int inet_aton(const char *string, struct in_addr *addr);`：将点分十进制的ip转换为网络序
- `string`：输入参数，点分十进制的ip地址
- `addr`：输出参数，struct in_addr类型的指针
- 返回值：成功返回1，失败返回0

`char *inet_ntoa(struct in_addr addr);`：将网络序的ip地址转换为点分十进制的字符串
- `addr`：输入参数，网络序的ip地址
- 返回值：成功返回转换后的点分十进制的ip地址，失败返回NULL

`int inet_pton(int family, const char *string, void *addr);`：将点分十进制的ip转换为网络序
- `family`：输入参数，协议族，AF_INET、AF_INET6
- `string`：输入参数，点分十进制的ip地址
- `addr`：输出参数，保存转换后的网络序ip地址
- 返回值：成功返回1，输入不是有效格式则返回0，失败返回-1

`const char *inet_ntop(int family, const void *addr, char *str, size_t len);`：将网络序的ip地址转换为点分十进制的字符串
- `family`：输入参数，协议族，AF_INET、AF_INET6
- `addr`：输入参数，网络序的ip地址
- `str`：输出参数，保存转换后的点分十进制地址
- `len`：输入参数，指定str的长度
- 返回值：成功返回str的地址，失败返回NULL

总结：不推荐使用inet_addr，有缺陷；对于inet_aton、inet_ntoa只支持ipv4，而inet_pton、inet_ntop则支持ipv4、ipv6

## 在socket中使用域名
`struct hostent *gethostbyname(const char *name);`：将域名解析为ip地址
- `name`：输入参数，域名，如"www.baidu.com"
- 返回值：成功返回一个指向struct hostent类型的指针，失败返回NULL

`struct hostent`结构体：
<pre><code class="language-c line-numbers"><script type="text/plain">struct hostent
{
  char *h_name;			/* Official name of host.  */
  char **h_aliases;		/* Alias list.  */
  int h_addrtype;		/* Host address type.  */
  int h_length;			/* Length of address.  */
  char **h_addr_list;		/* List of addresses from name server.  */
#ifdef __USE_MISC
# define	h_addr	h_addr_list[0] /* Address, for backward compatibility.*/
#endif
};
</script></code></pre>

一般我们使用`h_addr`就行了，如果你想用printf输出它指向的ip地址，别忘了进行字节序转换
`h_addr`是一个指向`struct in_addr`结构体的指针，可以用inet_ntoa()进行转换
如：`printf("host: %s, addr: %s\n", hostname, inet_ntoa(*(struct in_addr *)resolv->h_addr));`，resolv是一个`struct hostent`结构体的指针，即gethostbyname的返回值

## 获取已连接的socket的本地、远程地址
`int getsockname(int sockfd, struct sockaddr *localaddr, socklen_t *addrlen);`：获取本地地址信息
- `sockfd`：输入参数，一个已连接的socket套接字
- `localaddr`：输出参数，保存获取到的本地地址
- `addrlen`：输入参数，指明本地地址的长度
- 返回值：成功返回0，失败返回-1，并设置errno

`int getpeername(int sockfd, struct sockaddr *peeraddr, socklen_t *addrlen);`：获取对方地址信息
- `sockfd`：输入参数，一个已连接的socket套接字
- `peeraddr`：输出参数，保存获取到的对方地址
- `addrlen`：输入参数，指明对方地址的长度
- 返回值：成功返回0，失败返回-1，并设置errno

## socket阻塞与非阻塞
**`tcp_socket`**：
阻塞模式的读(recv)：`数据在不超过指定长度的时候有多少读多少，没有数据就一直等待`；
非阻塞模式的读(recv)：`数据在不超过指定长度的时候有多少读多少，没有数据就立即返回，不等待`；

阻塞模式的写(send)：`会一直等待，直到数据全部被发送完，才会返回`；
非阻塞模式的写(send)：`采取可以写多少就写多少的原则，并不会一直等到将数据全部发送完才返回`；

默认情况下，所有的tcp_socket的相关操作都是阻塞的，我们之前用的也都是阻塞模式的读写；
读：阻塞和非阻塞，一般都会考虑循环读取的方式，因为一次读取不能保证读取到我们想要的长度；
对于阻塞读：recv有一个参数`MSG_WAITALL`，会一直读取到指定长度的数据后才会返回，不过也是尽量读全，在有中断的情况下还是可能被打断，造成没有读到buf_size的长度；
所以即使是采用`阻塞模式recv + MSG_WAITALL`的模式，也不保证可以读到buf_size的长度，还是需要考虑采取循环的方式；
不过一般情况下，MSG_WAITALL还是可以读取到buf_size的长度的，这种方式的读取，性能会比单纯循环读的好一些；

写：阻塞模式下，也不保证能够发送完全部数据，因为还是可能被中断，导致只发送了一部分数据，所以还是需要考虑循环的方式；
不过通常情况下，一次调用还是能够发送完全部的数据

**`udp_socket`**：
对于udp的读，默认情况下也是阻塞的，会一直等待到一个完整的udp数据报的到来才会返回；
对于udp的写，因为没有发送缓冲区，所以不会因为缓冲区而阻塞，不过还是可能因其他原因而阻塞；

**将默认的阻塞套接字设置为非阻塞套接字**：
获取当前socket的flags值：`int flags = fcntl(sockfd, F_GETFL, 0);`
设置成非阻塞套接字：`fcntl(sockfd, F_SETFL, flags|O_NONBLOCK);`
设置为阻塞套接字：`fcntl(sockfd, F_SETFL, flags&~O_NONBLOCK);`

**非阻塞模式的注意事项**：
`connect`发起连接：
- 返回0，表示已建立连接
- 返回-1，errno为`EINPROGRESS`，表示当前进程正在处理
- 返回-1，errno为其它值，表示connect出错

`accept`接收连接：
- 返回>0，表示成功接受连接
- 返回-1，errno为`EAGAIN`，表示没有新连接
- 返回-1，errno为其它值，表示accept出错

`recv`接收数据：
- 返回>0，表示已接收数据
- 返回=0，表示已接收FIN报文(对方关闭了发送通道、关闭了连接等等)
- 返回-1，errno为`EAGAIN`，表示当前没有数据可读，应该继续读取
- 返回-1，errno为`EINTR`，表示当前操作被中断，应该继续读取
- 返回-1，errno为其它值，表示recv出错

`send`发送数据：
- 返回>0，表示已发送数据
- 返回-1，errno为`EAGAIN`，表示当前发送buffer空间不足，应该继续发送
- 返回-1，errno为`EINTR`，表示当前操作被中断，应该继续发送
- 返回-1，errno为其它值，表示send出错
