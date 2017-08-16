---
title: c语言 - 进程间通信 共享内存
date: 2017-08-16 09:03:33
categories:
- c
tags:
- c
keywords: c语言 进程间通信 共享内存
---

> 
c语言 - 进程间通信 共享内存

<!-- more -->

## 共享内存
**什么是共享内存**
顾名思义，共享内存就是允许两个不相关的进程访问同一个逻辑内存；共享内存是在两个正在运行的进程之间共享和传递数据的一种非常有效的方式；
不同进程之间共享的内存通常安排为同一段物理内存，进程可以将同一段共享内存连接到它们自己的地址空间中，所有进程都可以访问共享内存中的地址；
而如果某个进程向共享内存写入数据，所做的改动将立即影响到可以访问同一段共享内存的任何其他进程；

特别提醒：共享内存并未提供同步机制，也就是说，在第一个进程结束对共享内存的写操作之前，并无自动机制可以阻止第二个进程开始对它进行读取；所以我们通常需要用其他的机制来同步对共享内存的访问，例如`信号量`、`互斥锁`；

**共享内存的函数接口**
头文件：`sys/types.h`、`sys/ipc.h`、`sys/shm.h`

`int shmget(key_t shm_key, size_t shm_size, int shm_flg);`：创建共享内存
- `shm_key`：输入参数，shm_key用来标识一块共享内存：
shm_key通常通过函数`ftok()`来获取，或者自己指定一个固定值；
shm_key为`0`或`IPC_PRIVATE`，则会建立新的共享内存；
- `shm_size`：输入参数，共享内存的大小（单位：byte）：
注意内存分配的单位是页（一般为4kb，可通过`getpagesize()`获取）；
也就是说如果shm_size为1，那么也会分配4096字节的内存；
只获取共享内存时，shm_size可指定为0；
- `shm_flg`：输入参数，flags，通常还会指定权限（同文件的权限，如`0644`），通常为`0644 | IPC_CREAT`：
`0`：取共享内存标识符，若不存在则函数会报错；
`IPC_CREAT`：如果内核中不存在键值与shm_key相等的共享内存，则新建一个共享内存；如果存在这样的共享内存，返回此共享内存的标识符；
`IPC_CREAT | IPC_EXCL`：如果内核中不存在键值与shm_key相等的共享内存，则新建一个共享内存；如果存在这样的共享内存则报错；
- 返回值：成功返回`shm_id`，失败返回-1，并设置errno

`key_t ftok(const char *filename, int id);`：获取一个IPC键值
- 系统建立IPC通讯（消息队列、信号量和共享内存）时必须指定一个ID值；通常情况下，该id值通过ftok函数得到；
当删除重建文件后，索引节点号由操作系统根据当时文件系统的使用情况分配，因此与原来不同，所以得到的索引节点号也不同；
如果要确保key_t值不变，要么确保ftok的文件不被删除，要么不用ftok，指定一个固定的key_t值；
- `filename`：输入参数，一个必须真实存在的文件名（含路径），一般设置为一个固定的文件，如`/dev/null`；
- `id`：输入参数，子序号，一个8bit的整型（0-255）；
- 返回值：成功返回获取的key，失败返回-1，并设置errno；

`void *shmat(int shm_id, const void *shm_addr, int shm_flg);`：挂接共享内存
- `shm_id`：输入参数，`shmget()`的返回值；
- `shm_addr`：输入参数，指定挂接位置：若为NULL，则让系统自动选择合适的位置；
- `shm_flg`：输入参数，一组标志位flgs，通常为0，`SHM_RDONLY`表示只读模式，其他为读写模式；
- 返回值：成功返回首地址，失败返回-1，并设置errno

`int shmdt(const void *shm_addr);`：分离共享内存
- `shm_addr`：输入参数，挂接的地址；
- 返回值：成功返回0，失败返回-1，并设置errno

`int shmctl(int shm_id, int shm_cmd, struct shmid_ds *buf);`：控制共享内存
- `shm_id`：输入参数，`shmget()`的返回值；
- `shm_cmd`：输入参数，操作的命令：
`IPC_STAT`：获取共享内存的状态，并保存至buf中；
`IPC_SET`：设置共享内存的状态，把buf所指的shmid_ds结构中的uid、gid、mode复制到共享内存的shmid_ds结构内；
`IPC_RMID`：删除共享内存；
- `buf`：`struct shmid_ds *`类型的指针，可以为NULL；
- 返回值：成功返回0，失败返回-1，并设置errno

**shm共享内存简单实例**
`write.c`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>

#define FILENAME "/dev/null"

int main(int argc, char *argv[]){
    if(argc < 2){
        fprintf(stderr, "usage: %s string\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    key_t shm_key = ftok(FILENAME, 0);
    int shm_size = getpagesize();
    int shm_id = shmget(shm_key, shm_size, 0644 | IPC_CREAT);

    if(shm_id == -1){
        perror("shmget");
        exit(EXIT_FAILURE);
    }

    char *data = (char *)shmat(shm_id, NULL, 0);
    if(data == (void *)-1){
        perror("shmat");
        exit(EXIT_FAILURE);
    }

    strcpy(data, argv[1]);

    if(shmdt(data) == -1){
        perror("shmdt");
        exit(EXIT_FAILURE);
    }

    return 0;
}
</script></code></pre>


`read.c`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>

#define FILENAME "/dev/null"

int main(void){
    key_t shm_key = ftok(FILENAME, 0);
    int shm_size = getpagesize();
    int shm_id = shmget(shm_key, shm_size, 0644 | IPC_CREAT);

    if(shm_id == -1){
        perror("shmget");
        exit(EXIT_FAILURE);
    }

    char *data = (char *)shmat(shm_id, NULL, 0);
    if(data == (void *)-1){
        perror("shmat");
        exit(EXIT_FAILURE);
    }

    printf("data: %s\n", data);

    if(shmdt(data) == -1){
        perror("shmdt");
        exit(EXIT_FAILURE);
    }

    if(shmctl(shm_id, IPC_RMID, NULL) == -1){
        perror("shmctl");
        exit(EXIT_FAILURE);
    }

    return 0;
}
</script></code></pre>


<pre><code class="language-c line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [10:39:48]
$ gcc -o write write.c

# root @ arch in ~/work on git:master x [10:40:22]
$ gcc -o read read.c

# root @ arch in ~/work on git:master x [10:40:27]
$ ./write
usage: ./write string

# root @ arch in ~/work on git:master x [10:40:29] C:1
$ ./write www.zfl9.com

# root @ arch in ~/work on git:master x [10:40:34]
$ ./read
data: www.zfl9.com
</script></code></pre>
