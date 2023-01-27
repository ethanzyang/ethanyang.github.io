---
title: "数据结构与算法分析（c 语言描述）多项式 ADT 单链表实现"
date: 2018-12-19 22:06:00
slug: "data-structure-and-algorithm-analysis-(c-language)-implementation-of-polynomial-ADT-singly-linked-list"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---
```c
/**
 * 多项式-单链表实现
 * 思路：
 * 加法思路：P1(n~0) P2(i~0),假设n>i的(方便描述),P1(n)跟P2(n)比较关系,把大的或者相等的一方取出得到SumP(n~n-i)
 * 并且指针下移，最后加上 P2(n-i~0)。
 * 乘法思路一：P1(n~0)*P2(i~0) 然后合并同类项(排序) 复杂度为 O(N^4)
 * 乘法思路二：P1(n~0) P2(i~0), 首先P1(n)*P2(i~0)得到MultP(i~0),然后P1(n-1~0)*P2(i~0)得到
 * Tmp((n-1)*i~0)，把每次得到的记为Tmp(x),每次得到Tmp(x)就插入(合并)到MultP(i~0)中
 * 最后得到 MultP(k~0)，k为合并同类项后得到的项数,跟次数不一样;算法复杂度为O(N^3)
 */

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <unistd.h>

typedef struct Node *PtrToNode;
struct Node {
    int coefficient;
    int exponent;
    PtrToNode next;
};
typedef PtrToNode Polynomial;

/*typedef struct Node {
    int coefficient;
    int exponent;
    stuct Node *next;
} *Polynomial;*/

void print_poly(Polynomial poly);
Polynomial create_poly();
void rand_poly(Polynomial poly);
void add_poly(const Polynomial poly1, const Polynomial poly2, Polynomial poly_sum);
void mult_poly( Polynomial poly1,  Polynomial poly2, Polynomial poly_mult);
void attach(int coefficient, int exponent, Polynomial *pPoly);
void test();

int main(void)
{
    test();
    return 0;
}

Polynomial create_poly()
{
    Polynomial P;
    P = (Polynomial)malloc(sizeof(struct Node));
    P->next = NULL;
    return P;
}

void print_poly(Polynomial poly)
{
    Polynomial P;
    P = poly->next;
    while(P != NULL) {
        printf("%d*x^%d", P->coefficient, P->exponent);
        if (P->next != NULL)
            printf(" + ");
        P = P->next;
    }
    printf("\n");
}

void rand_poly(Polynomial poly)
{
    Polynomial tmpCell, P;
    int hight, i;
    P = poly;
    srand((unsigned)time(NULL));
    hight = rand() % 5 + 5; //5-9
    for (i = hight; i >= 0; i--) {
        tmpCell = (Polynomial)malloc(sizeof(struct Node));
        tmpCell->coefficient = rand() % 100 + 1; //1-100
        tmpCell->exponent = i;
        P->next = tmpCell;
        P = P->next;
    }
    P->next = NULL;
}

void attach(int coefficient, int exponent, Polynomial *pPoly)
{
    Polynomial tmp;
    tmp = (Polynomial)malloc(sizeof(struct Node));
    tmp->coefficient = coefficient;
    tmp->exponent =exponent;
    tmp->next = NULL;
    (*pPoly)->next = tmp; // 运算符的优先级,必须有()
    *pPoly = (*pPoly)->next;
}

void add_poly(const Polynomial poly1, const Polynomial poly2, Polynomial poly_sum)
{
    Polynomial p1, p2, P;
    P = poly_sum;
    p1 = poly1->next;
    p2 = poly2->next;
    while (p1 && p2) {
        if (p1->exponent > p2->exponent) {
            attach(p1->coefficient, p1->exponent, &P);
            p1 = p1->next;
        } else if(p1->exponent == p2->exponent) {
            attach(p1->coefficient + p2->coefficient, p2->exponent, &P);
            p1 = p1->next;
            p2 = p2->next;
        } else {
            attach(p2->coefficient, p2->exponent, &P);
            p2 = p2->next;
        }
    }
    // 加上没有参与计算的多项式(长的的一条)
    for (; p1; p1 = p1->next)
        attach(p1->coefficient, p1->exponent, &P);
    for (; p2; p2 = p2->next)
        attach(p2->coefficient, p2->exponent, &P);
    P->next = NULL;
}

void mult_poly( Polynomial poly1,  Polynomial poly2, Polynomial poly_mult)
{
    Polynomial p1, p2, P, tmp;
    int coefficient, exponent;
    P = poly_mult;
    p1 = poly1->next;
    p2 = poly2->next;

    while (p2) {
        attach(p1->coefficient * p2->coefficient, p1->exponent + p2->exponent, &P);
        p2 = p2->next;
    }
    while (p1) {
        p2 = poly2->next->next;
        while(p2) {
            coefficient = p1->coefficient * p2->coefficient;
            exponent = p1->exponent + p2->exponent;
            P = poly_mult->next;
            while (P) {
                if (P->exponent == exponent) {
                    P->coefficient += coefficient;
                    break;
                } else if(P->next && P->exponent > exponent && P->next->exponent < exponent) {
                    tmp = (Polynomial)malloc(sizeof(struct Node));
                    tmp->exponent = exponent;
                    tmp->coefficient = coefficient;
                    tmp->next = P->next;
                    P->next = tmp;
                    break;
                } else if(P->next == NULL) {
                    tmp = (Polynomial)malloc(sizeof(struct Node));
                    tmp->exponent = exponent;
                    tmp->coefficient = coefficient;
                    tmp->next = NULL;
                    P->next = tmp;
                    break;
                }
                P = P->next;
            }
            p2 = p2->next;
        }
        p1 = p1->next;
    }
}

void test()
{
    Polynomial poly1, poly2, poly_sum, poly_mult;
    poly1 = create_poly();
    poly2 = create_poly();
    poly_sum = create_poly();
    poly_mult = create_poly();

    int act;
    while(1) {
        printf("\n\t\tplease input a intger and choose a option.\n");
        printf("\t\t1.get tow random polynomial.\n");
        printf("\t\t2.add tow polynomial.\n");
        printf("\t\t3.multiplication tow polynomial.\n");
        printf("\t\t4.print a polynomial.\n");
        printf("\t\t0.exit.\n");

        scanf("%d", &act);
        if (act == 0)
            break;
        switch (act) {
            case 1:
                printf("get tow random polynomial.\n");
                rand_poly(poly1);
                printf("\t\tthe fist polynomial.\n");
                print_poly(poly1);
                sleep(1);
                rand_poly(poly2);
                printf("\t\tthe second polynomial.\n");
                print_poly(poly2);
                break;
            case 2:
                printf("add tow polynomial.\n");
                add_poly(poly1, poly2, poly_sum);
                print_poly(poly1);
                printf("\t\t+\n");
                print_poly(poly2);
                printf("\t\t=\n");
                print_poly(poly_sum);
                break;
            case 3:
                printf("multiplication tow polynomial.\n");
                mult_poly(poly1, poly2, poly_mult);
                print_poly(poly1);
                printf("\t\t\t*\n");
                print_poly(poly2);
                printf("\t\t\t=\n");
                print_poly(poly_mult);
                break;
            case 4:
            {
                int temp; //c语言在 switch case 里面要定义变量必须要有{}
                printf("\t\t\tplease input a intger polynomial to print.\n");
                scanf("%d", &temp);
                switch (temp) {
                    case 1:
                        print_poly(poly1);
                        break;
                    case 2:
                        print_poly(poly2);
                        break;
                    case 3:
                        print_poly(poly_sum);
                        break;
                    case 4:
                        print_poly(poly_mult);
                }
                break;
            }
        }
    }
}

```
运行结果截图
![file](https://cdn.learnku.com/uploads/images/201812/19/23174/FN4ALl4O1S.png!large)