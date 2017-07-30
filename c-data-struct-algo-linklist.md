---
title: (c语言)数据结构与算法 - 单链表
date: 2017-07-27 12:36:00
categories:
- c
- 数据结构与算法
tags:
- c
- 数据结构与算法
keywords: c语言 数据结构与算法 单链表
---

> 
`单向链表(单链表)`是链表的一种，其特点是链表的链接方向是单向的，对链表的访问要通过顺序读取从头部开始;
单向链表的数据结构可以分为两部分：`数据域`和`指针域`，数据域存储数据，指针域指向下一个储存节点的地址

<!-- more -->

## 单链表
由于`顺序表`的存贮特点是用物理上的相邻实现了逻辑上的相邻，它要求用连续的存储单元顺序存储线性表中各元素，因此，对`顺序表插入、删除`时需要通过`移动数据元素`来实现，影响了运行效率;
而线性表`链式存储结构`，它不需要用地址连续的存储单元来实现，因为它不要求逻辑上相邻的两个数据元素物理上也相邻，它是通过“链”建立起数据元素之间的逻辑关系来，因此对线性表的`插入、删除不需要移动数据元素`，只需要`改变指针的指向`

**`链表`**
`链表(Linked list)`是一种常见的基础数据结构，是一种`线性表`，但是并不会按线性的顺序存储数据，而是在每一个节点里存到下一个节点的`指针(Pointer)`;
由于不必须按顺序存储，`链表`在`插入`的时候可以达到`O(1)`的复杂度，比另一种线性表顺序表快得多，但是`查找`一个节点或者`访问`特定编号的节点则需要`O(n)`的时间，而`顺序表`相应的时间复杂度分别是`O(logn)`和`O(1)`

**`单链表`**
链表是通过一组任意的存储单元来存储线性表中的数据元素的，那么怎样表示出数据元素之间的线性关系呢？
为建立起数据元素之间的线性关系，对每个数据元素`ai`，除了存放数据元素的自身的信息`ai`之外，还需要和`ai`一起存放其`后继ai+1所在的存贮单元的地址`，这两部分信息组成一个`结点`
`存放数据元素信息`的称为`数据域`，`存放其后继地址`的称为`指针域`;
因此`n`个元素的线性表通过每个结点的指针域拉成了一个“链子”，称之为`链表`;
因为每个结点中`只有一个指向后继的指针`，所以称其为`单链表`

**头指针、头结点、首元结点**
链表中`第一个结点的存储位置`叫做`头指针`，如果链表有`头结点`，那么`头指针就是指向头结点数据域的指针`
`头结点`是为了操作的统一与方便而设立的，放在`第一个元素结点之前`，其`数据域一般无意义`(当然有些情况下也可存放链表的长度、用做监视哨等等)
`首元结点`是`第一个元素结点`，如果`存在头结点`，那就是`头结点后边的第一个结点`

**头结点不是链表所必需的**  头结点的存在只是为了操作的方便

## c语言实现单链表
**`LinkList.h`**
<pre><code class="language-c line-numbers"><script type="text/plain">#ifndef __LINKLIST_H
#define __LINKLIST_H

#include <stdbool.h>

/**
 * LNode: 结点类型
 * LinkList: 头指针类型，也就是链表
 * data: 数据域, 这里假设类型为int
 * next: 指针域, 指向下一个结点
**/
typedef struct LNode{
    int data;
    struct LNode *next;
} LNode, *LinkList;

/**
 * 创建一个单链表(尾插法)
 * @param void      不接受参数
 * @return LinkList 返回该链表的头指针
**/
extern LinkList createLinkList(void);

/**
 * 创建一个单链表(头插法)
 * @param void      不接受参数
 * @return LinkList 返回该链表的头指针
**/
extern LinkList createLinkList2(void);

/**
 * 获取链表长度
 * @param L     要操作的单链表L
 * @return int  返回L的长度
**/
extern int lengthLinkList(LinkList L);

/**
 * 清空单链表L的所有元素(除了头结点)
 * @param L     要操作的单链表L
 * @return void 无返回值
**/
extern void clearLinkList(LinkList L);

/**
 * 销毁单链表L
 * @param *pL   二级指针: 指向单链表L的指针
 * @return void 无返回值
**/
extern void destroyLinkList(LinkList *pL);

/**
 * 打印单链表L
 * @param L     要操作的单链表L
 * @return void 无返回值
**/
extern void printLinkList(LinkList L);

