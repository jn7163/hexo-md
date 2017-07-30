---
title: (c语言)数据结构与算法 - 串
date: 2017-07-30 10:54:00
categories:
- c
- 数据结构与算法
tags:
- c
- 数据结构与算法
keywords: c语言 数据结构与算法 串 字符串
---

> 
串(即字符串)是一种特殊的线性表，它的数据元素仅由一个字符组成，计算机非数值处理的对象经常是字符串数据；
如在汇编和高级语言的编译程序中，源程序和目标程序都是字符串数据；在事物处理程序中，顾客的姓名、地址、货物的产地、名称等，一般也是作为字符串处理的

<!-- more -->

## 串的定义
**串是由零个或多个任意字符组成的字符序列**
一般记作：`s="a1 a2 a3 ... an"`

其中`s`是串名；用双引号作为串的定界符，引号引起来的字符序列为串值，引号本身不属于串的内容；
`ai(1<=i<=n)`是一个任意字符，它称为串的元素，是构成串的基本单位，`i`是它在整个串中的序号；
`n`为串的长度，表示串中所包含的字符个数，当`n=0`时，称为空串，通常记为`Ф`

**几个术语**
`子串`与`主串`：串中任意连续的字符组成的子序列称为该串的子串，包含子串的串相应地称为主串；
`子串的位置`：子串的第一个字符在主串中的序号称为子串的位置；
`串相等`：称两个串是相等的，是指两个串的长度相等且对应字符都相等；

**串的基本运算**
- `StrLen(s)`：求串长
操作条件：串s存在
操作结果：求出串s的长度

- `StrAssign(s1, s2)`：串赋值
操作条件：s1是一个串变量，s2或者是一个串常量，或者是一个串变量(通常s2是一个串常量时称为串赋值，是一个串变量称为串拷贝)
操作结果：将s2的串值赋值给s1，s1原来的值被覆盖掉

- `StrCat(s1, s2, s)`或`StrCat (s1, s2)`：串拼接
操作条件：串s1，s2存在
操作结果：两个串的联接就是将一个串的串值紧接着放在另一个串的后面，连接成一个串；前者是产生新串s，s1和s2不改变；后者是在s1的后面联接s2的串值，s1改变，s2不改变

- `SubStr(s, i, len)`：求子串
操作条件：串s存在，`1<=i<=StrLen(s)`，`0<=len<=StrLen(s)-i+1`
操作结果：返回从串s的第i个字符开始的长度为len的子串，len=0得到的是空串

- `StrCmp(s1, s2)`：串比较
操作条件：串s1，s2存在
操作结果：若`s1==s2`，操作返回值为0；若`s1<s2`，返回值小于0；若`s1>s2`，返回值大于0

- `StrIndex(s, t)`：子串定位
操作条件：串s，t存在
操作结果：若`t∈s`，则操作返回t在s中首次出现的位置，否则返回值为-1

- `StrInsert(s, i, t)`：串插入
操作条件：串s，t存在，`1<=i<=StrLen(s)+1`
操作结果：将串t插入到串s的第i个字符位置上，s的串值发生改变

- `StrDelete(s, i, len)`：串删除
操作条件：串s存在，`1<=i<=StrLen(s)`，`0<=len<=StrLen(s)-i+1`
操作结果：删除串s中从第i个字符开始的长度为len的子串，s的串值改变

- `StrRep(s, t, r)`：串替换
操作条件：串s，t，r存在，t不为空
操作结果：用串r替换串s中出现的所有与串t相等的不重叠的子串，s的串值改变

其中`前5个操作`是最为基本的，它们不能用其他的操作来合成，因此通常将这5个基本操作称为`最小操作集`

## 串的实现
**`String.h`**
<pre><code class="language-c line-numbers"><script type="text/plain">#ifndef __STRING_H
#define __STRING_H

#define MAXSIZE 256
typedef char String[MAXSIZE];

/**
 * 求字符串长度(不包括'\0'字符)
 * @param str   字符串str
 * @return int  返回长度len
**/
extern int StrLen(const char *str);

/**
 * 字符串赋值，将str2的串值赋值给str1，str1原来的串值被覆盖
 * @param str1      字符串str1
 * @param str2      字符串str2
 * @return char *   返回str1的指针
**/
extern char *StrCpy(char *str1, const char *str2);

/**
 * 字符串拼接，将s1、s2的串值拼接在一起，并保存到s
 * @param s1        字符串s1
 * @param s2        字符串s2
 * @param s         字符串s
 * @return char *   返回s的指针
**/
extern char *StrCat1(const char *s1, const char *s2, char *s);

/**
 * 字符串拼接，将s2的串值拼接在s1之后
 * @param s1        字符串s1
 * @param s2        字符串s2
 * @return char *   返回s1的指针
**/
extern char *StrCat2(char *s1, const char *s2);

