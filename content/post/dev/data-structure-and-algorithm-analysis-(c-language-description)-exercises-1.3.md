---
title: "数据结构与算法分析（c 语言描述）习题 1.3"
date: 2018-12-14 22:08:00
slug: "data-structure-and-algorithm-analysis-(c-language-description)-exercises-1.3"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

```c
/**
 * 1.3
 * 只使用处理I/O的PrintDigit函数，编写一个过程以输出任意实数
 *
 */

#include <stdio.h>

#define PrintDigit( Ch ) ( putchar( ( Ch ) + '0' ) )

void print_int(int N);

void print_out(double N, int j);

int main(void)
{
    // double a, b, c;
    // a = 3.3339;
    // b = 1.1;
    // c = 9.1;
    print_out(-4473.154837, 3);
    putchar('\n');
    return 0;
}

void print_int(int N)
{
    if (N > 10) {
        print_int(N / 10);
    }
    PrintDigit(N % 10);
}

void print_out(double N, int j)
{
    if (N < 0) {
        putchar('-');
        N = -N;
    }
    int n = (int)N;
    print_int(n);
    putchar('.');
    //小数部分
    double decimal = N - n;
    while(j--) {
        decimal *= 10;
        print_int((int)decimal);
        decimal = decimal - (int)decimal;
    }
}
```