/**
 * 判断单链表L是否为空
 * @param L     要操作的单链表L
 * @return bool true: 空; false: 非空
**/
extern bool isEmptyLinkList(LinkList L);

/**
 * 在索引值为index的位置插入一个值为val的新结点
 * @param L     要操作的单链表L
 * @param index 插入的位置
 * @param val   索引值index的数据域
 * @return bool true: 插入成功; false: 参数有误
**/
extern bool insertLinkList(LinkList L, int index, int val);

/**
 * 删除索引值为index的结点
 * @param L     要操作的单链表L
 * @param index 索引值index
 * @return bool true: 删除成功; false: 参数有误
**/
extern bool deleteLinkList(LinkList L, int index);

/**
 * 删除第一个值为val的结点
 * @param L     要操作的单链表L
 * @param val   值val
 * @return bool true: 删除成功; false: 删除失败
**/
extern bool deleteValLinkList(LinkList L, int val);

/**
 * 获取索引值为index的结点的数据域
 * @param L     要操作的单链表L
 * @param index 索引值index
 * @return int  返回数据域的值
**/
extern int getLinkList(LinkList L, int index);

/**
 * 按值查找，返回第一次出现的索引值index
 * @param L     要操作的单链表L
 * @param val   所查找的值
 * @return int  成功: 返回第一次出现的索引值index; 失败: 返回-1
**/
extern int locateLinkList(LinkList L, int val);

/**
 * 修改索引值为index上的值为val
 * @param L     要操作的单链表
 * @param index 索引值index
 * @param val   新值val
 * @return bool true: 修改成功; false: 参数有误
**/
extern bool modifyLinkList(LinkList L, int index, int val);

/**
 * 在链表后添加一个结点，值为val
 * @param L     要操作的单链表L
 * @param val   值val
 * @return int  插入成功则返回所插入的值val
**/
extern int appendLinkList(LinkList L, int val);

/**
 * 移除链表的最后一个结点
 * @param L     要操作的单链表L
 * @return int  被删除的结点的值val
**/
extern int popLinkList(LinkList L);

/**
 * 排序(冒泡排序)
 * @param L     要操作的单链表L
 * @param rev   0:升序排列; 1:降序排列
 * @return void 无返回值
**/
extern void sortLinkList(LinkList L, int rev);

/**
 * 逆置单链表
 * @param L     要操作的单链表L
 * @return void 无返回值
**/
extern void reverseLinkList(LinkList L);

#endif
</script></code></pre>

**`LinkList.c`**
<pre><code class="language-c line-numbers"><script type="text/plain">#include "LinkList.h"
#include <stdio.h>
#include <stdlib.h>

LinkList createLinkList(void){
    LinkList L = NULL;
    LNode *LHead = (LNode *)malloc(sizeof(LNode));
    LHead->data = 0;
    LHead->next = NULL;
    L = LHead;
    LNode *LTail = LHead;
    int len;
    printf("请输入链表长度: ");
    scanf("%d", &len);
    printf("请输入相应的元素: ");
    for(int i=0; i<len; i++){
        int val;
        scanf("%d", &val);
        LNode *LNew = (LNode *)malloc(sizeof(LNode));
        LNew->data = val;
        LNew->next = NULL;
        LTail->next = LNew;
        LTail = LNew;
    }
    return L;
}

LinkList createLinkList2(void){
    LNode *LHead = (LNode *)malloc(sizeof(LNode));
    LHead->data = 0;
    LHead->next = NULL;
    LinkList L = LHead;
    int len;
    printf("请输入链表长度: ");
    scanf("%d", &len);
    printf("请输入相应的元素: ");
    for(int i=0; i<len; i++){
        int val;
        scanf("%d", &val);
        LNode *LNew = (LNode *)malloc(sizeof(LNode));
        LNew->data = val;
        LNew->next = L->next;
        L->next = LNew;
    }
    return L;
}

int lengthLinkList(LinkList L){
    int len = 0;
    LNode *p = L->next;
    for(; p!=NULL; len++, p=p->next);
    return len;
}

void clearLinkList(LinkList L){
    if(L == NULL || L->next == NULL){
        return;
    }else{
        LNode *p = L;
        while(p->next != NULL){
            LNode *tmp = p->next->next;
            free(p->next);
            p->next = tmp;
        }
        return;
    }
}