/**
 * 提取子串，从str的第i个字符开始，提取len长度，保存到substr
 * @param substr    字符串substr
 * @param str       字符串str
 * @param i         索引值i
 * @param len       提取长度len
 * @return char *   返回substr的指针
**/
extern char *StrSub(char *substr, const char *str, int i, int len);

/**
 * 比较两个字符串，若s1<s2则返回负值，若s1=s2则返回0，若s1>s2则返回正值
 * @param s1    字符串s1
 * @param s2    字符串s2
 * @return int  0: s1=s2; 正整数: s1>s2; 负整数: s1<s2;
**/
extern int StrCmp(const char *s1, const char *s2);

/**
 * 字符串插入，将字符串ins插入到字符串str的第i个位置
 * @param str       字符串str
 * @param i         索引值i
 * @param ins       字符串ins
 * @return char *   返回str的指针
**/
extern char *StrIns(char *str, int i, const char *ins);

/**
 * 字符串删除，从str第i个字符起，删除len个长度的字符
 * @param str       字符串str
 * @param i         索引值i
 * @param len       长度len
 * @return char *   返回str的指针
**/
extern char *StrDel(char *str, int i, int len);

/**
 * 查找子串，BF算法，在字符串str中查找pattern匹配的子串，并返回其位置
 * @param str       字符串str
 * @param pattern   字符串pattern(也称为模式)
 * @return int      若查找成功，返回第一个字符的位置，否则返回-1
**/
extern int StrIndexBF(const char *str, const char *pattern);

#endif
</script></code></pre>

**`String.c`**
<pre><code class="language-c line-numbers"><script type="text/plain">#include "String.h"
#include <stdio.h>

int StrLen(const char *str){
    int len = 0;
    while(str[len] != '\0'){
        len++;
    }
    return len;
}

char *StrCpy(char *str1, const char *str2){
    int i = 0;
    for(; str2[i]!='\0'; i++){
        str1[i] = str2[i];
    }
    str1[i] = '\0';
    return str1;
}

char *StrCat1(const char *s1, const char *s2, char *s){
    if(StrLen(s1) + StrLen(s2) + 1 > MAXSIZE){
        return NULL;
    }else{
        int i = 0, j = 0;
        while(s1[i] != '\0'){
            s[j++] = s1[i++];
        }
        i = 0;
        while(s2[i] != '\0'){
            s[j++] = s2[i++];
        }
        s[j] = '\0';
        return s;
    }
}

char *StrCat2(char *s1, const char *s2){
    int len1 = StrLen(s1);
    int len2 = StrLen(s2);
    if(len1 + len2 + 1 > MAXSIZE){
        return NULL;
    }else{
        int i = 0;
        for(; i<len2; i++){
            s1[len1+i] = s2[i];
        }
        s1[len1+i] = '\0';
        return s1;
    }
}

char *StrSub(char *substr, const char *str, int i, int len){
    int strlen = StrLen(str);
    if(i < 0 || i >= strlen || len < 0 || i + len > strlen){
        return NULL;
    }else{
        int j = 0;
        for(; j<len; j++){
            substr[j] = str[i+j];
        }
        substr[j] = '\0';
        return substr;
    }
}

int StrCmp(const char *s1, const char *s2){
    int i = 0;
    while(s1[i] == s2[i] && s1[i] != '\0'){
        i++;
    }
    return s1[i] - s2[i];
}

char *StrIns(char *str, int i, const char *ins){
    int len1 = StrLen(str);
    int len2 = StrLen(ins);
    if(len1 + len2 + 1 > MAXSIZE){
        return NULL;
    }else if(i == len1){
        int j = 0;
        for(; j<len2; j++){
            str[len1+j] = ins[j];
        }
        str[len1+j] = '\0';
        return str;
    }else{
        for(int j=len1; j>=i; j--){
            str[j+len2] = str[j];
        }
        for(int j=0; j<len2; j++){
            str[i+j] = ins[j];
        }
        return str;
    }
}

char *StrDel(char *str, int i, int len){
    int j;
    for(j=i+len; str[j]!='\0'; j++){
        str[j-len] = str[j];
    }
    str[j-len] = '\0';
    return str;
}

int StrIndexBF(const char *str, const char *pattern){
    int len1 = StrLen(str);
    int len2 = StrLen(pattern);
    int i = 0, j = 0;
    while(i < len1 && j < len2){
        if(str[i] == pattern[j]){
            i++;
            j++;
        }else{
            i = i - j + 1;
            j = 0;
        }
    }
    if(j == len2){
        return i - len2;
    }else{
        return -1;
    }
}
</script></code></pre>
