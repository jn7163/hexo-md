---
title: c语言 - 正则表达式
date: 2017-07-31 13:12:00
categories:
- c
tags:
- c
keywords: c语言 正则表达式 regex.h pcre.h
---

> 
c语言 - 正则表达式，`regex.h`、`pcre.h`

<!-- more -->

## regex.h
> 
`regex.h`是Linux中的一个正则头文件，提供的常用函数有`regcomp()`、`regexec()`、`regfree()`和`regerror()`

**C语言中使用正则表达式一般分为三步**：
- 编译正则表达式 `regcomp()`
- 匹配正则表达式 `regexec()`
- 释放正则表达式 `regfree()`

**函数原型**
`int regcomp(regex_t *compiled, const char *pattern, int cflags);`
- `compiled`：输出参数，保存编译好的regex模式
- `pattern`：输入参数，regex模式
- `cflags`：输入参数，编译选项；`REG_EXTENDED`扩展正则、`REG_ICASE`忽略大小写、`REG_NEWLINE`多行模式、`REG_NOSUB`不存储匹配结果
- 返回值：编译成功，返回0

> 
`regcomp()`默认是`单行模式`，即：`.`匹配所有字符，`^`、`$`匹配字符串开始、结束位置
`REG_NEWLINE`是`多行模式`，即：`.`匹配除`\n`外的所有字符，`^`、`$`匹配行头、行尾位置
注意：对于模式`^.*$\n^.*$`，字符串`www.zfl9.com\nwww.google.com`；`regex.h`的多行模式无法匹配，而`pcre.h`的多行模式正常匹配

`int regexec(regex_t *compiled, char *string, size_t nmatch, regmatch_t matchptr[], int eflags);`
- `compiled`：输入参数，已编译的regex模式
- `string`：输入参数，需要匹配的文本
- `nmatch`：输入参数，matchptr的长度
- `matchptr`：输出参数，保存匹配到的regmatch_t对象
- `eflags`：输入参数，编译选项
- 返回值：匹配成功，返回0

结构体`regmatch_t`：
<pre><code class="language-c line-numbers"><script type="text/plain">typedef int regoff_t;

typedef struct
{
  regoff_t rm_so;  /* Byte offset from string's start to substring's start.  */
  regoff_t rm_eo;  /* Byte offset from string's start to substring's end.  */
} regmatch_t;
</script></code></pre>

`void regfree(regex_t *compiled);`
- `compiled`：输入参数，不需要使用的regex模式

`size_t regerror(int errcode, regex_t *compiled, char *buffer, size_t length);`
- 当执行regcomp()或者regexec()产生错误的时候，调用这个函数可以返回一个包含错误信息的字符串
- `errcode`：输入参数，regcomp()、regexec()返回的错误代号
- `compiled`：输入参数，已编译好的regex模式，可以为NULL
- `buffer`：输出参数，保存相关错误信息的字符串描述
- `length`：输入参数，指明buffer的长度
- 返回值：返回错误描述信息的长度

**re_posix.c**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <regex.h>

#define RMSIZE 20

