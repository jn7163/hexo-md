---
title: (c语言)数据结构与算法 - 栈
date: 2017-07-28 12:56:00
categories:
- c
- 数据结构与算法
tags:
- c
- 数据结构与算法
keywords: c语言 数据结构与算法 栈 顺序栈 链栈
---

> 
`栈(stack)`又名堆栈，它是一种`运算受限的线性表`。其限制是仅允许在表的一端进行插入和删除运算。这一端被称为`栈顶`，相对地，把另一端称为`栈底`。
向一个栈`插入新元素`又称作`进栈`、`入栈`或`压栈`，它是把新元素放到栈顶元素的上面，使之成为新的栈顶元素；
从一个栈`删除元素`又称作`出栈`或`退栈`，它是把栈顶元素删除掉，使其相邻的元素成为新的栈顶元素

<!-- more -->

## 栈的定义
`栈`和`队列`是在软件设计中常用的两种数据结构，它们的`逻辑结构`和`线性表`相同;
其特点在于运算受到了限制：`栈`按`后进先出`的规则进行操作，`队`按`先进先出`的规则进行操作，故称运算受限制的线性表

**栈是限制在表的一端进行插入和删除的线性表**
允许插入、删除的这一端称为`栈顶`，另一个固定端称为`栈底`，当表中没有元素时称为`空栈`

栈又称为`后进先出的线性表(Last In First Out)`，简称`LIFO表`

**基本运算**

- `Init_Stack()`：栈初始化
初始条件：栈s不存在
操作结果：构造了一个空栈

- `Empty_Stack(s)`：判栈空
初始条件：栈s已存在
操作结果：若s为空栈返回为1，否则返回为0

- `Push_Stack(s, x)`：入栈
初始条件：栈s已存在
操作结果：在栈s的顶部插入一个新元素x，x成为新的栈顶元素

- `Pop_Stack(s)`：出栈
初始条件：栈s存在且非空
操作结果：栈s的顶部元素从栈中删除，栈中少了一个元素

- `Top_Stack(s)`：读栈顶元素
初始条件：栈s存在且非空
操作结果：栈顶元素作为结果返回，栈不变化

## 栈的实现
> 由于栈是运算受限的线性表，因此线性表的存储结构对栈也是适用的，只是操作不同而已

### 顺序栈
**利用`顺序存储`方式实现的栈称为`顺序栈`**
类似于顺序表的定义，栈中的数据元素用一个预设的足够长度的一维数组来实现：`ElemType data[MAXSIZE]`，栈底位置可以设置在数组的任一个端点，而栈顶是随着插入和删除而变化的，用一个`int top`来作为`栈顶的指针`，指明当前栈顶的位置，同样将`data`和`top`封装在一个结构中，顺序栈的类型描述如下：
<pre><code class="language-c line-numbers"><script type="text/plain">typedef struct{
    ElemType data[MAXSIZE];
    int top;
} SeqStack;
</script></code></pre>

定义一个指向顺序栈的指针`SeqStack *s`；
通常将0下标端设为栈底，这样空栈时栈顶指针`s->top=-1`；
入栈时，栈顶指针加1，即`s->top++`；出栈时，栈顶指针减1，即`s->top--`；

**`SeqStack.h`**
<pre><code class="language-c line-numbers"><script type="text/plain">#ifndef __SEQSTACK_H
#define __SEQSTACK_H
#include <stdbool.h>

#define MAXSIZE 1024
typedef int ElemType;
typedef struct{
    ElemType data[MAXSIZE];
    int top;
} SeqStack;

/**
 * 初始化
 * @param void          不接受参数
 * @return SeqStack *   返回指向SeqStack的指针
**/
extern SeqStack *init_stack(void);

/**
 * 判断s是否为空
 * @param s     要操作的顺序栈s
 * @return bool true: 空; false: 非空
**/
extern bool empty_stack(SeqStack *s);

/**
 * 入栈
 * @param s     要操作的顺序栈s
 * @param val   入栈的元素val
 * @return bool true: 入栈成功; false: 传参有误或空间不足
**/
extern bool push_stack(SeqStack *s, ElemType val);

/**
 * 出栈
 * @param s     要操作的顺序栈s
 * @param val   保存弹出元素的指针
 * @return bool true: 出栈成功; false: 传参有误或s为空栈
**/
extern bool pop_stack(SeqStack *s, ElemType *val);

/**
 * 读取栈顶元素
 * @param s         要操作的顺序栈s
 * @return ElemType 返回栈顶元素的值，若s为空栈则返回0
**/
extern ElemType top_stack(SeqStack *s);

#endif
</script></code></pre>

**`SeqStack.c`**
<pre><code class="language-c line-numbers"><script type="text/plain">#include "SeqStack.h"
#include <stdlib.h>

SeqStack *init_stack(void){
    SeqStack *s = (SeqStack *)malloc(sizeof(SeqStack));
    s->top = -1;
    return s;
}

bool empty_stack(SeqStack *s){
    if(s == NULL || s->top == -1){
        return true;
    }else{
        return false;
    }
}

