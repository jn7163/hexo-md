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

## 共享内存 System V
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
- `shm_flg`：输入参数，一组标志位flgs，通常还会指定权限（同文件的权限，如`0644`），通常为`0644 | IPC_CREAT`：
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

## 信号量 System V
**什么是信号量**
为了防止出现因多个程序同时使用一个共享资源而引发的一系列问题，我们需要一种方法，它可以通过生成并使用令牌来授权，在任一时刻只能有一个执行线程访问代码的临界区域；临界区域是指执行数据更新的代码需要独占式地执行；
而信号量就可以提供这样的一种访问机制，让一个临界区同一时间只有一个线程在访问它，也就是说信号量是用来调协进程对共享资源的访问的；

信号量是一个特殊的变量，程序对其访问都是`原子操作`，且只允许对它进行等待（即`P(sem_val)`）和发送（即`V(sem_val)`）操作；
最简单的信号量是只能取0和1的变量，这也是信号量最常见的一种形式，叫做二进制信号量；而可以取多个正整数的信号量被称为通用信号量；这里主要讨论二进制信号量；

**信号量的工作原理**
由于信号量只能进行两种操作：等待和发送信号，即`P(sv)`和`V(sv)`：
- `P(sv)`：如果sv的值大于零，就给它减1；如果它的值为零，就挂起该进程的执行；
- `V(sv)`：如果有其他进程因等待sv而被挂起，就让它恢复运行，如果没有进程因等待sv而挂起，就给它加1；

举个例子，就是两个进程共享信号量sv，一旦其中一个进程执行了P(sv)操作，它将得到信号量，并可以进入临界区，使sv减1；
而第二个进程将被阻止进入临界区，因为当它试图执行P(sv)时，sv为0，它会被挂起以等待第一个进程离开临界区域并执行V(sv)释放信号量，这时第二个进程就可以恢复执行；

**Linux的信号量机制**
Linux提供了一组精心设计的信号量接口来对信号进行操作，它们不只是针对二进制信号量，下面将会对这些函数进行介绍，但请注意，这些函数都是用来对成组的信号量值进行操作的；它们声明在头文件`sys/sem.h`中：

`int semget(key_t sem_key, int sem_nums, int sem_flg);`：创建信号量
- `sem_key`：输入参数，IPC键值，一个key确定一组信号量；同shmget()中的key；
- `sem_nums`：输入参数，要创建的信号量数目；通常我们只需要一个，即为1；
- `sem_flg`：输入参数，一组标志位flgs，同shmget()中的flgs；
- 返回值：成功返回一个sem_id，失败返回-1，并设置errno

`int semop(int sem_id, struct sembuf *sem_opa, size_t sem_opa_len);`：操作信号量的值
- `sem_id`：输入参数，semget()的返回值；
- `sem_opa`：输入参数，`struct sembuf []`类型的数组，通常只有一个元素；
- `sem_opa_len`：输入参数，数组的长度；通常只有一个元素，即为1；
- 返回值：成功返回0，失败返回-1，并设置errno

`struct sembuf`结构体：
<pre><code class="language-c line-numbers"><script type="text/plain">struct sembuf
{
  unsigned short int sem_num;   /* semaphore number */
  short int sem_op;             /* semaphore operation */
  short int sem_flg;            /* operation flag */
  // 有两种状态: SEM_UNDO、SEM_NOWAIT；
  // SEM_NOWAIT  对信号的操作不能满足时，semop()函数就会阻塞，并立即返回，同时设置错误信息;
  // SEM_UNDO    程序结束时(无论是否正常结束)，保证信号值会被重新设为semop()调用前的值;
  //             这样做的目的在于避免在异常情况下结束时未将锁定资源解锁，造成该资源永远锁定;
};
</script></code></pre>