int main(int argc, char *argv[]){
    if(argc != 3){
        printf("usage: %s <pattern> <str>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    const char *pattern = argv[1];
    const char *str = argv[2];
    regmatch_t regmatch[RMSIZE];

    regex_t re;
    int cflags = REG_EXTENDED;
    //int cflags = REG_EXTENDED + REG_NEWLINE;

    regcomp(&re, pattern, cflags);

    int status = regexec(&re, str, RMSIZE, regmatch, 0);

    printf("pattern: \"%s\"\nstr: \"%s\"\n", pattern, str);

    if(status == REG_NOMATCH){
        printf("no match!\n");
    }else if(status == 0){
        printf("match_str: \"%.*s\"\n", regmatch[0].rm_eo-regmatch[0].rm_so, str+regmatch[0].rm_so);

        int subc = 0;
        for(int i=0; i<(signed int)strlen(pattern); i++){
            if(pattern[i] == '('){
                subc++;
            }
        }

        for(int i=1; i<=subc; i++){
            printf("sub%d_str: \"%.*s\"\n", i, regmatch[i].rm_eo-regmatch[i].rm_so, str+regmatch[i].rm_so);
        }
    }

    regfree(&re);

    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [14:05:56]
$ gcc re_posix.c

# root @ localhost in ~/work [14:06:14]
$ ./a.out
usage: ./a.out <pattern> <str>

# root @ localhost in ~/work [14:06:18] C:1
$ ./a.out '(\w+?)\.(\w+?)\.(\w+)$' https://www.zfl9.com
pattern: "(\w+?)\.(\w+?)\.(\w+)$"
str: "https://www.zfl9.com"
match_str: "www.zfl9.com"
sub1_str: "www"
sub2_str: "zfl9"
sub3_str: "com"

# root @ localhost in ~/work [14:09:11]
$ ./a.out '.*' https://www.zfl9.com$'\n'https://www.google.com.hk
pattern: ".*"
str: "https://www.zfl9.com
https://www.google.com.hk"
match_str: "https://www.zfl9.com
https://www.google.com.hk"

# root @ localhost in ~/work [14:16:29] C:1
$ ./a.out '<.*>' '<h1>hello</h1>'
pattern: "<.*>"
str: "<h1>hello</h1>"
match_str: "<h1>hello</h1>"

# root @ localhost in ~/work [14:16:32]
$ ./a.out '<.*?>' '<h1>hello</h1>'
pattern: "<.*?>"
str: "<h1>hello</h1>"
match_str: "<h1>hello</h1>"
</script></code></pre>

最后一个例子可以发现，posix的正则并不支持非贪婪匹配！

## pcre.h
> 
`PCRE(Perl Compatible Regular Expressions)`是一个Perl库，包括perl兼容的正则表达式库；
这些在执行正则表达式模式匹配时用与Perl 5同样的语法和语义是很有用的；

相比上面的posix的正则库，pcre支持贪婪、非贪婪模式，提供更加强大和灵活的正则函数；

**安装pcre**
- CentOS/RHEL：`yum -y install pcre`
- ArchLinux：`pacman -S --need pcre`

`re_pcre.c`
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <pcre.h>

#define RMSIZE 20

int main(int argc, char *argv[]){
    if(argc != 3){
        printf("usage: %s <pattern> <str>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    const char *pattern = argv[1];
    const char *str = argv[2];
    const char *errmsg = NULL;
    int erroffset = -1;
    int rmoffsets[RMSIZE] = {0};

    pcre *re = NULL;
    int cflags = PCRE_EXTENDED + PCRE_MULTILINE;
    //int cflags = PCRE_EXTENDED + PCRE_DOTALL;
    /*
     * PCRE_EXTENDED    扩展正则表达式
     * PCRE_MULTILINE   多行模式
     * PCRE_DOTALL      单行模式
     * PCRE_CASELESS    忽略大小写
     * PCRE_UTF8        utf-8
    */

    re = pcre_compile(pattern, cflags, &errmsg, &erroffset, NULL);

    if(re == NULL){
        printf("erroffset: %d, errmsg: %s\n", erroffset, errmsg);
        exit(EXIT_FAILURE);
    }

    int status = pcre_exec(re, NULL, str, strlen(str), 0, 0, rmoffsets, RMSIZE);

    printf("pattern: \"%s\"\nstr: \"%s\"\n", pattern, str);

    if(status < 0){
        printf("no match!\n");
    }else if(status > 0){
        printf("match_str: \"%.*s\"\n", rmoffsets[1]-rmoffsets[0], str+rmoffsets[0]);

        int subc = 0;
        for(int i=0; i<(signed int)strlen(pattern); i++){
            if(pattern[i] == '('){
                subc++;
            }
        }

        for(int i=1; i<=subc; i++){
            printf("sub%d_str: \"%.*s\"\n", i, rmoffsets[i*2+1]-rmoffsets[i*2], str+rmoffsets[i*2]);
        }
    }

    pcre_free(re);

    return 0;
}
</script></code></pre>

<pre><code class="language-c line-numbers"><script type="text/plain"># root @ localhost in ~/work [14:30:44]
$ gcc -lpcre re_pcre.c

# root @ localhost in ~/work [14:30:49]
$ ./a.out '<.*>' '<h1>hello</h1>'
pattern: "<.*>"
str: "<h1>hello</h1>"
match_str: "<h1>hello</h1>"

# root @ localhost in ~/work [14:31:13]
$ ./a.out '<.*?>' '<h1>hello</h1>'
pattern: "<.*?>"
str: "<h1>hello</h1>"
match_str: "<h1>"

# root @ localhost in ~/work [14:31:17]
$ ./a.out '^(.*)$\n^(.*)$' www.zfl9.com$'\n'www.google.com
pattern: "^(.*)$\n^(.*)$"
str: "www.zfl9.com
www.google.com"
match_str: "www.zfl9.com
www.google.com"
sub1_str: "www.zfl9.com"
sub2_str: "www.google.com"
</script></code></pre>

## 单行模式、多行模式
**`默认`**
`.`匹配除`\n`外的所有字符，`^`、`$`匹配整个字符串开始、结束位置

**`单行模式`**
`.`匹配包括`\n`之内的所有字符，`^`、`$`匹配整个字符串开始、结束位置

**`多行模式`**
`.`匹配除`\n`外的所有字符，`^`、`$`匹配一行字符串的开始、结束位置

简单来说：`单行模式`改变`.`的意义，`多行模式`改变`^`、`$`的意义
