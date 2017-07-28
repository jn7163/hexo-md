---
title: (c语言)数据结构与算法 - 顺序表
date: 2017-07-24 08:10:00
categories:
- c
- 数据结构与算法
tags:
- c
- 数据结构与算法
keywords: C语言 数据结构与算法 顺序表 线性表
---

> 
线性表是`最简单`、`最基本`、也是`最常用`的一种`线性结构`;
它有两种存储方法：`顺序存储`和`链式存储`;
它的主要基本操作是`插入`、`删除`和`检索`等

<!-- more -->

## 线性表的定义
**线性表是具有相同数据类型的n(n>=0)个数据元素的有限序列**
通常记为`L = (a1, a2, ..., ai-1, ai, ai+1, ..., an)`；其中n为表长，n=0时称为空表

表中相邻元素之间存在着顺序关系；将`ai-1`称为`ai`的`直接前趋`，`ai+1`称为`ai`的`直接后继`
其中，a1是唯一的“第一个”数据元素，又称为`表头元素`；an是唯一的“最后一个”数据元素，又称为`表尾元素`;
除第一个元素外，每个元素有且仅有一个直接前驱。除最后一个元素外，每个元素有且仅有一个直接后继

由此，我们得出线性表的特点如下：
- 表中元素的个数有限
- 表中元素具有逻辑上的顺序性，在序列中各元素排序有其先后次序
- 表中元素都是数据元素，每一个表元素都是单个元素
- 表中元素的数据类型都相同，这意味着每一个表元素占有相同数量的存储空间
- 表中元素具有抽象性，就是说，仅讨论表元素之间的逻辑关系，不考虑元素究竟表示什么内容

注意：线性表是一种逻辑结构，表示元素之间一对一的相邻关系；顺序表和链表是指存储结构，两者属于不同层面的概念，不要将其混淆

## 线性表的基本操作
一个数据结构的基本操作是指其最核心、最基本的操作；其他较复杂的操作可以通过调用其基本操作来实现，线性表的主要操作如下：
- `InitList(L)`：`初始化表`；构造一个空的线性表
- `Length(L)`：`求表长度`；返回线性表L的长度，即L中数据元素的个数
- `LocateElem(L, e)`：`按值查找`操作；在表L中查找具有给定关键字值的元素
- `GetElem(L, i)`：`按位查找`操作；获取表L中第i个位置的元素的值
- `ListInsert(L, i, e)`：`插入`操作；在表L中第i个位置上插入指定元素
- `ListDelete(L, i, e)`：`删除`操作；删除表L中第i个位置的元素
- `PrintList(L)`：`输出`操作；按前后顺序输出线性表L的所有元素值
- `Empty(L)`：`判空`操作；若L为空表，则返回true,否则返回false
- `DestroyList(L)`：`销毁`操作；销毁线性表，并释放线性表L所占用的内存空间

`数据结构的运算`是定义在`逻辑结构`层次上的;
而`运算的具体实现`是建立在`存储结构`上的。

## 顺序表
`线性表的顺序存储`是指在内存中用地址连续的一块存储空间顺序存放线性表的各元素，用这种存储形式存储的线性表称其为`顺序表`；

只要知道`顺序表首地址`和`每个数据元素所占地址单元的个数`就可求出`第i个数据元素的地址`来，这也是**顺序表具有按数据元素的序号随机存取的特点**

`一维数组`在内存中占用的存储空间就是一组连续的存储区域，因此，用一维数组来表示顺序表的数据存储区域是再合适不过的

从结构性上考虑，通常将`data`和`len`封装为一个结构，作为顺序表的类型：
<pre><code class="language-c line-numbers"><script type="text/plain">typedef struct{
    datatype data[MAXSIZE];
    int len;
} SeqList;
</script></code></pre>

定义一个顺序表：`SeqList L`，`L.len`为当前表长，`MAXSIZE`为一个足够大的整数，数据元素分别存放在`L.data[0] ~ L.data[L.len-1]`

有时候定义一个指向类型为`SeqList`的指针更为方便：`SeqList *L`
通过`L = (SeqList *)malloc(sizeof(SeqList));`申请内存空间
L是一个指针，存放的是顺序表的地址，表长`L->len`，存储区域`L->data`，存储空间为`L->data[0] ~ L->data[L->len-1]`

## c语言实现顺序表
**`SeqList.h`**
<pre><code class="language-c line-numbers"><script type="text/plain">#ifndef __SEQLIST_H
#define __SEQLIST_H

#define MAXSIZE 100   // MAXSIZE
typedef struct{
    int data[MAXSIZE];    // data
    int len;        // data的长度，即元素个数
} SeqList;

// 初始化
extern SeqList *Init_SeqList();