`int semctl(int sem_id, int sem_num, int command, ...);`：控制信号量的属性
- `sem_id`：输入参数，semget()的返回值；
- `sem_num`：输入参数，信号量的下标值；
- `command`：输入参数，操作方式：
`IPC_STAT`：读取一个信号量集的数据结构semid_ds，并将其存储在semun中的buf参数中；
`IPC_SET`：设置信号量集的数据结构semid_ds中的元素ipc_perm，其值取自semun中的buf参数；
`IPC_RMID`：将信号量集从内存中删除；
`GETALL`：用于读取信号量集中的所有信号量的值；
`GETNCNT`：返回正在等待资源的进程数目；
`GETPID`：返回最后一个执行semop操作的进程的PID；
`GETVAL`：返回信号量集中的一个单个的信号量的值；
`GETZCNT`：返回这在等待完全空闲的资源的进程数目；
`SETALL`：设置信号量集中的所有的信号量的值；
`SETVAL`：设置信号量集中的一个单独的信号量的值；
- 返回值：成功返回值大于等于0，失败返回-1，并设置errno

**共享内存和信号量的简单示例**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/sem.h>
#include <sys/wait.h>

#define FILENAME "/dev/null"

union semun{
    int val;
    struct semid_ds *buf;
    unsigned short *array;
};

int sem_set(int sem_id);
int sem_p(int sem_id);
int sem_v(int sem_id);
int sem_del(int sem_id);

int main(void){
    key_t shm_key = ftok(FILENAME, 0);
    if(shm_key == -1){
        perror("ftok");
        exit(EXIT_FAILURE);
    }

    int shm_size = getpagesize();
    int shm_id = shmget(shm_key, shm_size, 0644 | IPC_CREAT);
    if(shm_id == -1){
        perror("shmget");
        exit(EXIT_FAILURE);
    }

    char *data = (char *)shmat(shm_id, NULL, 0);
    if(data == (char *)-1){
        perror("shmat");
        exit(EXIT_FAILURE);
    }
    memset(data, 0, shm_size);

    key_t sem_key = shm_key;
    int sem_id = semget(sem_key, 1, 0644 | IPC_CREAT);
    if(sem_id == -1){
        perror("semget");
        exit(EXIT_FAILURE);
    }

    if(sem_set(sem_id) == -1){
        perror("sem_set");
        exit(EXIT_FAILURE);
    }

    pid_t pid = fork();
    if(pid < 0){
        perror("fork");
        exit(EXIT_FAILURE);
    }else if(pid > 0){
        if(sem_p(sem_id) == -1){
            perror("sem_p");
            exit(EXIT_FAILURE);
        }

        printf("parent_proc(%d) writing now ... \n", getpid());
        for(int i=0; i<3; i++){
            sprintf(data + 5*i, "www%d ", i);
            sleep(1);
        }
        printf("parent_proc(%d) writing complete\n", getpid());

        if(sem_v(sem_id) == -1){
            perror("sem_v");
            exit(EXIT_FAILURE);
        }

        if(shmdt(data) == -1){
            perror("shmdt");
            exit(EXIT_FAILURE);
        }

        wait(NULL);
        if(sem_del(sem_id) == -1){
            perror("sem_del");
            exit(EXIT_FAILURE);
        }
        if(shmctl(shm_id, IPC_RMID, NULL) == -1){
            perror("shmctl");
            exit(EXIT_FAILURE);
        }
    }else if(pid == 0){
        usleep(3);

        printf("child_proc(%d) waiting to read ... \n", getpid());
        if(sem_p(sem_id) == -1){
            perror("sem_p");
            exit(EXIT_FAILURE);
        }

        printf("child_proc(%d) read_data: %s\n", getpid(), data);

        if(sem_v(sem_id) == -1){
            perror("sem_v");
            exit(EXIT_FAILURE);
        }

        if(shmdt(data) == -1){
            perror("shmdt");
            exit(EXIT_FAILURE);
        }
    }

    return 0;
}

int sem_set(int sem_id){
    union semun sem_u;
    sem_u.val = 1;

    if(semctl(sem_id, 0, SETVAL, sem_u) == -1){
        return -1;
    }else{
        return 0;
    }
}

