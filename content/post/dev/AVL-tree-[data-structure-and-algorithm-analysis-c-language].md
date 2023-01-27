---
title: "AVL 树【数据结构与算法分析 c 语言描述】"
date: 2019-01-13 03:11:00
slug: "AVL-tree-[data-structure-and-algorithm-analysis-c-language]"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

## 1. 概述
>主要知识点
* AVL 树定义
* 单旋转（左左单旋转、右右单旋转）
* 双旋转（左右双旋转、右左双旋转）

## 2. 什么是 AVL 树
>定义
带有平衡条件的二叉查找树。
平衡条件：其每个节点的左子树和右子树的高度最多相差1的二叉查找树。（空树的高度为 -1）。

* 所有的树操作都可以以时间 O(logN) 执行。
* 当插入时有可能会破坏平衡条件，我们通过**旋转（rotation）**来进行修正。

## 3. 如何解决不平衡情况
根据 AVL 树的特性可以知道：α 点的两颗子树的高度差2（α 是需要平衡的节点）。
所以这种不平衡在插入时可能出现的情况如下四种：
1.  α 的左儿子的左子树进行一次插入。（左左单旋转）
2. α 的左儿子的右子树进行一次插入。 （左右双旋转）
3. α 的右儿子的左子树进行一次插入。 （右左双旋转）
4. α 的右儿子的右子树进行一次插入。 （右右单旋转）

以上四种情况可以合成两种情况
* 1 和 4 是插入发生在外部（左左、右右）单旋转处理。
* 2 和 3 是插入发生的内部（左右、右左）双旋转处理。

## 4. 关于旋转
旋转是对树的常用操作，有两个最重要的属性：
* 旋转轴
* 旋转方向

### 4.1 旋转轴的确定
* 单旋转：α 的孩子节点。
* 双旋转：α 的孙子节点。

α 为需要平衡的节点即是不满足 AVL 特性的最小树的跟节点。

### 4.2 旋转方向
* 左左单旋转（顺时针）。
* 右右单旋转（逆时针）

### 4.3 单旋转还是双旋转
* 情况 1 和情况 4 用单旋转。
* 情况 2 和情况 3 需要用双旋转。

根据二叉搜索树的特性可以得到（设插入数为 x，旋转轴为 β）
* x 的大小阶于 α 的值与 β 的值之外（即是情况 1 和 4，外部插入）。
* x 的大小阶于 α 的值与 β 的值之间（即是情况 2 和 3，内部插入）。

