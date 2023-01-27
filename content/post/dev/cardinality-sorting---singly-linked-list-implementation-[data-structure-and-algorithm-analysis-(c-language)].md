---
title: "基数排序-单链表实现 [数据结构与算法分析（c 语言描述）]"
date: 2018-12-23 22:37:00
slug: "cardinality-sorting---singly-linked-list-implementation-[data-structure-and-algorithm-analysis-(c-language)]"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

## 实现思路
基数排序本身是一个：分配-收集 过程
1. 将一组需要排序的数，用链表存储 记为 L
2. 用一组链表的来存放分配的节点 N[0-9]
3. 遍历 L 每个节点分配到对应的 N[0-9]
4. 将 N[0-9] 重新收集到 L
5. 反复重复 3-4 直到最高位结束, 排序结束 ,
6. L 就变成了有序

## 排序过程
待排序数
64 8 216 512 27 729 0 1 343 125
第一次分配
N0    0
N1    1
N2    512
N3    343
N4    64
N5    125
N6    216
N7    27
N8    8
N9    729
收集
0->1->512->343->64->125->216->27->8->729
第二次分配
N0    0->1->8
N1    512->216
N2    125->27->729
N3  
N4    343
N5
N6    64
N7
N8
N9
收集
0->1->8->512->516->125->27->729->343->64
....
类推
## 代码实现
```c
/**
 * 基数排序-单链表实现
 */

#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define Error(Str) FatalError(Str)
#define FatalError(Str) fprintf(stderr,"%s\n",Str),exit(1)

#define N 10 //数个数
#define RADIX 10 //基数
#define POS_LEN 3 //位长

typedef struct Node
{
    int data;
    struct Node *next;
} *pNode;
typedef pNode LNode;

pNode create();
int get_num_pos(int num, int pos, int radix);
void radix_sort(pNode collect, int radix, int pos_len);
void append_node(pNode L, int num);
void insert(pNode P, int num);
void delete_list(pNode L);
void test();
void print(pNode L);

int main(void)
{
    test();
    return 0;
}

void radix_sort(pNode collect, int radix, int pos_len)
{
    // collect assign
    LNode assign[radix - 1], P, tmp, p;
    int i, num;
    for (i = 0; i < radix; i++) {
        assign[i] = create();
    }
    for (i = 1; i <= pos_len; i++) {
        P = collect;
        while (NULL != P->next) {
            p = P->next;
            P->next = p->next;
            p->next = NULL;
            int num = get_num_pos(p->data, i, radix);
            tmp = assign[num];
            while (NULL != tmp->next) {
                tmp = tmp->next;
            }
            tmp->next = p;
        }
        printf("第%d次分配\n", i);
        for (int j = 0; j < radix; j++) {
            printf("N-%d\t", j);
            print(assign[j]);
        }

        //assign
        P = collect;
        for (int j = 0; j < radix; j++) {
            LNode phead;
            phead = assign[j];
            while (NULL != phead->next) {
                p = phead->next;
                phead->next = p->next;
                p->next = NULL;
                P->next = p;
                P = P->next;
            }
        }
        printf("第%d次收集\n", i);
        print(collect);
        // 置空
        // for (int j = 0; j < radix; j++) {
        //  delete_list(assign[j]);
        // }
    }
    // free space
    for (i = 0; i < radix; i++) {
        delete_list(assign[i]);
    }
}

int get_num_pos(int num, int pos, int radix)
{
    int temp = 1, i;
    for (i = 0; i < pos - 1; i++)
        temp *= radix;
    return (num / temp) % radix;
}
void insert(pNode P, int num)
{
    pNode temp;
    temp = (pNode)malloc(sizeof(struct Node));
    if (NULL == temp)
        FatalError("out of space");
    temp->data = num;
    temp->next = NULL;
    P->next = temp;
}

void append_node(pNode L, int num)
{
    pNode temp, P;
    P = L;
    temp = (pNode)malloc(sizeof(struct Node));
    if (NULL == temp)
        FatalError("out of space");
    temp->next = NULL;

    while (NULL != P->next) {
        P = P->next;
    }
    P->next = temp;
}

pNode create()
{
    pNode L;
    L = (pNode)malloc(sizeof(struct Node));
    if (NULL == L)
        FatalError("out of space");
    L->next = NULL;
    return L;
}

void delete_list(pNode L)
{
    pNode P, temp;
    P = L->next;
    L->next = NULL;
    while (NULL != P) {
        temp = P->next;
        free(P);
        P = temp;
    }
}

void print(pNode L)
{
    pNode P;
    P = L->next;
    while (NULL != P)
    {
        printf("%d\t", P->data);
        P = P->next;
    }
    printf("\n");
}

void test()
{
    pNode source, tmp, P;
    int arr[N] = {64, 8, 216, 512, 27, 729, 0, 1, 343, 125};
    int i;
    int max = 1;
    for (i = 0; i <= POS_LEN - 1; i++)
        max *= RADIX;
    source = create();
    P = source;
    srand((unsigned)time(NULL));
    for (i = 0; i < N; i++) {
        insert(P, arr[i]);
        P = P->next;
    }
    print(source);
    radix_sort(source, RADIX, POS_LEN);
    printf("结果\n");
    print(source);
}
```
## 运行截图
![file](https://cdn.learnku.com/uploads/images/201812/23/23174/Cntx5yVlvJ.png!large)