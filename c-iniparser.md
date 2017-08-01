---
title: c语言 - iniparser解析配置文件
date: 2017-08-01 09:01:00
categories:
- c
tags:
- c
keywords: c语言 iniparser解析配置文件
---

> 
c语言 - iniparser解析配置文件，`IniParser`是一个解析ini文件的C函数库，它小而稳定，并且平台无关，用ANSI C编写

<!-- more -->

## iniparser
[iniparser - github](https://github.com/ndevilla/iniparser)

**安装**
<pre><code class="language-bash line-numbers"><script type="text/plain">git clone https://github.com/ndevilla/iniparser
cd iniparser
make
cp -af src/*.h /usr/local/include/
cp -af libiniparser.* /usr/local/lib/
</script></code></pre>

`example/`文件夹下有示例程序，以及ini配置文件的样例

基本步骤：
**`load`**
`dictionary *iniparser_load(const char *ini_name);`
加载配置文件，在内存中建立字典dictionary

**`get`**
`char *iniparser_getstring(dictionary *d, const char *key, char *default);`
`int iniparser_getint(dictionary *d, const char *key, int default);`
`double iniparser_getdouble(dictionary *d, const char *key, double default);`
`int iniparser_getboolean(dictionary *d, const char *key, int default);`
第1个参数为已装载的字典d
第2个参数为键key，格式为`"section:key"`节+键的组合形式
第3个参数为默认值，即当查找的键key不存在或查找失败时，返回该值

**`set`**
`int iniparser_set(dictionary *d, const char *key, const char *value);`
设置值、修改字典d

**`save`**
`void iniparser_dump_ini(dictionary *d, FILE *fp);`
保存字典d到配置文件

**`dump`**
`void iniparser_dump(dictionary *d, FILE *fp);`
输出字典d

**`free`**
`void iniparser_freedict(dictionary *d);`
释放内存中的字典d

## 例子
**`a.ini`**
<pre><code class="language-ini line-numbers"><script type="text/plain">[server]
host = www.zfl9.com
port = 8080
pass = 123456
ssl = true
tcp = enable
udp = disable

[client]
bind = 0.0.0.0
listen = 1080
verbose = false
daemon = true
</script></code></pre>

**`a.c`**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <string.h>
#include "iniparser.h"

int main(int argc, char *argv[]){
    if(argc < 2){
        fprintf(stderr, "usage: %s <ini.file>\n", argv[0]);
        return 1;
    }

    char *file = argv[1];
    dictionary *ini = iniparser_load(file);

    printf("[server]\n");
    printf("host = %s\n", iniparser_getstring(ini, "server:host", NULL));
    printf("port = %d\n", iniparser_getint(ini, "server:port", 0));
    printf("pass = %s\n", iniparser_getstring(ini, "server:pass", NULL));
    printf("ssl = %d\n", iniparser_getboolean(ini, "server:ssl", 0));
    printf("tcp = %d\n", strcmp(iniparser_getstring(ini, "server:tcp", NULL), "enable") ? 0:1);
    printf("udp = %d\n", strcmp(iniparser_getstring(ini, "server:udp", NULL), "enable") ? 0:1);

    printf("\n[client]\n");
    printf("bind = %s\n", iniparser_getstring(ini, "client:bind", NULL));
    printf("listen = %d\n", iniparser_getint(ini, "client:listen", 0));
    printf("verbose = %d\n", iniparser_getboolean(ini, "client:verbose", 0));
    printf("daemon = %d\n", iniparser_getboolean(ini, "client:daemon", 0));

    iniparser_freedict(ini);
    return 0;
}
</script></code></pre>

**`a.out`**
<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work on git:master x [9:32:35]
$ gcc a.c -liniparser

# root @ localhost in ~/work on git:master x [9:34:09]
$ ./a.out a.ini
[server]
host = www.zfl9.com
port = 8080
pass = 123456
ssl = 1
tcp = 1
udp = 0

[client]
bind = 0.0.0.0
listen = 1080
verbose = 0
daemon = 1
</script></code></pre>
