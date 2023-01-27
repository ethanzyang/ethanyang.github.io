---
title: "树【数据结构与算法分析 c 语言描述】"
date: 2018-12-28 00:27:00
slug: "tree-[data-structure-and-algorithm-analysis-c-language]"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

## 1. 树的概念
树就是一种非线性的数据结构，其平均复杂度为O(logN)。
## 2. 实现思路
* 只需要一个指向第一个节点的指针跟右侧兄弟节点的指针
![file](https://cdn.learnku.com/uploads/images/201812/26/23174/g2YCeTu7zU.png!large)

## 3. 实现过程
定义树的结构体
```c
typedef int element_type;
typedef struct node
{
    element_type element;
    struct node *first_child;
    struct node *next_sibling;
} *tree;
```

要实现的方法
```c
tree create(element_type x);
void make_empty(tree t);
int is_empty(tree t);
void insert(element_type x, tree t, int is_child);
void print_tree(tree t);
void init(tree t); //初始化生成树手动输入

void rand_tree(tree t); //生成一个随机树
```
## 4. 全部代码
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define error(str) fatal_error(str)
#define fatal_error(str) fprintf(stderr, "%s\n", str),exit(1)

typedef int element_type;
typedef struct node
{
    element_type element;
    struct node *first_child;
    struct node *next_sibling;
} *tree;

tree create(element_type x);
void make_empty(tree t);
int is_empty(tree t);
void insert(element_type x, tree t, int is_child);
void print_tree(tree t);
void init(tree t); //初始化生成树手动输入

void rand_tree(tree t); //生成一个随机树
void test();

void init(tree t)
{
    tree temp_cell;
    element_type x;
    int has_child,  has_sibling, is_leaf;

    printf("has sibling node ?(1 or 0)\n");
    scanf("%d", &has_sibling);

    if (has_sibling == 1) {
        printf("please input a intger as a tree node\n");
        scanf("%d", &x);
        temp_cell = (tree)malloc(sizeof(struct node));
        temp_cell->element = x;
        temp_cell->first_child = NULL;
        temp_cell->next_sibling = NULL;

        t->next_sibling = temp_cell;

        init(temp_cell);
    }

    printf("has child node ?(1 or 0)\n");
    scanf("%d", &has_child);

    if (has_child == 1) {
        printf("please input a intger as a tree node\n");
        scanf("%d", &x);
        temp_cell = (tree)malloc(sizeof(struct node));
        temp_cell->element = x;
        temp_cell->first_child = NULL;
        temp_cell->next_sibling = NULL;

        t->first_child = temp_cell;

        init(temp_cell);
    }
}

tree create(element_type x)
{
    tree t;

    t = (tree)malloc(sizeof(struct node));
    if (NULL == t)
        fatal_error("out of space");

    t->first_child = NULL;
    t->next_sibling = NULL;
    t->element = x;

    // init(t);

    return t;
}

void make_empty(tree t)
{
    //置空一个 tree
    //递归调用每个节点销毁
    if (NULL != t) {
        make_empty(t->first_child);
        make_empty(t->next_sibling);
        free(t);
    }
}

int is_empty(tree t)
{
    return t == NULL;
}

// 插入做得比较简单
void insert(element_type x, tree t, int is_child)
{
    tree temp_cell;
    temp_cell = (tree)malloc(sizeof(struct node));
    temp_cell->first_child = NULL;
    temp_cell->next_sibling = NULL;
    temp_cell->element = x;
    //插那个节点，左插还是右插
    if (t->first_child == NULL)
        t->first_child = temp_cell;
    else
        t->next_sibling = temp_cell;
}

void rand_tree(tree t)
{
    tree temp_cell;
    static int hight = 0, width = 0, n = 1;

    if (n <= 10) {
        srand((unsigned)time(NULL));
        if ( (rand() % 2) == 1 && n != 1 ) {
            srand((unsigned)time(NULL) + hight + width);
            temp_cell = (tree)malloc(sizeof(struct node));
            temp_cell->first_child = NULL;
            temp_cell->next_sibling = NULL;
            temp_cell->element = rand() % 100 + 1;

            t->next_sibling = temp_cell;
            width++;
            n++;
            rand_tree(temp_cell);
        }

        if ( (rand() % 2) == 1 ) {
            srand((unsigned)time(NULL) + hight + width);
            temp_cell = (tree)malloc(sizeof(struct node));
            temp_cell->first_child = NULL;
            temp_cell->next_sibling = NULL;
            temp_cell->element = rand() % 100 + 1;

            t->first_child = temp_cell;
            hight++;
            width = 0;
            n++;
            rand_tree(temp_cell);
        }
    }

}

void print_tree(tree t)
{
    if (NULL != t) {
        printf("%d", t->element);   //先序遍历 先节点

        if (NULL != t->next_sibling)
            printf("\t");
        print_tree(t->next_sibling);
        printf("%d", t->element);   //中序遍历

        if (NULL != t->first_child)
            printf("\n");
        print_tree(t->first_child);
        printf("%d", t->element);   //后序遍历
    }
}

void test()
{
    tree t;
    t = create(1);
    rand_tree(t);
    print_tree(t);
}

int main(int argc, char const *argv[])
{
    test();
    return 0;
}
```