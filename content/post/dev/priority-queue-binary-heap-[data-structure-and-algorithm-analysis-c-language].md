---
title: "优先队列 二叉堆【数据结构与算法分析 c 语言描述】"
date: 2019-02-21 00:35:00
slug: "priority-queue-binary-heap-[data-structure-and-algorithm-analysis-c-language]"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

# 前言
有段时间没看算法跟数据结构了，今天看了『优先队列』,并写成笔记。
# 概念
## 为什么需要优先队列?
>在一个队列中的作业，有的时候我们需要优先处理某些作业。
>或者说我们就是就是想要队列里面的作业有一个优先级的概念，可供自由的调整。

这就要优先队列来完成这些任务。
## 二叉堆
官方的定义：
>『二叉堆』是一颗被完全填满的二叉树，底层上的元素从左到右填入，这也叫『完全二叉树』

我的理解满足以下条件的二叉树
* 除叶子节点外所有节点都被填满
* 底层上的元素从左往右填「右边可能没有元素」

一个重要的发现是，我们可以用数组而不是指针来表示一个二叉堆的结构。
如果用数组的下标 key 来表示节点位置，1 为 root 节点，那么任意第 i 个节点，它的左儿子是 `2*i`，右儿子是 `2*i+1`
PS:这个动手画一下就理解了。
## 堆序
堆序就是始终要让最小或者最大的关键字在根节点上『最小堆』、『最大堆』。
也就是：任意节点一定大于它的后裔。
# 思路
结构体定义「数组实现」，关键的两个难点
* 插入元素，同时要维持堆序
* 弹出最大、最小值「root 节点」，找到最大、最小值到根节点

### 插入并维持堆序
定义 `size`为当前数组中元素个数「节点个数」。插入一个元素 X，首先 `size + 1`，我们计划用这个位置来放新插入的元素 X，因为 X 可能小可能大，所以我们要找到 X 在数组「二叉堆」中合适的位置以维持「堆序」。
我们假设 X 填入 `size + 1`的空位，从 `size + 1` 开始，比较 `(size + 1) / 2` 「父节点」与 X，X 小则 父子交换，一直重复直到 X 大于其中一个节点，或者 X 到 root 节点，这个过程叫『上溢』；此时完成插入。
### 删除
删除时候会让 root 节点为空，我们需要在数组中找出最小元素填入 root 节点。
root 节点被删除了，这时就有个空位。将这个空位跟它的子节点中小的节点交换『下溢』，直到空位到最后，将最后的元素填入空位即完成删除。
# 实现
头文件
```c
typedef int element_type;

struct heap_struct;

typedef struct heap_struct *priority_queue;

priority_queue initialize(int max_elements);
void destroy(priority_queue h);
void insert(element_type x, priority_queue h);
element_type delete_min(priority_queue h);
element_type find_min(priority_queue h);
int is_empty(priority_queue h);
int is_full(priority_queue h);

void random_heap(int size, priority_queue h);
void print_heap(priority_queue h);

void test();

struct heap_struct {
    int capacity;
    int size;
    element_type *elements;
};
```

heap.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <math.h>
#include "heap.h"

#define MIN_PQ_SIZE 5

#define error(str) fatal_error(str)
#define fatal_error(str) fprintf(stderr, "%s\n", str), exit(1)

priority_queue initialize(int max_elements)
{
    priority_queue h;
    
    if (max_elements < MIN_PQ_SIZE)
        fatal_error("Prioity queue size is too small");

    h = (priority_queue)malloc(sizeof(struct heap_struct));
    if (NULL == h)
        fatal_error("out of space");

    h->elements = malloc(sizeof(element_type) * (max_elements + 1) );
    if (NULL == h->elements)
        fatal_error("out of space");

    h->capacity = max_elements;
    h->size = 0;
    h->elements[1] = 0;

    return h;
}

void destroy(priority_queue h)
{
    if (NULL == h)
        fatal_error("empty priority queue");

    free(h->elements);
    h->elements = NULL;

    free(h);
    h = NULL;
}


void insert(element_type x, priority_queue h)
{
    int i; 

    if (is_full(h))
        fatal_error("priority queue is full");

    for (i = ++h->size; h->elements[ i / 2 ] > x; i /= 2)
        h->elements[ i ] = h->elements[ i / 2 ];

    h->elements[ i ] = x; 
}

element_type delete_min(priority_queue h)
{
    int i, child;
    element_type min_ele, last_ele;

    last_ele = h->elements[h->size--];
    min_ele = h->elements[1];

    for (i = 1; i * 2 <= h->size; i = child) {
        child = i * 2; // 左儿子

        if (child != h->size && h->elements[child+1] < h->elements[child]) // 左右儿子中取小
            child = child + 1;

        if (last_ele > h->elements[child]) // 为假的情况 child 为最后一个元素,否则违背堆序
            h->elements[i] = h->elements[child]; // 下溢一个
        else 
            break;
    }

    h->elements[i] = last_ele;

    return min_ele;
}

element_type find_min(priority_queue h)
{
    return h->elements[1];
}

int is_empty(priority_queue h)
{
    return h->size == 0;
}


int is_full(priority_queue h)
{
    return h->size == h->capacity;
}

void random_heap(int size, priority_queue h)
{
    int i;
    
    srand((unsigned)time(NULL));
    
    for (i = 0; i < size; i++)
        insert(rand() % 50, h);
}

void print_heap(priority_queue h)
{
    int i;

    printf("\t\t Array \n");

    for (i = 1; i < h->size; i++)
        printf("\t%d", h->elements[i]);

    printf("\n");

}

void test()
{
    priority_queue h;

    h = initialize(10);
    random_heap(8, h);
    print_heap(h);

    delete_min(h);

    print_heap(h);

    insert(3, h);

    print_heap(h);
}

int main(int argc, char const *argv[])
{
    test();
    return 0;
}
```
# 运行结果
![file](https://cdn.learnku.com/uploads/images/201902/20/23174/qxAEGXGL1V.png!large)
# 总结
* 优先队列的定义
* 二叉堆的定义跟性质
* 二叉堆的插入跟删除
* 最小堆的实现