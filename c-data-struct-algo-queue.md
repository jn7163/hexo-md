---
title: (c语言)数据结构与算法 - 队列
date: 2017-07-29 09:59:00
categories:
- c
- 数据结构与算法
tags:
- c
- 数据结构与算法
keywords: c语言 数据结构与算法 队列
---

> 
`队列(queue)`，是`先进先出(FIFO, First-In-First-Out)的线性表`；
在具体应用中通常用`链表`或者`数组`来实现，队列只允许在`后端(称为rear)进行插入操作`，在`前端(称为front)进行删除操作`；

<!-- more -->

## 队列的定义
前面所讲的栈是一种“后进先出”的数据结构，而在实际问题中还经常使用一种“先进先出”的数据结构：
即插入在表一端进行，而删除在表的另一端进行，我们将这种数据结构称为队或队列；
把允许插入的一端叫`队尾(rear)`，把允许删除的一端叫`队头(front)`

**基本运算**
- `Init_Queue(q)`：初始化
初始条件：队q不存在
操作结果：构造了一个空队

- `In_Queue(q, x)`：入队
初始条件：队q存在
操作结果：对已存在的队列q，插入一个元素x到队尾，队发生变化

- `Out_Queue(q, x)`：出队
初始条件：队q存在且非空
操作结果：删除队首元素，并返回其值，队发生变化

- `Empty_Queue(q)`：判空
初始条件：队q存在
操作结果：若q为空队则返回为1，否则返回为0

## 队列的实现
### 顺序队
**`顺序存储`的队称为`顺序队`**
因为队的队头和队尾都是活动的，因此，除了队列的数据区外还有队头、队尾两个指针；顺序队的类型定义如下：
<pre><code class="language-c line-numbers"><script type="text/plain">typedef struct{
    ElemType data[MAXSIZE];
    int front;
    int rear;
    int size;
} SeqQueue;
</script></code></pre>

定义一个指向队的指针变量：`SeqQueue *q;`
申请一个顺序队的存储空间：`q = (SeqQueue *)malloc(sizeof(SeqQueue));`
队列的数据区为：`q->data[0] ~ q->data[MAXSIZE-1]`
队头指针：`q->front`
队尾指针：`q->rear`

设`队头指针`指向`队头元素前面一个位置`，`队尾指针`指向`队尾元素`(这样的设置是为了某些运算的方便，并不是唯一的方法)

置空队则为：`q->front = q->rear = -1;`

**`SeqQueue.h`**
<pre><code class="language-c line-numbers"><script type="text/plain">#ifndef __SEQQUEUE_H
#define __SEQQUEUE_H
#include <stdbool.h>

#define MAXSIZE 1024
typedef int ElemType;
typedef struct{
    ElemType data[MAXSIZE];
    int front;
    int rear;
    int size;
} SeqQueue;

// 初始化
extern SeqQueue *init_queue(void);

// 入队
extern bool in_queue(SeqQueue *q, ElemType val);

// 出队
extern bool out_queue(SeqQueue *q, ElemType *val);

// 判空
extern bool isEmpty_queue(SeqQueue *q);

#endif
</script></code></pre>

**`SeqQueue.c`**
<pre><code class="language-c line-numbers"><script type="text/plain">#include "SeqQueue.h"
#include <stdlib.h>

SeqQueue *init_queue(void){
    SeqQueue *q = (SeqQueue *)malloc(sizeof(SeqQueue));
    q->front = q->rear = -1;
    q->size = 0;
    return q;
}

bool in_queue(SeqQueue *q, ElemType val){
    if(q == NULL || q->size == MAXSIZE){
        return false;
    }else{
        q->rear = (q->rear+1) % MAXSIZE;
        q->data[q->rear] = val;
        q->size++;
        return true;
    }
}

bool out_queue(SeqQueue *q, ElemType *val){
    if(q == NULL || q->size == 0){
        return false;
    }else{
        q->front = (q->front+1) % MAXSIZE;
        *val = q->data[q->front];
        q->size--;
        return true;
    }
}

bool isEmpty_queue(SeqQueue *q){
    if(q == NULL || q->size == 0){
        return true;
    }else{
        return false;
    }
}
</script></code></pre>

### 链队
**`链式存储`的队称为`链队`**
和链栈类似，用单链表来实现链队，根据队的FIFO原则，为了操作上的方便，我们分别需要一个头指针和尾指针：
<pre><code class="language-c line-numbers"><script type="text/plain">typedef struct LNode{
    ElemType data;
    struct LNode *next;
} LNode;
typedef struct LinkQueue{
    LNode *front;
    LNode *rear;
} LinkQueue;
</script></code></pre>

**`LinkQueue.h`**
<pre><code class="language-c line-numbers"><script type="text/plain">#ifndef __LINKQUEUE_H
#define __LINKQUEUE_H
#include <stdbool.h>

typedef int ElemType;
typedef struct LNode{
    ElemType data;
    struct LNode *next;
} LNode;
typedef struct LinkQueue{
    LNode *front;
    LNode *rear;
} LinkQueue;

// 初始化
extern LinkQueue *init_queue(void);

// 判空
extern bool isEmpty_queue(LinkQueue *q);

// 入队
extern void in_queue(LinkQueue *q, ElemType val);

// 出队
extern bool out_queue(LinkQueue *q, ElemType *val);

#endif
</script></code></pre>

**`LinkQueue.c`**
<pre><code class="language-c line-numbers"><script type="text/plain">#include "LinkQueue.h"
#include <stdlib.h>

LinkQueue *init_queue(void){
    LinkQueue *q = (LinkQueue *)malloc(sizeof(LinkQueue));
    q->front = q->rear = (LNode *)malloc(sizeof(LNode));
    q->front->next = NULL;
    return q;
}

bool isEmpty_queue(LinkQueue *q){
    if(q == NULL || q->front == q->rear){
        return true;
    }else{
        return false;
    }
}

void in_queue(LinkQueue *q, ElemType val){
    LNode *LNew = (LNode *)malloc(sizeof(LNode));
    LNew->data = val;
    LNew->next = NULL;
    q->rear->next = LNew;
    q->rear = LNew;
}

bool out_queue(LinkQueue *q, ElemType *val){
    if(q == NULL || q->front == q->rear){
        return false;
    }else{
        LNode *tmp = q->front->next->next;
        *val = q->front->next->data;
        free(q->front->next);
        q->front->next = tmp;
        if(q->front->next == NULL){
            q->rear = q->front;
        }
        return true;
    }
}
</script></code></pre>
