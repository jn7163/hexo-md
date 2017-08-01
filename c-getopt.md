---
title: c语言 - 解析命令行参数
date: 2017-07-31 16:19:00
categories:
- c
tags:
- c
keywords: c语言 解析命令行参数 getopt getopt_long
---

> 
c语言 - 解析命令行参数，`getopt()`、`getopt_long()`

<!-- more -->

## 参数解析
**函数原型**
`int getopt(int argc, char *argv[], const char *optstr);`
- 头文件`unistd.h`，支持解析短选项，如`-h`
- `argc`：输入参数，参数个数，也就是main()中的argc
- `argv`：输入参数，参数列表，也就是main()中的argv
- `optstr`：输入参数，选项处理方式，定义如何解析传入的参数
- 返回值：getopt()逐个扫描选项，选项解析成功则返回选项，遇到未知选项则返回'?'，直到全部读取完毕，返回-1

**选项处理方式**
- 单个字符，如`a`：表示一个选项a
- 单个字符后接`:`，如`a:`：表示选项a后必须有参数值，参数可以紧跟选项，也可以空格隔开，若无参数则输出错误信息
- 单个字符后接`::`，如`a::`：表示选项a后必须有参数值，参数必须紧跟选项，不能空格隔开，默认参数为0

**相关全局变量**
- `exrern char *optarg`：有参数的选项的参数指针，如`"a:"`，传入参数`"./main -a www.zfl9.com"`，则optarg指向"www.zfl9.com"
- `extetn int optind`：argv[]数组的索引；因为一般选项参数是从argv[1]开始，所以optind初始化为1，下一次调用getopt()函数，从optind位置重新开始检查选项，即从下一个‘-’选项开始
- `extern int opterr`：初始化值为1；当值为0时，getopt()不向stderr输出错误信息
- `extern int optopt`：当选项不在optstr中或者选项缺少必要的参数时，该选项存储在optopt中，并且函数getopt()函数返回'?'

**`getopt_demo.c`**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <unistd.h>
#include <string.h>

int opterr = 0;

int main(int argc, char *argv[]){
    char *optstr = "s:p:m:k:b:l:uv";
    int opt;
    while((opt = getopt(argc, argv, optstr)) != -1){
        switch(opt){
            case 'v':
                printf("version 2.3.0\n");
                return 0;
            case 's':
                printf("server: %s\n", optarg);
                break;
            case 'p':
                printf("port: %s\n", optarg);
                break;
            case 'm':
                printf("method: %s\n", optarg);
                break;
            case 'k':
                printf("passwd: %s\n", optarg);
                break;
            case 'b':
                printf("bind: %s\n", optarg);
                break;
            case 'l':
                printf("listen: %s\n", optarg);
                break;
            case 'u':
                printf("enable udp\n");
                break;
            case '?':
                if(strchr(optstr, optopt) == NULL){
                    fprintf(stderr, "unknown option '-%c'\n", optopt);
                }else{
                    fprintf(stderr, "option requires an argument '-%c'\n", optopt);
                }
                return 1;
        }
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [17:19:20]
$ gcc a.c

# root @ localhost in ~/work [17:20:04]
$ ./a.out

# root @ localhost in ~/work [17:20:07]
$ ./a.out -v
version 2.3.0

# root @ localhost in ~/work [17:20:08]
$ ./a.out -vv
version 2.3.0

# root @ localhost in ~/work [17:20:11]
$ ./a.out -q
unknown option '-q'

# root @ localhost in ~/work [17:20:18] C:1
$ ./a.out -s
option requires an argument '-s'

# root @ localhost in ~/work [17:20:22] C:1
$ ./a.out -s ss.org -p 8080 -m rc4-md5 -k 123456pass -b 0.0.0.0 -l1080 -u
server: ss.org
port: 8080
method: rc4-md5
passwd: 123456pass
bind: 0.0.0.0
listen: 1080
enable udp
</script></code></pre>

**长选项解析**
`int getopt_long(int argc, char *argv[], const char *optstr, const struct option *opts, int *longindex);`
- 头文件`getopt.h`，支持解析短选项、长选项，如`-h`、`--help`
- 前三个参数同getopt()，选项处理方式及4个全局变量同getopt()
- `opts`：输入参数，一个指向结构体option的数组
- `longindex`：输入参数，一般用于调试，一般我们设置为NULL
- 返回值：同getopt()

**`option结构体`**
<pre><code class="language-c line-numbers"><script type="text/plain">struct option {
    const char *name;
    int has_arg;
    int *flag;
    int val;
};

/*
name    长选项的名称，例如"help"
has_arg 是否带参数，0不带参数，1必须带参数，2参数可选
flag    指定长选项如何返回结果，如果flag为NULL，getopt_long()返回val即短选项'h'; 如果flag不为NULL，getopt_long()返回0，并且将val的值存储到flag中
val     短选项的名称，例如'h'
*/
</script></code></pre>

例如：
<pre><code class="language-c line-numbers"><script type="text/plain">struct option opts[] = {
       {"version", 0, NULL, 'v'},
       {"server", 1, NULL, 's'},
       {"help", 0, NULL, 'h'},
       {0, 0, 0, 0}
};
</script></code></pre>

注意：最后一个元素必须为`{0, 0, 0, 0}`，以标识结构体数组的结束，否则出现段错误

**`getopt_long_demo.c`**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <string.h>
#include <getopt.h>

int opterr = 0;

int main(int argc, char *argv[]){
    char *optstr = "s:p:m:k:b:l:uv";
    struct option opts[] = {
        {"server", 1, NULL, 's'},
        {"port", 1, NULL, 'p'},
        {"method", 1, NULL, 'm'},
        {"passwd", 1, NULL, 'k'},
        {"bind", 1, NULL, 'b'},
        {"listen", 1, NULL, 'l'},
        {"udp", 0, NULL, 'u'},
        {"version", 0, NULL, 'v'},
        {0, 0, 0, 0},
    };
    int opt;
    while((opt = getopt_long(argc, argv, optstr, opts, NULL)) != -1){
        switch(opt){
            case 'v':
                printf("version 2.3.0\n");
                return 0;
            case 's':
                printf("server: %s\n", optarg);
                break;
            case 'p':
                printf("port: %s\n", optarg);
                break;
            case 'm':
                printf("method: %s\n", optarg);
                break;
            case 'k':
                printf("passwd: %s\n", optarg);
                break;
            case 'b':
                printf("bind: %s\n", optarg);
                break;
            case 'l':
                printf("listen: %s\n", optarg);
                break;
            case 'u':
                printf("enable udp\n");
                break;
            case '?':
                if(strchr(optstr, optopt) == NULL){
                    fprintf(stderr, "unknown option '-%c'\n", optopt);
                }else{
                    fprintf(stderr, "option requires an argument '-%c'\n", optopt);
                }
                return 1;
        }
    }
    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [17:45:46] C:1
$ ./a.out -s ss.org -p 8080 -m rc4-md5 -k 123456pass -b 0.0.0.0 -l1080 -u
server: ss.org
port: 8080
method: rc4-md5
passwd: 123456pass
bind: 0.0.0.0
listen: 1080
enable udp

# root @ localhost in ~/work [17:46:01]
$ ./a.out --server ss.org --port 8080 --method rc4-md5 --passwd 123456pass --bind 0.0.0.0 --listen 1080 --udp
server: ss.org
port: 8080
method: rc4-md5
passwd: 123456pass
bind: 0.0.0.0
listen: 1080
enable udp
</script></code></pre>
