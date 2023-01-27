---
title: "队列 ADT 【数据结构与算法分析 c 语言描述】"
date: 2018-12-26 00:42:00
slug: "queue-ADT-[data-structure-and-algorithm-analysis-c-language]"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

## 1. 队列模型
![file](https://cdn.learnku.com/uploads/images/201812/25/23174/nmCTCOqQ8n.png!large)
## 2. 队列数组实现
* 数组实现在创建时候需要比链表多传入一个 max_size ,即需要指定数组的长度。
* 数组存放队列内容，size - 队列长度， front - 队头，rear - 队尾，同时都与数组下标对应。
* `enqueue `值存入数组 `rear + 1`位置，同时 `rear + 1`，`size + 1`，如果超过右边界，则从 0 开始。
* `dequeue`返回 `front + 1`位置的值，同时 `front + 1`，`size - 1`如果超过右边界，则从 0 开始。
结构体定义
```c
typedef int element_type;
typedef struct queue_record
{
    int capacity; //数组长度,队列最大长度
    int size; //队列长度
    int front; //队头
    int rear; //队尾
    element_type *arr;
} *queue;
```
函数定义
```c
queue create_queue(int max_size);
void make_empty(queue q);
int is_full(queue q);
int is_empty(queue q);
void enqueue(element_type x, queue q);
void dequeue(queue q);
element_type front(queue q);
element_type front_and_dequeue(queue q);
void dispose_queue(queue *q);
```
完整代码：

```c
/**
 * 队列ADT-数组实现
 */

#include <stdio.h>
#include <stdlib.h>

#define MAX_QUEUE_SIZE (5)

#define error(str) fatal_error(str)
#define fatal_error(str) fprintf(stderr, "%s\n", str),exit(1)

typedef int element_type;
typedef struct queue_record
{
    int capacity;
    int size; //队列长度
    int front; //队头
    int rear; //队尾
    element_type *arr;
} *queue;

queue create_queue(int max_size);
void make_empty(queue q);
int is_full(queue q);
int is_empty(queue q);
void enqueue(element_type x, queue q);
void dequeue(queue q);
element_type front(queue q);
element_type front_and_dequeue(queue q);
void dispose_queue(queue *q);
void print_queue(queue q);

void test();

queue create_queue(int max_size)
{
    queue q;
    if (max_size < MAX_QUEUE_SIZE)
        error("max_len is too small.");
    q = (queue)malloc(sizeof(struct queue_record));
    if (NULL == q)
        fatal_error("out of space.");

    q->capacity = max_size;
    q->arr = (element_type*)malloc(sizeof(element_type) * max_size);

    make_empty(q);

    return q;
}

void make_empty(queue q)
{
    q->size = 0;
    q->front = 1;
    q->rear = 0;
}

int is_full(queue q)
{
    return q->size == q->capacity;
}

int is_empty(queue q)
{
    return q->size == 0;
}

void enqueue(element_type x, queue q)
{
    if (NULL == q)
        fatal_error("Must create queue");
    if (is_full(q)){
        error("Full queue");
    } else {
        q->size++;
        if (++q->rear == q->capacity)
            q->rear = 0;
        q->arr[q->rear] = x;
    }
}

void dequeue(queue q)
{
    if (NULL == q)
        fatal_error("Must create queue");
    if (is_empty(q)){
        error("Empty queue");
    } else {
        q->size--;
        if (++q->front == q->capacity)
            q->front = 0;
    }
}

element_type front(queue q)
{
    if (!is_empty(q))
        return q->arr[q->front];
    error("empty queue");
    return 0;
}

element_type front_and_dequeue(queue q)
{
    if (!is_empty(q)) {
        element_type ret = q->arr[q->front];
        dequeue(q);
        return ret;
    }
    error("empty queue");
    return 0;
}

void print_queue(queue q)
{
    int i;
    for (i = q->front; i <= q->size; i++) {
        printf("%d\t", q->arr[i]);
    }
    printf("\n");
}

void dispose_queue(queue *q)
{
    if ( NULL != (*q) ) {
        free((*q)->arr);
        free((*q));
        (*q) = NULL;
    }
}

void test()
{
    queue q;
    int act;
    while (1) {
        printf("\n\tplease input a intger and choose a option.\n");
        printf("\t\t1.create a empty queue.\n");
        printf("\t\t2.enqueue a element to queue.\n");
        printf("\t\t3.dequeue a element from queue.\n");
        printf("\t\t4.get the front element of queue.\n");
        printf("\t\t5.print queue.\n");
        printf("\t\t6.dispose queue.\n");
        printf("\t\t0.exit\n");

        scanf("%d", &act);
        if (act == 0)
            break;
        switch(act) {
            case 1:{
                int max_len;
                printf("please input max_len of stack.\n");
                scanf("%d", &max_len);
                q = create_queue(max_len);
                printf("create queue success\n");
                break;
            }
            case 2: {
                int x;
                printf("please input a intger.\n");
                scanf("%d", &x);
                enqueue(x, q);
                printf("enqueue success\n");
                print_queue(q);
                break;
            }
            case 3:
                dequeue(q);
                printf("dequeue a element success\n");
                break;
            case 4:
                printf("queue front:%d\n", front(q));
                break;
            case 5:
                print_queue(q);
                break;
            case 6:
                dispose_queue(&q);
                printf("dispose queue success\n");
                break;
        }
    }
}

int main(int argc, char const *argv[])
{
    test();
    return 0;
}
```

## 3. 队列链表实现
* 链表每个节点表示队列的一个元素
* 入队 `enqueue` 新节点插入到末尾。
* 出队`dequeue`返回开头的元素，即 header->next。

结构定义
```c
typedef int element_type;
typedef struct node
{
    element_type element;
    struct node *next;
} *queue;
```
全部代码实现：

```c
/**
 * 队列 ADT-链表实现
 */

#include <stdio.h>
#include <stdlib.h>

#define error(str) fatal_error(str)
#define fatal_error(str) fprintf(stderr, "%s\n", str),exit(1)

typedef int element_type;
typedef struct node *ptr_to_node;
typedef ptr_to_node queue;

struct node
{
    element_type element;
    ptr_to_node next;
};

queue create_queue();
void make_empty(queue q);
int is_empty(queue q);
void enqueue(element_type x, queue q);
void dequeue(queue q);
element_type front(queue q);
element_type front_and_dequeue(queue q);
void dispose_queue(queue *q);

void test();

int main(void)
{
    test();
    return 0;
}

queue create_queue()
{
    queue q;
    q = (queue)malloc(sizeof(struct node));
    if (NULL == q)
        fatal_error("Out of space");
    q->next = NULL;
    make_empty(q);
    return q;
}

void make_empty(queue q)
{
    if (NULL == q)
        error("must create queue");
    else
        while (!is_empty(q)) {
            dequeue(q);
        }
}

int is_empty(queue q)
{
    return q->next == NULL;
}

void enqueue(element_type x, queue q)
{
    if (NULL == q)
        fatal_error("must create a queue");
    queue temp_cell, p;
    temp_cell = (queue)malloc(sizeof(struct node));
    if (NULL == temp_cell)
        fatal_error("out of space");

    temp_cell->element = x;
    temp_cell->next = NULL;

    p = q;
    while (NULL != p->next)
        p = p->next;
    p->next = temp_cell;
}

void dequeue(queue q)
{
    if (NULL == q)
        fatal_error("must create a queue");
    queue temp;
    temp = q->next;
    q->next = temp->next;
    free(temp);
    temp = NULL;
}

element_type front(queue q)
{
    if ( !is_empty(q) )
        return q->next->element;
    error("empty queue");
    return 0;
}

element_type front_and_dequeue(queue q)
{
    if ( !is_empty(q) ){
        element_type ret = q->next->element;
        dequeue(q);
        return ret;
    }
    error("empty queue");
    return 0;
}

void dispose_queue(queue *q)
{
    make_empty((*q));
    free((*q));
    (*q) = NULL;
}

void print_queue(queue q)
{
    if (NULL == q)
        fatal_error("must be a queue");
    queue p;
    p = q->next;
    printf("队头\t");
    while(p) {
        printf("\t%d", p->element);
        p = p->next;
    }
    printf("\t队尾\n");
}

void test()
{
    queue q;
    int act;
    while (1) {
        printf("\n\tplease input a intger and choose a option.\n");
        printf("\t\t1.create a empty queue.\n");
        printf("\t\t2.enqueue a element to queue.\n");
        printf("\t\t3.dequeue a element from queue.\n");
        printf("\t\t4.get the front element of queue.\n");
        printf("\t\t5.print queue.\n");
        printf("\t\t6.dispose queue.\n");
        printf("\t\t0.exit\n");

        scanf("%d", &act);
        if (act == 0)
            break;
        switch(act) {
            case 1:
                q = create_queue();
                printf("create queue success\n");
                break;
            case 2: {
                int x;
                printf("please input a intger.\n");
                scanf("%d", &x);
                enqueue(x, q);
                printf("enqueue success\n");
                print_queue(q);
                break;
            }
            case 3:
                dequeue(q);
                printf("dequeue a element success\n");
                break;
            case 4:
                printf("queue front:%d\n", front(q));
                break;
            case 5:
                print_queue(q);
                break;
            case 6:
                dispose_queue(&q);
                printf("dispose queue success\n");
                break;
        }
    }
}
```