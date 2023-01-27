---
title: "数据结构与算法分析（c 语言描述）习题 1.1"
date: 2018-12-12 15:56:00
slug: "data-structure-and-algorithm-analysis-(c-language-description)-exercises-1.1"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

```c
/**
 * 问题描述：编写一个程序解决选择问题。令k = N / 2。画出表格显示你的程序对于N为不同值时的运行时间。
 *（设有一组 N 个数确定其中第 k 个最大者，称选择问题（selection problem））
 */

#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define N 100000

/*声明函数*/
int my_select(int arr[], int n, int k);

/*main 函数*/
int main(void)
{
    int * arr;
    int value;
    // int N;
    clock_t elapse;

    // scanf("%d", &N);

    srand((unsigned)time(NULL)); /*设置产生随机数的种子*/
    arr = (int *)malloc(sizeof(int) * N);
    for (int j = 0; j < N; j++) {
        arr[j] = rand() % 100000;
        printf("%d ", arr[j]);
    }
    printf("\n");
    // putchar("\n");

    elapse = clock();
    value = my_select(arr, N, N / 2);
    elapse = clock() - elapse;
    printf("Value: %d, elapsed: %.4lfs\n", value, (double)elapse / CLOCKS_PER_SEC);

    free(arr);
    // system("PAUSE");
    return 0;
}

/* 选择数组中第k个最大者 */
int my_select( int arr[], int n, int k)
{
    int * tmp;
    int i, j, ret;

    tmp = (int *)malloc(sizeof(int) * k); /*手动开辟内存空间*/
    tmp[0] = arr[0];
    /*读入k个元素并降序排序*/
    for (i = 1; i < k; i++) {
        tmp[i] = arr[i];
        for (j = i; j > 0; j--) { /* 冒泡排序 */
            if (arr[i] > tmp[j-1]) {
                tmp[j] = tmp[j-1];
                tmp[j-1] = arr[i];
            }
        }
    }
    /*读入arr[k,...]元素*/
    for (i = k; i < n; i++) {
        if (arr[i] > tmp[k-1]) {
            tmp[k-1] = arr[i];
            for (j = k-1; j > 0; j--) {
                if (arr[i] > tmp[j-1]) {
                    tmp[j] = tmp[j-1];
                    tmp[j-1] = arr[i];
                }
            }
        }
    }
    ret = tmp[k-1];
    free(tmp);
    return ret;
}
```