## 5. 单旋转
单旋转：进行一次旋转。
1、4 的情况需要用到单旋转来修正。
左左单旋转（顺时针）
![file](https://cdn.learnku.com/uploads/images/201901/11/23174/YnRlrnLxCk.png!large)
右右单旋转（逆时针）
![file](https://cdn.learnku.com/uploads/images/201901/11/23174/Z7flhGJa9p.png!large)
## 6. 双旋转
进行两次单旋转。
2、3 的情况需要用双旋转来修正。
左右双旋转：先右右单旋转再左左单旋转
![file](https://cdn.learnku.com/uploads/images/201901/11/23174/oPmziHguR1.png!large)
右左双旋转：先左左单旋转再右右单旋转
![file](https://cdn.learnku.com/uploads/images/201901/11/23174/KwSbri4Dmg.png!large)
## 7. 代码实现
头文件，核心在 insert 跟旋转，其他实现跟二叉搜索树一样[二叉搜索树](https://learnku.com/articles/21753)
```c
typedef int element_type;

struct avl_node;
typedef struct avl_node *position;
typedef struct avl_node *avl_tree;

avl_tree make_empty(avl_tree t);
position find(element_type x, avl_tree t);
position find_min(avl_tree t);
position find_max(avl_tree t);
avl_tree insert(element_type x, avl_tree t);
avl_tree delete(element_type x, avl_tree t);
element_type retrieve(position p);

position single_rotate_with_left_left(position k2);
position single_rotate_with_right_right(position k2);
position double_rotate_with_left_right(position k3);
position double_rotate_with_right_left(position k3);

int height(position p);
int max(int a, int b);

void print_tree(avl_tree t);
void test();

struct avl_node
{
    element_type element;
    avl_tree left;
    avl_tree right;
    int height;
}
```

所有代码
```c
/**
 * AVL树
 */

#include <stdio.h>
#include <stdlib.h>
#include "avltree.h"

#define error(str) fatal_error(str)
#define fatal_error(str) fprintf(stderr, "%s\n", str),exit(1)

int height(position p)
{
    if (NULL == p)
        return -1;
    else
        return p->height;
}

position find(element_type x, avl_tree t)
{
    if (NULL == t)
        return NULL;
    if (x > t->element)
        return find(x, t->right);
    else if (x < t->element)
        return find(x, t->left);
    else
        return t;
}

position find_min(avl_tree t)
{
    if (NULL != t)
        while (NULL != t->left)
            t = t->left;
    return t;
}

position find_max(avl_tree t)
{
    if (NULL == t)
        return NULL;

    if (NULL == t->right)
        return t;
    else
        return find_max(t->right);
}

avl_tree delete( element_type x, avl_tree t)
{
    position temp;
    if (NULL == t)
        error("empty tree");

    if (x > t->element)
        t->right = delete(x, t->right);
    else if (x < t->element)
        t->left = delete(x, t->left);
    else if(t->left && t->right) {
        temp = find_min(t->right);
        t->element = temp->element;
        t->right = delete(t->element, t->right);
    } else {
        temp = t;
        if (NULL == t->left)
            t = t->right;
        else if (NULL == t->right)
            t = t->left;
        free(temp);
        temp = NULL;
    }
    return t;
}

avl_tree make_empty(avl_tree t)
{
    if (NULL != t) {
        make_empty(t->left);
        make_empty(t->right);
        free(t);
    }

    return NULL;
}

avl_tree insert(element_type x, avl_tree t)
{
    if (NULL == t) {
        t = (avl_tree)malloc(sizeof(struct avl_node));
        if (NULL == t)
            fatal_error("out of space");
        t->element = x;
        t->left = t->right = NULL;
        t->height = 0;
    } else if (x < t->element) {
        t->left = insert(x, t->left);
        if ( ( height(t->left) - height(t->right) ) == 2 ) {
            if (x < t->left->element)
                t = single_rotate_with_left_left(t);
            else
                t = double_rotate_with_left_right(t);
        }
    } else if (x > t->element) {
        t->right = insert(x, t->right);
        if ( ( height(t->right) - height(t->right) ) == 2 ) {
            if (x > t->right->element)
                t = single_rotate_with_right_right(t);
            else
                t = double_rotate_with_right_left(t);
        }
    }
    //节点高度为 左右节点高度最大值 + 1
    t->height = max(height(t->left), height(t->right) ) + 1;
    return t;
}

// 左左单旋转
position single_rotate_with_left_left(position k2)
{
    position k1;
    k1 = k2->left;
    k2->left = k1->right;
    k1->right = k2;
    k1->height = max( height(k1->left), height(k1->right) ) + 1;
    k2->height = max( height(k2->left), height(k2->right) ) + 1;
    return k1;
}
// 右右单旋转
position single_rotate_with_right_right(position k2)
{
    position k1;
    k1 = k2->right;
    k2->right = k1->left;
    k1->left = k2;
    k1->height = max( height(k1->left), height(k1->right) ) + 1;
    k2->height = max( height(k2->left), height(k2->right) ) + 1;
    return k1;
}
// 左右双旋转
position double_rotate_with_left_right(position k3)
{
    k3->left = single_rotate_with_right_right(k3->left);
    return single_rotate_with_left_left(k3);
}
// 右左双旋转
position double_rotate_with_right_left(position k3)
{
    k3->right = single_rotate_with_left_left(k3->right);
    return single_rotate_with_right_right(k3);
}
int max(int a, int b)
{
    return a > b ? a : b;
}

void print_tree(int depth, int left, avl_tree t)
{
    int i;
    if (t) {
        for(i = 0; i < left; i++)
            printf("    ");
        printf("%d\n", t->element);
        print_tree(depth + 1, left - 1, t->left);
        print_tree(depth + 1, left + 1, t->right);
    } 
}

void test()
{
    avl_tree t;
    printf("\n ====== test for building the AVLTree ====== \n");
    printf("\n [the left-left single rotate case] test with inserting 3, 2, 1 in trun \n");

    t = NULL;
    t = insert(3, t);   
    t = insert(2, t);   
    t = insert(1, t);   
    print_tree(1,5, t); 
}

int main(int argc, char const *argv[])
{
    test();
    return 0;
}
```
## 8. 测试截图
![file](https://cdn.learnku.com/uploads/images/201901/13/23174/43TVcwLqIT.png!large)
最后附上一个讲单双旋转讲得很好的文章
[单双旋转详解](https://blog.csdn.net/pacosonswjtu/article/details/50522677)