bool push_stack(SeqStack *s, ElemType val){
    if(s == NULL || s->top == MAXSIZE-1){
        return false;
    }else{
        s->top++;
        s->data[s->top] = val;
        return true;
    }
}

bool pop_stack(SeqStack *s, ElemType *val){
    if(s == NULL || s->top == -1){
        return false;
    }else{
        *val = s->data[s->top];
        s->top--;
        return true;
    }
}

ElemType top_stack(SeqStack *s){
    if(s == NULL || s->top == -1){
        return 0;
    }else{
        return s->data[s->top];
    }
}
</script></code></pre>

### 链栈
**用`链式存储`结构实现的栈称为`链栈`**
通常链栈用单链表表示，因此其结点结构与单链表的结构相同，在此用`LinkStack`表示，即有：
<pre><code class="language-c line-numbers"><script type="text/plain">typedef struct SNode{
    ElemType data;
    struct SNode *next;
} SNode, *LinkStack;
</script></code></pre>

`LinkStack top`：top为栈顶指针
因为栈中的主要运算是在栈顶插入、删除，显然在链表的头部做栈顶是最方便的，而且没有必要像单链表那样为了运算方便附加一个头结点
**`LinkStack.h`**
<pre><code class="language-c line-numbers"><script type="text/plain">#ifndef __LINKSTACK_H
#define __LINKSTACK_H
#include <stdbool.h>

typedef int ElemType;
typedef struct SNode{
    ElemType data;
    struct SNode *next;
} SNode, *LinkStack;

/**
 * 初始化
 * @param void          不接受参数
 * @return LinkStack    返回栈顶指针top
**/
extern LinkStack init_stack(void);

/**
 * 判断链栈top是否为空
 * @param top   要操作的链栈top
 * @return bool true: 空; false: 非空
**/
extern bool empty_stack(LinkStack top);

/**
 * 入栈
 * @param top   要操作的链栈指针top(二级指针)
 * @param val   入栈的元素
 * @return bool true: 操作成功; false: 操作失败
**/
extern bool push_stack(LinkStack *top, ElemType val);

/**
 * 出栈
 * @param top   要操作的链栈指针top(二级指针)
 * @param val   保存弹出元素的指针
 * @return bool true: 操作成功; false: 操作失败
**/
extern bool pop_stack(LinkStack *top, ElemType *val);

/**
 * 读取栈顶元素
 * @param top       要操作的链栈top
 * @return ElemType 返回元素的值，若栈为空则返回0
**/
extern ElemType top_stack(LinkStack top);

#endif
</script></code></pre>

**`LinkStack.c`**
<pre><code class="language-c line-numbers"><script type="text/plain">#include "LinkStack.h"
#include <stdlib.h>

LinkStack init_stack(void){
    return NULL;
}

bool empty_stack(LinkStack top){
    if(top == NULL){
        return true;
    }else{
        return false;
    }
}

bool push_stack(LinkStack *top, ElemType val){
    if(top == NULL){
        return false;
    }else{
        SNode *node = (SNode *)malloc(sizeof(SNode));
        node->data = val;
        node->next = *top;
        *top = node;
        return true;
    }
}

bool pop_stack(LinkStack *top, ElemType *val){
    if(top == NULL || *top == NULL){
        return false;
    }else{
        *val = (*top)->data;
        LinkStack newtop = (*top)->next;
        free(*top);
        *top = newtop;
        return true;
    }
}

ElemType top_stack(LinkStack top){
    if(top == NULL){
        return 0;
    }else{
        return top->data;
    }
}
</script></code></pre>

## 栈的简单应用
**将一个十进制数转换为任意进制的数字**
简单分析：十进制转其他进制的转换方法利用辗转相除法，计算产生的数字是从低位开始的，而输出的数字是从高位开始的，适合使用stack这种数据结构

**利用顺序栈实现**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include "SeqStack.h"

void convert(int dec, int base){
    SeqStack *s = init_stack();
    printf("dec(%d) >> base.%d(", dec, base);
    while(dec != 0){
        push_stack(s, dec % base);
        dec /= base;
    }
    while(!empty_stack(s)){
        int val;
        pop_stack(s, &val);
        printf("%d", val);
    }
    printf(")\n");
}

int main(void){
    convert(100, 2);
    convert(100, 8);
    convert(100, 16);
    convert(100, 32);
    return 0;
}
</script></code></pre>

**利用链栈实现**
<pre><code class="language-c line-numbers"><script type="text/plain">#include <stdio.h>
#include "LinkStack.h"

void convert(int dec, int base){
    LinkStack top = init_stack();
    printf("dec(%d) >> base.%d(", dec, base);
    while(dec != 0){
        push_stack(&top, dec % base);
        dec /= base;
    }
    while(!empty_stack(top)){
        int val;
        pop_stack(&top, &val);
        printf("%d", val);
    }
    printf(")\n");
}

int main(void){
    convert(100, 2);
    convert(100, 8);
    convert(100, 16);
    convert(100, 32);
    return 0;
}
</script></code></pre>