// 判空
extern int Empty_SeqList(SeqList *L);

// 获取长度
extern int Length_SeqList(SeqList *L);

// 打印SeqList
extern void Print_SeqList(SeqList *L);

// 销毁
extern int Destroy_SeqList(SeqList *L);

// 取表元
extern int GetElem_SeqList(SeqList *L, int i);

// 按值查找
extern int Locate_SeqList(SeqList *L, int x);

// 表尾插入元素
extern int Append_SeqList(SeqList *L, int x);

// 移除表的最后一项
extern int Pop_SeqList(SeqList *L);

// 插入元素
extern int Insert_SeqList(SeqList *L, int index, int x);

// 删除元素
extern int Delete_SeqList(SeqList *L, int index);

// 将顺序表(a1, a2, ..., an)以a1为界重排：a1前面的值均比a1小; a1后面的值均比a1大
extern int Part_SeqList(SeqList *L);

// 合并两个增序列为新的增序列
extern SeqList *Merge_SeqList(SeqList *L1, SeqList *L2);

// 比较两个顺序表
extern int Compare_SeqList(SeqList *L1, SeqList *L2);

// 查找最大值
extern int Max_SeqList(SeqList *L);

// 查找最小值
extern int Min_SeqList(SeqList *L);

// 排序，rev参数如果为1表示倒序排列，为0则表示正序排列
extern int Sort_SeqList(SeqList *L, int rev);

// 逆置元素
extern void Reverse_SeqList(SeqList *L);

#endif
</script></code></pre>

**`SeqList.c`**
<pre><code class="language-c line-numbers"><script type="text/plain">#include "SeqList.h"
#include <stdio.h>
#include <stdlib.h>

// 初始化
SeqList *Init_SeqList(){
    SeqList *L = (SeqList *)malloc(sizeof(SeqList));
    if(L == NULL){
        return NULL;    // 申请内存失败
    }else{
        L->len = 0; // 将len长度置为0
        return L;   // 返回SeqList首地址
    }
}

// 判空
int Empty_SeqList(SeqList *L){
    if(L == NULL || L->len == 0){
        return 1;   // true, 当前SeqList为空
    }else{
        return 0;   // false, 不为空
    }
}

// 获取长度
int Length_SeqList(SeqList *L){
    if(L == NULL){
        return -1;  // 传入的参数有误
    }
    return L->len;  // 返回当前顺序表的长度
}

// 打印SeqList
void Print_SeqList(SeqList *L){
    if(L == NULL || L->len == 0){
        return; // 传入的参数有误
    }
    printf("Length:%d  { ", L->len);
    for(int i=0; i<L->len; i++){
        printf("%d, ", L->data[i]);
    }
    printf("\b\b }\n");
}

// 销毁
int Destroy_SeqList(SeqList *L){
    if(L == NULL){
        return 0;   // false, 空指针
    }else{
        free(L);
        return 1;   // true, 操作成功
    }
}

// 取表元
int GetElem_SeqList(SeqList *L, int i){
    if(L == NULL || L->len == 0 || i<0 || i>=L->len){
        return 0;   // 传入的参数有误
    }
    return L->data[i];  // 返回索引为i的元素的值
}

// 按值查找
int Locate_SeqList(SeqList *L, int x){
    if(L == NULL || L->len == 0){
        return -1;  // 传入的参数有误
    }else{
        int index = -1;
        for(int i=0; i<L->len; i++){
            if(L->data[i] == x){
                index = i;
                break;
            }
        }
        return index;   // 若查找成功, 返回第一次出现的索引值index; 否则返回-1
    }
}

// 表尾插入元素
int Append_SeqList(SeqList *L, int x){
    if(L == NULL || L->len == MAXSIZE){
        return -1;  // 传入的参数有误
    }else{
        L->data[L->len] = x;
        L->len++;
        return x;   // 操作成功，返回插入的元素的值
    }
}

// 移除表的最后一项
int Pop_SeqList(SeqList *L){
    if(L == NULL || L->len == 0){
        return -1;  // 传入的参数有误
    }else{
        int val = L->data[L->len-1];
        L->data[L->len-1] = 0;
        L->len--;
        return val; // 操作成功，返回移除的元素的值
    }
}

// 插入元素
int Insert_SeqList(SeqList *L, int index, int x){
    if(L == NULL || index >= MAXSIZE || L->len == MAXSIZE){
        return -1;  // 传入的参数有误
    }else if(L->len == 0 || index > L->len-1){
        L->data[index] = x;
        L->len = index+1;
        return 0;   // 操作成功
    }else{
        for(int i=L->len-1; i>=index; i--){
            L->data[i+1] = L->data[i];
        }
        L->data[index] = x;
        L->len++;
        return 0;   // 操作成功
    }
}

