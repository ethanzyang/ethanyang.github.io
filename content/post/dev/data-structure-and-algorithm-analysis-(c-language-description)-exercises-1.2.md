---
title: "数据结构与算法分析（c 语言描述）习题 1.2"
date: 2018-12-14 22:04:00
slug: "data-structure-and-algorithm-analysis-(c-language-description)-exercises-1.2"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

```c
/**
 * 编写一个程序求解字谜游戏问题。
 *
 * 描述：输入是由一些字母和单词构成的二维数组，目标是找出字谜中的单词，这些单词可以是水平、垂直或沿对角线以任何方向放置。找出二维数组中所有的单词
 *
 */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 字典
char puzzle[4][4] = {
    {'t','h','i','s'},
    {'w','a','t','s'},
    {'o','a','h','g'},
    {'f','g','g','t'}
};

// 单词
char *dict[4] = {"this", "tow", "fat", "that"};

int word_exist(int x, int y, int direction, int strlen, char *word);

int main(void)
{
    char word[5];
    int x, y, direction, strlen;

    for (x = 0; x < 4; x++) {
        for (y = 0; y < 4; y++) {
            for (direction = 0; direction < 8; direction++) {
                for (strlen = 2; strlen <= 4; strlen++) {
                    if (word_exist(x, y, direction, strlen, word) == 1) {
                        printf("word:%s\n", word);
                        break;
                    }
                }
            }
        }
    }

    return 0;
}

// 查找单词
int word_exist(int x, int y, int direction, int strlen, char *word)
{
    // 单词长度从2开始分别向不同方向去找
    char sword[5];
    int i = 0, j;
    while(i < strlen) {
        sword[i] = puzzle[x][y];
        sword[i + 1] = '\0';
        i++;
        //遍历规则
        switch (direction){
            case 0:     //从左往右
                y++;
                break;
            case 1:     //从右往左
                y--;
                break;
            case 2:     //从上往下
                x++;
                break;
            case 3:     //从下往上
                x--;
                break;
            case 4:     //从左上往右下
                y++;
                x++;
                break;
            case 5:     //从右下往左上
                y--;
                x--;
                break;
            case 6:     //从右上往左下
                y--;
                x++;
                break;
            case 7:     //从左下往右上
                y++;
                x--;
                break;
            default:
                puts("Direction error");
                return 0;
        }
        if (x < 0 || y < 0) {
            for (j = 0; j < 4; j++) {
                if (strcmp(sword, dict[j]) == 0) {
                    strcpy(word, dict[j]);
                    return 1;
                }
            }
            return 0;
        }
    }
    // 与词典比较
    for (j = 0; j < 4; j++) {
        if (strcmp(sword, dict[j]) == 0) {
            strcpy(word, dict[j]);
            return 1;
        }
    }
    return 0;
}
```