void destroyLinkList(LinkList *pL){
    if(*pL == NULL){
        return;
    }else{
        LNode *p = *pL;
        while(p->next != NULL){
            LNode *tmp = p->next->next;
            free(p->next);
            p->next = tmp;
        }
        free(*pL);
        *pL = NULL;
        return;
    }
}

void printLinkList(LinkList L){
    if(L->next == NULL){
        printf("The LinkList is empty!\n");
        return;
    }else{
        printf("length:%d { ", lengthLinkList(L));
        for(LNode *p=L->next; p!=NULL; p=p->next){
            printf("%d, ", p->data);
        }
        printf("\b\b }\n");
        return;
    }
}

bool isEmptyLinkList(LinkList L){
    if(L->next == NULL || L == NULL){
        return true;
    }else{
        return false;
    }
}

bool insertLinkList(LinkList L, int index, int val){
    int len = lengthLinkList(L);
    if(L == NULL || index < 0 || index > len){
        return false;
    }else{
        LNode *LNew = (LNode *)malloc(sizeof(LNode));
        LNew->data = val;
        LNode *p = L;
        for(int i=0; i<index; i++, p=p->next);
        LNode *tmp = p->next;
        p->next = LNew;
        LNew->next = tmp;
        return true;
    }
}

bool deleteLinkList(LinkList L, int index){
    int len = lengthLinkList(L);
    if(L == NULL || index < 0 || index >= len){
        return false;
    }else{
        LNode *p = L;
        for(int i=0; i<index; i++, p=p->next);
        LNode *tmp = p->next->next;
        free(p->next);
        p->next = tmp;
        return true;
    }
}

bool deleteValLinkList(LinkList L, int val){
    if(L->next == NULL){
        return false;
    }else{
        LNode *p = L;
        for(; p->next->data!=val && p->next->next!=NULL; p=p->next);
        if(p->next->data != val){
            return false;
        }else{
            LNode *tmp = p->next->next;
            free(p->next);
            p->next = tmp;
            return true;
        }
    }
}

int getLinkList(LinkList L, int index){
    int len = lengthLinkList(L);
    if(index < 0 || index >= len){
        exit(1);
    }else{
        LNode *p = L->next;
        for(int i=0; i<index; i++, p=p->next);
        return p->data;
    }
}

int locateLinkList(LinkList L, int val){
    LNode *p = L->next;
    for(int i=0; p!=NULL; i++, p=p->next){
        if(p->data == val){
            return i;
        }
    }
    return -1;
}

bool modifyLinkList(LinkList L, int index, int val){
    int len = lengthLinkList(L);
    if(index < 0 || index >= len){
        return false;
    }else{
        LNode *p = L->next;
        for(int i=0; i<index; i++, p=p->next);
        p->data = val;
        return true;
    }
}

int appendLinkList(LinkList L, int val){
    if(L == NULL){
        exit(1);
    }
    LNode *p = L->next;
    for(; p->next!=NULL; p=p->next);
    LNode *LNew = (LNode *)malloc(sizeof(LNode));
    LNew->data = val;
    LNew->next = NULL;
    p->next = LNew;
    return val;
}

int popLinkList(LinkList L){
    if(L == NULL || L->next == NULL){
        exit(EXIT_FAILURE);
    }
    LNode *p = L->next;
    for(; p->next->next!=NULL; p=p->next);
    int val = p->next->data;
    free(p->next);
    p->next=NULL;
    return val;
}

void sortLinkList(LinkList L, int rev){
    if(L == NULL || L->next == NULL){
        return;
    }else{
        int len = lengthLinkList(L);
        for(int i=0; i<len-1; i++){
            bool sorted = true;
            LNode *p = L->next;
            for(int j=0; j<len-1-i; j++, p=p->next){
                if(!rev){
                    if(p->data > p->next->data){
                        sorted = false;
                        int tmp = p->data;
                        p->data = p->next->data;
                        p->next->data = tmp;
                    }
                }else{
                    if(p->data < p->next->data){
                        sorted = false;
                        int tmp = p->data;
                        p->data = p->next->data;
                        p->next->data = tmp;
                    }
                }
            }
            if(sorted){
                break;
            }
        }
        return;
    }
}

void reverseLinkList(LinkList L){
    LNode *p = L->next;
    L->next = NULL;
    while(p != NULL){
        LNode *tmp = p->next;
        p->next = L->next;
        L->next = p;
        p = tmp;
    }
}
</script></code></pre>
