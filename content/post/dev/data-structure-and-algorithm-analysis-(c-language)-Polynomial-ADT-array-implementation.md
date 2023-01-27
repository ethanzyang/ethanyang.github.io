---
title: "数据结构与算法分析（c 语言描述）多项式 ADT 数组实现"
date: 2018-12-19 21:51:00
slug: "data-structure-and-algorithm-analysis-(c-language)-Polynomial-ADT-array-implementation"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

```c
/**
 * 多项式 ADT 数组实现
 * 思路：以数组的 key 作为多项式的次数, value 作为多项式的系数。Hightpower 作为多项式的最高次数
 */

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <unistd.h>

#define MaxDegree 50

typedef struct Pol {
    int CoeffArray[MaxDegree + 1]; //次数=>系数
    int HightPower; //多项式的最高次数
} *Polynomial;

// 第二种声明
/*struct Pol {
    int CoeffArray[MaxDegree + 1]; //次数=>系数
    int HightPower; //多项式的最高次数
};
typedef struct Pol *Polynomial;*/

int Max(int a, int b);
void ZeroPolynomial(Polynomial Poly);
void AddPolynomial(const Polynomial Poly1, const Polynomial Poly2, Polynomial PolySum);
void MultPolynomial( const Polynomial Poly1, const Polynomial Poly2, Polynomial PolyMult);
void PrintPolynomial(Polynomial Poly);
void RandPolynomial(Polynomial Poly);

int main(void)
{
    Polynomial Poly1, Poly2, PolySum, PolyMult;
    Poly1 = malloc(sizeof(struct Pol));
    Poly2 = malloc(sizeof(struct Pol));
    PolySum = malloc(sizeof(struct Pol));
    PolyMult = malloc(sizeof(struct Pol));
    ZeroPolynomial(Poly1);
    ZeroPolynomial(Poly2);
    ZeroPolynomial(PolySum);
    ZeroPolynomial(PolyMult);

    int act;
    while(1) {
        printf("\n\t\tplease input a intger and choose a option.\n");
        printf("\t\t1.get tow random polynomial\n");
        printf("\t\t2.add tow polynomial\n");
        printf("\t\t3.multiplication tow polynomial\n");
        printf("\t\t4.print a polynomial\n");
        printf("\t\t0.exit\n");

        scanf("%d", &act);
        if (act == 0)
            break;
        switch (act) {
            case 1:
                printf("get tow random polynomial\n");
                RandPolynomial(Poly1);
                printf("\t\tthe fist polynomial.\n");
                PrintPolynomial(Poly1);
                sleep(1);
                RandPolynomial(Poly2);
                printf("\t\tthe second polynomial.\n");
                PrintPolynomial(Poly2);
                break;
            case 2:
                printf("add tow polynomial\n");
                AddPolynomial(Poly1, Poly2, PolySum);
                PrintPolynomial(Poly1);
                printf("\t\t+\n");
                PrintPolynomial(Poly2);
                printf("\t\t=\n");
                PrintPolynomial(PolySum);
                break;
            case 3:
                printf("multiplication tow polynomial\n");
                MultPolynomial(Poly1, Poly2, PolyMult);
                PrintPolynomial(Poly1);
                printf("\t\t\t*\n");
                PrintPolynomial(Poly2);
                printf("\t\t\t=\n");
                PrintPolynomial(PolyMult);
                break;
            case 4:
            {
                int temp; //c语言在 switch case 里面要定义变量必须要有{}
                printf("\t\t\tplease input a intger polynomial to print.\n");
                scanf("%d", &temp);
                switch (temp) {
                    case 1:
                        PrintPolynomial(Poly1);
                        break;
                    case 2:
                        PrintPolynomial(Poly2);
                        break;
                    case 3:
                        PrintPolynomial(PolySum);
                        break;
                    case 4:
                        PrintPolynomial(PolyMult);
                }
                break;
            }
        }
    }

    return 0;
}

int Max(int a, int b)
{
    return a > b ? a : b;
}

void ZeroPolynomial(Polynomial Poly)
{
    for (int i = 0; i <= MaxDegree; i++)
        Poly->CoeffArray[i] = 0;
    Poly->HightPower = 0;
}

void AddPolynomial(const Polynomial Poly1, const Polynomial Poly2, Polynomial PolySum)
{
    // ZeroPolynomial(PolySum);
    PolySum->HightPower = Max(Poly1->HightPower, Poly2->HightPower);

    for (int i = PolySum->HightPower; i >= 0; i--){
        PolySum->CoeffArray[i] = Poly1->CoeffArray[i] + Poly2->CoeffArray[i];
    }
}

void MultPolynomial(const Polynomial Poly1, const Polynomial Poly2, Polynomial PolyMult)
{
    // ZeroPolynomial(PolyMult);
    PolyMult->HightPower = Poly1->HightPower + Poly2->HightPower;
    for(int i = Poly1->HightPower; i >= 0; i--) {
        for(int j = Poly2->HightPower; j >= 0; j--) {
            PolyMult->CoeffArray[i + j] += Poly1->CoeffArray[i] * Poly2->CoeffArray[j];
        }
    }
}
/**
 * 格式化输出多项式
 */
void PrintPolynomial(Polynomial Poly)
{
    for (int i = 0; i <= Poly->HightPower; i++) {
        if(i != 0) {
            printf(" + ");
        }
        printf("%d*x^%d", Poly->CoeffArray[i], i);
    }
    printf("\n");
}
// 随机生成多项式
void RandPolynomial(Polynomial Poly)
{
    int i, HightPower;
    srand((unsigned)time(NULL));
    HightPower = rand() % 5 + 5; // 5-9
    Poly->HightPower = HightPower;
    for (i = 0; i <= HightPower; i++) {
        Poly->CoeffArray[i] = rand() % 100 + 1; //1-100;
    }
}
```