// 删除元素
int Delete_SeqList(SeqList *L, int index){
    if(L == NULL || L->len == 0 || index >= L->len){
        return -1;  // 传入的参数有误
    }else if(index == L->len-1){
        L->data[index] = 0;
        L->len--;
        return 0;   // 操作成功
    }else{
        for(int i=index+1; i<L->len; i++){
            L->data[i-1] = L->data[i];
        }
        L->len--;
        return 0;   // 操作成功
    }
}

// 将顺序表(a1, a2, ..., an)以a1为界重排：a1前面的值均比a1小; a1后面的值均比a1大
int Part_SeqList(SeqList *L){
    if(L == NULL || L->len == 0){
        return -1;  // 传入的参数有误
    }else{
        int base = L->data[0];
        int temp;
        for(int i=1; i<L->len; i++){
            if(L->data[i] < base){
                temp = L->data[i];
                for(int j=i-1; j>=0; j--){
                    L->data[j+1] = L->data[j];
                }
                L->data[0] = temp;
            }
        }
        return 0;   // 操作成功
    }
}

// 合并两个增序列为新的增序列
SeqList *Merge_SeqList(SeqList *L1, SeqList *L2){
    if(L1 == NULL || L2 == NULL || L1->len ==0 || L2->len == 0 || L1->len+L2->len > MAXSIZE){
        return NULL;    // 传入的参数有误
    }else{
        SeqList *L = Init_SeqList();
        if(L == NULL){
            return NULL;    // 申请内存失败
        }
        int i=0, j=0, k=0;
        while(i<L1->len && j<L2->len){
            if(L1->data[i] < L2->data[j]){
                L->data[k++] = L1->data[i++];
                L->len++;
            }else{
                L->data[k++] = L2->data[j++];
                L->len++;
            }
        }
        for(; i<L1->len; i++, k++){
            L->data[k] = L1->data[i];
            L->len++;
        }
        for(; j<L2->len; j++, k++){
            L->data[k] = L2->data[j];
            L->len++;
        }
        return L;   // 返回新增序列的首地址
    }
}

// 比较两个顺序表
int Compare_SeqList(SeqList *L1, SeqList *L2){
    if(L1 == NULL || L2 == NULL || L1->len == 0 || L2->len == 0){
        return 0;   // 传参有误
    }else{
        int mark=0;
        while(L1->data[mark] == L2->data[mark] && (mark < L1->len || mark < L2->len)){
            mark++;
        }
        if(mark == L1->len && mark == L2->len){
            return 0;   // L1 == L2
        }else if((mark == L1->len && mark < L2->len) || (mark < L1->len && mark < L2->len && L1->data[mark] < L2->data[mark])){
            return -1;  // L1 < L2
        }else{
            return 1;   // L1 > L2
        }
    }
}

// 查找最大值
int Max_SeqList(SeqList *L){
    if(L == NULL || L->len == 0){
        return 0;   // 传入的参数有误
    }else{
        int maxVal = L->data[0];
        for(int i=1; i<L->len; i++){
            if(L->data[i] > maxVal){
                maxVal = L->data[i];
            }
        }
        return maxVal;  // 返回最大值
    }
}

// 查找最小值
int Min_SeqList(SeqList *L){
    if(L == NULL || L->len == 0){
        return 0;   // 传入的参数有误
    }else{
        int minVal = L->data[0];
        for(int i=1; i<L->len; i++){
            if(L->data[i] < minVal){
                minVal = L->data[i];
            }
        }
        return minVal;  // 返回最小值
    }
}

// 排序，rev参数如果为1表示倒序排列，为0则表示正序排列
int Sort_SeqList(SeqList *L, int rev){
    if(L == NULL || L->len == 0){
        return -1;  // 传入的参数有误
    }else{
        for(int i=0; i<L->len-1; i++){
            int sorted = 1;
            for(int j=0; j<L->len-1-i; j++){
                if(!rev){
                    if(L->data[j] > L->data[j+1]){
                        sorted = 0;
                        int temp = L->data[j];
                        L->data[j] = L->data[j+1];
                        L->data[j+1] = temp;
                    }
                }else{
                    if(L->data[j] < L->data[j+1]){
                        sorted = 0;
                        int temp = L->data[j];
                        L->data[j] = L->data[j+1];
                        L->data[j+1] = temp;
                    }
                }
            }
            if(sorted){
                break;
            }
        }
        return 0;   // 操作成功
    }
}

// 逆置元素
void Reverse_SeqList(SeqList *L){
    for(int i=0; i<L->len/2; i++){
        int tmp = L->data[i];
        L->data[i] = L->data[L->len-1-i];
        L->data[L->len-1-i] = tmp;
    }
}
</script></code></pre>
