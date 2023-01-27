---
title: "数据结构与算法分析（c 语言描述）多项式 ADT 单链表实现"
date: 2018-12-20 22:13:00
slug: "data-structure-and-algorithm-analysis-(c-language)-Radix-sorting-Array-implementation"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

```c
/**
 * 基数排序数组实现
 * 思路:
 * 基数排序是一个 分配-收集 的过程
 * N--需要排序的个数 radix-基数 pos_len 位数
 * 其中用二维数组来表示桶[j][N].j-桶位,B[j][0]:分配的元素个数,B[j][1~N]:分配的元素
 * 然后在收集到原数组中,再重复上面动作指导到达 pos_len 的最高位,排序完毕
 * 以空间换时间的思想
 */

#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define Error(Str) FatalError(Str)
#define FatalError(Str) fprintf(stderr,"%s\n",Str),exit(1)

// #define DEBUG 1
#define N 10 //要排序的数个数
#define RADIX 12 // 基数 进制
#define BUCKET_LEN 12 //桶的长度 桶长=基数
#define POS_LEN 4 //位数

void radix_sort(int *pArr, int n, int _radix, int bucket_len, int pos_len);
int get_num_in_pos(int num, int pos, int _radix);
void print_arr(int arr[], int n);
void print_bucket(int *arr, int n);
void test();

int main(void)
{
    test();
    return 0;
}

/**
 * 基数排序核心
 * @param pArr       需要排序数组的指针
 * @param n          数的个数
 * @param _radix     基数
 * @param bucket_len 桶长
 * @param pos_len    位长
 */
void radix_sort(int *pArr, int n, int _radix, int bucket_len, int pos_len)
{
    int pos;
    int *bucket[bucket_len];
    for (int i = 0; i < bucket_len; i++) {
        bucket[i] = (int*)malloc(sizeof(int) * n + 1); //可能该位上的所有数相同
        if (NULL == bucket[i])
            FatalError("out of space");
        bucket[i][0] = 0; //记录该桶上的数个数
    }

    for (pos = 1; pos <= pos_len; pos++) {
        //分配
        for (int i = 0; i < n; i++) {
            int num_in_pos = get_num_in_pos(pArr[i], pos, _radix);
            int index = ++bucket[num_in_pos][0];
            bucket[num_in_pos][index] = pArr[i];
        }
        // 格式化输出桶,过程更形象直观
#ifdef DEBUG
        int *arr1[n];
        for (int i = 0; i < n; i++) {
            arr1[i] = (int*)malloc(sizeof(int) * (bucket_len - 1) );
            if (NULL == bucket[i])
                FatalError("out of space");
        }
        for (int i = 0; i < bucket_len; i++) {
            for (int j = 1; j <= bucket[i][0]; j++) {
                arr1[j-1][i] = bucket[i][j];
            }
        }
        printf("第%d位\n", pos);
        for (int i = n - 1; i >= 0 ; i--) {
            int count = 0, t_count = 0;
            for (int j = 0; j < bucket_len; j++) {
                if (arr1[i][j]) {
                    count++;
                    if (t_count) {
                        for (int k = 0; k <= j - t_count; k++)
                            printf("\t");
                    } else {
                        for (int k = 0; k <= j; k++)
                            printf("\t");
                    }
                    printf("%d ", arr1[i][j]);
                    t_count = j + 1;
                }
            }
            if (count > 0)
                printf("\n");
        }
        for (int i = 0; i < bucket_len; i++) {
            printf("\t%d", i);
        }
        printf("\n");
#endif

        //收集
        int k = 0; //pArr索引
        for (int i = 0; i < bucket_len; i++) {
            for (int j = 1; j <= bucket[i][0]; j++)
                pArr[k++] = bucket[i][j];
            bucket[i][0] = 0; //重置次数
        }
#ifdef DEBUG
        printf("\n");
        print_arr(pArr, n);
#endif
    }
}

/**
 * 根据基数跟位数获取当前位上的数值
 * @param  num   数
 * @param  pos   第几位数
 * @param  _radix 基数
 * @return       pos位上的数
 */
int get_num_in_pos(int num, int pos, int _radix)
{
    int temp = 1;
    for (int i = 0; i < pos - 1; i++)
        temp *= _radix;
    return (num / temp) % _radix;
}

void test()
{
    int i, arr[N];
    srand((unsigned)time(NULL));
    int max = 1;
    for (i = 0; i <= POS_LEN - 1; i++)
        max *= RADIX;
    for (i = 0; i < N; i++) {
        arr[i] = rand() % max;
    }
    printf("get a random intger array\n");
    print_arr(arr, N);
    printf("\tafter sort\n");
    radix_sort(arr, N, RADIX, BUCKET_LEN, POS_LEN);
    print_arr(arr, N);

}

void print_arr(int arr[], int n)
{
    for (int i = 0; i < n; i++)
        printf("\t%d", arr[i]);
    printf("\n");
}
```
结果截图
![file](https://cdn.learnku.com/uploads/images/201812/20/23174/Pm4tzvCpOx.png!large)