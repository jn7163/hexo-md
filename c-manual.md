---
title: c语言 - 常用函数手册
date: 2017-07-18 15:36:00
categories:
- c
tags:
- c
keywords: c语言 常用函数查找手册
---

> 
c语言 - 常用函数手册

<!-- more -->

## stdio.h
宏：
- `FOPEN_MAX`：一个整数，代表了系统可以打开的文件数量
- `FILENAME_MAX`：一个整数，代表了文件名的最大长度
- `L_tmpnam`：一个整数，代表了由 tmpnam() 函数创建的临时文件名的最大长度
- `TMP_MAX`：这个宏是 tmpnam() 函数可生成的独特文件名的最大数量

函数：
- `int remove(const char *filename)`：删除文件
- `int rename(const char *oldname, const char *newname)`：重命名文件
- `FILE *tmpfile(void)`：以 wb+ 模式创建临时文件
- `char *tmpnam(char *str)`：生成并返回一个有效的临时文件名

## stdlib.h
宏：
- `EXIT_FAILURE`：这是 exit() 函数失败时要返回的值
- `EXIT_SUCCESS`：这是 exit() 函数成功时要返回的值
- `RAND_MAX`：这个宏是 rand() 函数返回的最大值

函数：
- `int atoi(const char *str)`：转换为int
- `long int atol(const char *str)`：转换为long int
- `double atof(const char *str)`：转换为double
- `long int strtol(const char *str, char **endptr, int base)`：转换为long int
- `unsigned long int strtoul(const char *str, char **endptr, int base)`：转换为unsigned long int
- `double strtod(const char *str, char **endptr)`：转换为double
- `void abort(void)`：终止一个程序
- `int atexit(void (*func)(void))`：当程序正常终止时，调用函数func
- `char *getenv(const char *name)`：获取环境变量
- `int system(const char *command)`：传递命令给系统执行
- `int abs(int x)`：返回x的绝对值
- `long int labs(long int x)`：返回x的绝对值

## string.h
- `char *strchr(const char *str, int c)`：在参数 str 所指向的字符串中搜索第一次出现字符 c 的位置
- `char *strrchr(const char *str, int c)`：在参数 str 所指向的字符串中搜索最后一次出现字符 c 的位置
- `int strcmp(const char *str1, const char *str2)`：把 str1 所指向的字符串和 str2 所指向的字符串进行比较
- `int strncmp(const char *str1, const char *str2, size_t n)`：把 str1 和 str2 进行比较，最多比较前 n 个字节
- `void *memchr(const char *str, int c, size_t n)`：在参数 str 所指向的字符串的前 n 个字节中搜索第一次出现字符 c 的位置
- `int memcmp(const void *str1, const void *str2, size_t n)`：把 str1 和 str2 的前 n 个字节进行比较
- `char *strncat(char *dest, const char *src, size_t n)`：把 src 所指向的字符串追加到 dest 所指向的字符串的结尾，直到 n 字符长度为止
- `char *strncpy(char *dest, const char *src, size_t n)`：把 src 所指向的字符串复制到 dest，最多复制 n 个字符

## ctype.h
测试和映射字符，这些函数接受 int 为参数，如果满足描述的条件，返回非零(true)，不满足则返回0
- `int islower(int c)`：小写字母
- `int isupper(int c)`：大写字母
- `int isalpha(int c)`：字母
- `int isalnum(int c)`：字母或数字
- `int isdigit(int c)`：十进制数字
- `int isxdigit(int c)`：十六进制数字
- `int iscntrl(int c)`：控制字符
- `int isprint(int c)`：可打印字符
- `int isgraph(int c)`：有图形表示法的字符
- `int ispunct(int c)`：标点符号字符
- `int isspace(int c)`：空白符
- `int tolower(int c)`：转换为小写字母
- `int toupper(int c)`：转换为大写字母

## math.h
定义了各种数学函数，在这个库中所有可用的功能都带有一个 double 类型的参数，且都返回 double 类型的结果
- `double modf(double x, double *integer)`：返回值为小数部分，并设置integer为整数部分
- `double pow(double x, double y)`：返回x的y次幂
- `double sqrt(double x)`：返回x的平方根
- `double floor(double x)`：返回小于或等于x的最大的整数值
- `double ceil(double x)`：返回大于或等于x的最小的整数值
- `double fabs(double x)`：返回x的绝对值
- `double fmod(double x, double y)`：返回x除以y的余数