int sem_p(int sem_id){
    struct sembuf sem_b[1];
    sem_b[0].sem_num = 0;
    sem_b[0].sem_op = -1;
    sem_b[0].sem_flg = SEM_UNDO;

    if(semop(sem_id, sem_b, 1) == -1){
        return -1;
    }else{
        return 0;
    }
}

int sem_v(int sem_id){
    struct sembuf sem_b[1];
    sem_b[0].sem_num = 0;
    sem_b[0].sem_op = 1;
    sem_b[0].sem_flg = SEM_UNDO;

    if(semop(sem_id, sem_b, 1) == -1){
        return -1;
    }else{
        return 0;
    }
}

int sem_del(int sem_id){
    if(semctl(sem_id, 0, IPC_RMID) == -1){
        return -1;
    }else{
        return 0;
    }
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [17:17:59]
$ gcc -o shm_sem shm_sem.c

# root @ arch in ~/work on git:master x [17:18:08]
$ ./shm_sem
parent_proc(1360) writing now ...
child_proc(1361) waiting to read ...
parent_proc(1360) writing complete
child_proc(1361) read_data: www0 www1 www2
</script></code></pre>

## 共享内存 Posix
**概念**
共享内存区，按标准可分为Posix共享内存区和System V共享内存区，两者在概念上类似；

Posix表示可移植操作系统接口（Portable Operating System Interface，缩写为POSIX），POSIX标准定义了操作系统应该为应用程序提供的接口标准，是IEEE为要在各种UNIX操作系统上运行的软件而定义的一系列API标准的总称，其正式称呼为IEEE 1003，而国际标准名称为ISO/IEC 9945；

System V，曾经也被称为AT&T System V，是Unix操作系统众多版本中的一支；它最初由AT&T开发，在1983年第一次发布；一共发行了4个System V的主要版本：版本1、2、3和4；

创建的System V共享内存和信号量，可在shell中使用命令`ipcs`查看；
创建的Posix共享内存和信号量，可在`/dev/shm/`中查看到相应文件；

Posix.1提供了两种在无亲缘关系进程间共享内存区的方法
- `内存映射文件`(memory-mapped file)：由open函数打开，由mmap函数把所得到的描述符映射到当前进程空间地址中的一个文件；
- `共享内存区对象`(shared-memory object)：由shm_open函数打开一个Posix.1 IPC名字，所返回的描述符由mmap函数映射到当前进程的地址空间；

这两种共享内存区的区别在于共享的数据的载体(底层支撑对象)不一样：
- `内存映射文件`：数据载体是`物理文件`；
- `共享内存区对象`：数据载体是`物理内存`；

**我们经常说的共享内存，一般是指`共享内存区对象`，也就是共享物理内存**

Posix共享内存区涉及以下两个步骤要求：
（1）指定一个名字参数调用shm_open，以创建一个新的共享内存区对象或打开一个已存在的共享内存区对象；
（2）调用mmap把这个共享内存区映射到调用进程的地址空间；

调用`shm_open`创建或打开的对象其实都是存在于`/dev/shm/`目录下的一个文件，该挂载点的文件系统是`tmpfs`，即内存中的文件系统；

`int shm_open(const char *name, int oflag, mode_t mode);`：创建共享内存
- 头文件：`sys/mman.h`、`sys/stat.h`、`fcntl.h`，须链接库`-lrt`；
- `name`：输入参数，实质为一个文件名，存在于`/dev/shm/`，创建成功后可查看到该文件；
- `oflag`：输入参数，一组flags，多个flag可用按位或`|`连接：
`O_RDONLY`：只读方式打开；
`O_RDWR`：读写模式打开；
`O_CREAT`：如果文件不存在则新建，存在则打开；
`O_CREAT | O_EXCL`：如果文件不存在则新建，存在则返回错误信息；
`O_TRUNC`：清空文件的原有内容；
- `mode`：输入参数，文件的权限，如`0644`，如果打开已有文件，则权限应为`0`；
- 返回值：成功返回文件描述符fd，失败返回-1，并设置errno；

`int shm_unlink(const char *name);`：删除一个共享内存区对象的名字，删除一个名字不会影响对于其底层支撑对象的现有引用，直到对于该对象的引用全部关闭为止
- 头文件：`sys/mman.h`、`sys/stat.h`、`fcntl.h`，须链接库`-lrt`；
- `name`：输入参数，文件名；
- 返回值：成功返回0，失败返回-1，并设置errno；

`int ftruncate(int fd, off_t length);`：修改共享内存对象的大小
- 头文件：`sys/types.h`、`unistd.h`；
- `fd`：输入参数，文件描述符；
- `length`：输入参数，修改后的大小（单位：byte）；
- 返回值：成功返回0，失败返回-1，并设置errno；

`void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);`：挂载共享内存
- 头文件：`sys/mman.h`；
- `addr`：输入参数，挂载的地址，若设置为NULL，则让系统自动选择合适的地址；
- `length`：输入参数，共享内存的长度；
- `prot`：输入参数，一组flags，读写权限，多个可用按位或`|`连接，通常设置为`PROT_READ | PROT_WRITE`：
`PROT_READ`：可读；
`PROT_WRITE`：可写；
`PROT_EXEC`：可执行；
`PROT_NONE`：无权限；
- `flags`：输入参数，一组flags，是否共享：
`MAP_SHARED`：共享的；
`MAP_PRIVATE`：私有的；
- `fd`：输入参数，文件描述符；
- `offset`：输入参数，文件指针偏移量，一般为0；
- 返回值：成功返回挂载的首地址，失败返回`MAP_FAILED`（`(void *)-1`），并设置errno；

**mmap成功返回后，fd参数可以关闭；该操作对由于mmap建立的映射关系没有影响**

`int munmap(void *addr, size_t length);`：分离共享内存
- 头文件：`sys/mman.h`；
- `addr`：输入参数，要分离的共享内存首地址；
- `length`：输入参数，共享内存的长度；
- 返回值：成功返回0，失败返回-1，并设置errno；

**shm实例**
`write.c`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/mman.h>
#include <fcntl.h>

#define FILENAME "shm"
#define MAXSIZE 1024*4

int main(void){
    int fd = shm_open(FILENAME, O_CREAT | O_EXCL | O_RDWR, 0644);
    if(fd == -1){
        perror("shm_open");
        exit(EXIT_FAILURE);
    }

    if(ftruncate(fd, MAXSIZE) == -1){
        perror("ftruncate");
        exit(EXIT_FAILURE);
    }

    char *data = (char *)mmap(NULL, MAXSIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if(data == MAP_FAILED){
        perror("mmap");
        exit(EXIT_FAILURE);
    }
    close(fd);

    strcpy(data, "www.zfl9.com");

    if(munmap(data, MAXSIZE) == -1){
        perror("munmap");
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
#include <sys/stat.h>
#include <sys/mman.h>
#include <fcntl.h>

#define FILENAME "shm"
#define MAXSIZE 1024*4

int main(void){
    int fd = shm_open(FILENAME, O_RDWR, 0);
    if(fd == -1){
        perror("shm_open");
        exit(EXIT_FAILURE);
    }

    if(ftruncate(fd, MAXSIZE) == -1){
        perror("ftruncate");
        exit(EXIT_FAILURE);
    }

    char *data = (char *)mmap(NULL, MAXSIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if(data == MAP_FAILED){
        perror("mmap");
        exit(EXIT_FAILURE);
    }
    close(fd);

    printf("data: %s\n", data);

    if(munmap(data, MAXSIZE) == -1){
        perror("munmap");
        exit(EXIT_FAILURE);
    }

    if(shm_unlink(FILENAME) == -1){
        perror("shm_unlink");
        exit(EXIT_FAILURE);
    }

    return 0;
}
</script></code></pre>


<pre><code class="language-c line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [18:37:38]
$ gcc -o write -lrt write.c

# root @ arch in ~/work on git:master x [18:37:57]
$ gcc -o read -lrt read.c

# root @ arch in ~/work on git:master x [18:38:02]
$ ./write

# root @ arch in ~/work on git:master x [18:38:03]
$ ./read
data: www.zfl9.com
</script></code></pre>

## 信号量 Posix
**基本概念**
`信号量（semaphore）`是一种用于提供不同进程间或一个给定进程的不用线程间`同步手段的原语`；

共有三种类型的信号量：
1）`Posix有名信号量`：使用Posix IPC名字标识，可用于进程或线程间的同步；
2）`Posix基于内存的信号量`：存放在共享内存区中，可用于进程或线程间的同步；
3）`System V信号量`：在内核中维护，可用于进程或者线程间同步；

一个进程可以在某个信号量上执行三种操作：
1）`创建（create）一个信号量`：这还要求调用者指定初始值；
2）`等待（wait）一个信号量`：该操作会测试这个信号量的值，如果其值小于或等于0，那就等待（阻塞），一旦值变为大于0就将它减1，这两个步骤是一个原子操作；
3）`挂出（post）一个信号量`：该操作将信号量的值加1，挂出操作同样是原子的；

有了互斥锁和条件变量还提供信号量的原因是：
“本标准提供信号量的主要目的是提供一种进程间同步方式；这些进程可能共享也可能不共享内存区；互斥锁和条件变量是作为线程间的同步机制说明的，这些线程总是共享（某个）内存区；这两者都是已广泛使用了多年的同步范式；每组原语都特别适合于特定的问题”；

尽管信号量的意图在于进程间同步，互斥锁和条件变量的意图则在于线程间同步，但是信号量也可用于线程间，互斥锁和条件变量也可用于进程间；我们应该使用适合具体应用的那组原语；

上面提到Posix提供两类信号量：`有名（named）信号量`和`基于内存的（memory-based）信号量`，后者也称为`无名（unnamed）信号量`；

有名信号量的创建：`sem_open()`
无名信号量的创建：`sem_init()`
有名/无名信号量的操作：`sem_wait()`、`sem_trywait()`、`sem_post()`、`sem_getvalue()`
有名信号量的销毁：`sem_close()`、`sem_unlink()`
无名信号量的销毁：`sem_destroy()`

**多线程编程通常使用互斥锁、读写锁、条件变量、自旋锁，所以下面不再对基于内存的信号量展开**

**有名信号量**
若信号量名为`somename`，则创建的信号量在`/dev/shm/`中，被命名为`sem.somename`；

`sem_t *sem_open(const char *name, int oflag);`：打开一个已存在的信号量
`sem_t *sem_open(const char *name, int oflag, mode_t mode, unsigned int value);`：创建一个信号量
- 头文件：`sys/stat.h`、`fcntl.h`、`semaphore.h`，须链接库`-lpthread`；
- `name`：输入参数，名称；
- `oflag`：输入参数，一组打开标志位，可用按位或`|`连接，同shm_open的参数；
- `mode`：输入参数，文件权限，同shm_open的参数；
- `value`：输入参数，信号量的初始值；通常为1；
- 返回值：成功返回信号量指针，失败返回`SEM_FAILED`（`(void *)-1`），并设置errno；

`int sem_close(sem_t *sem);`：关闭信号量
- 头文件：`semaphore.h`，须链接库`-lpthread`；
- `sem`：输入参数，信号量指针；
- 返回值：成功返回0，失败返回-1，并设置errno；

`int sem_unlink(const char *name);`：删除信号量
- 头文件：`semaphore.h`，须链接库`-lpthread`；
- `name`：输入参数，信号量文件名；
- 返回值：成功返回0，失败返回-1，并设置errno；

`int sem_wait(sem_t *sem);`：等待，阻塞
`int sem_trywait(sem_t *sem);`：等待，不阻塞
`int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);`：等待，超时
头文件：`semaphore.h`，须链接库`-lpthread`；
成功返回0，失败返回-1，并设置errno；

`int sem_post(sem_t *sem);`：挂出
- 头文件：`semaphore.h`，须链接库`-lpthread`；
- `sem`：输入参数，信号量指针；
- 返回值：成功返回0，失败返回-1，并设置errno；

`int sem_getvalue(sem_t *sem, int *sval);`：测试
- 头文件：`semaphore.h`，须链接库`-lpthread`；
- `sem`：输入参数，信号量指针；
- `sval`：输出参数，获取的值；
- 返回值：成功返回0，失败返回-1，并设置errno；

**共享内存和信号量的简单示例**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/mman.h>
#include <semaphore.h>
#include <fcntl.h>
#include <sys/wait.h>

#define SHM_NAME "shm.test"
#define SEM_NAME "test"
#define SHM_SIZE 1024*4

int main(void){
    int fd = shm_open(SHM_NAME, O_CREAT | O_EXCL | O_RDWR, 0644);
    if(fd == -1){
        perror("shm_open");
        exit(EXIT_FAILURE);
    }

    if(ftruncate(fd, SHM_SIZE) == -1){
        perror("ftruncate");
        exit(EXIT_FAILURE);
    }

    char *data = (char *)mmap(NULL, SHM_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if(data == MAP_FAILED){
        perror("mmap");
        exit(EXIT_FAILURE);
    }
    close(fd);

    sem_t *sem = sem_open(SEM_NAME, O_CREAT | O_EXCL | O_RDWR, 0644, 1);
    if(sem == SEM_FAILED){
        perror("sem_open");
        exit(EXIT_FAILURE);
    }

    pid_t pid = fork();
    if(pid < 0){
        perror("fork");
        exit(EXIT_FAILURE);
    }else if(pid > 0){
        if(sem_wait(sem) == -1){
            perror("sem_wait");
            exit(EXIT_FAILURE);
        }

        printf("parent_proc(%d) writing now ... \n", getpid());
        for(int i=0; i<3; i++){
            sprintf(data + 5*i, "www%d ", i);
            sleep(1);
        }
        printf("parent_proc(%d) writing complete\n", getpid());

        if(sem_post(sem) == -1){
            perror("sem_post");
            exit(EXIT_FAILURE);
        }

        if(sem_close(sem) == -1){
            perror("sem_close");
            exit(EXIT_FAILURE);
        }

        if(munmap(data, SHM_SIZE) == -1){
            perror("munmap");
            exit(EXIT_FAILURE);
        }

        wait(NULL);
        if(sem_unlink(SEM_NAME) == -1){
            perror("sem_unlink");
            exit(EXIT_FAILURE);
        }
        if(shm_unlink(SHM_NAME) == -1){
            perror("shm_unlink");
            exit(EXIT_FAILURE);
        }
    }else if(pid == 0){
        usleep(3);

        printf("child_proc(%d) waiting to read ... \n", getpid());
        if(sem_wait(sem) == -1){
            perror("sem_wait");
            exit(EXIT_FAILURE);
        }

        printf("child_proc(%d) read_data: %s\n", getpid(), data);

        if(sem_post(sem) == -1){
            perror("sem_post");
            exit(EXIT_FAILURE);
        }

        if(sem_close(sem) == -1){
            perror("sem_close");
            exit(EXIT_FAILURE);
        }

        if(munmap(data, SHM_SIZE) == -1){
            perror("munmap");
            exit(EXIT_FAILURE);
        }
    }

    return 0;
}
</script></code></pre>


<pre><code class="language-c line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [19:17:58]
$ gcc -o sem sem.c -lrt -lpthread

# root @ arch in ~/work on git:master x [19:18:20]
$ ./sem
parent_proc(9484) writing now ...
child_proc(9485) waiting to read ...
parent_proc(9484) writing complete
child_proc(9485) read_data: www0 www1 www2
</script></code></pre>
