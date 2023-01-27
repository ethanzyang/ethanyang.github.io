---
title: "栈的应用之平衡符号 【数据结构与算法分析 c 语言描述】"
date: 2018-12-26 18:38:00
slug: "The-balance-symbol-of-the-application-of-the stack-[data-structure-and-algorithm-analysis-c-language]"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

## 描述
![file](https://cdn.learnku.com/uploads/images/201812/26/23174/acWw6DNC7e.png!large)
## 实现思路
* 读入文件挨个字符遍历直到 `\0`
* 识别字符。
* 遇到 { [ ( 入栈。
* 遇到 } ] ) ，对比栈顶元素，是否成对，是就弹出，否就报错，栈顶为空也报错。
* 遍历结束，栈非空也报错。

如何识别字符
可以根据 ascii 编码，我这里自己做了两个数组分别存放符号的 “开” 跟 “闭”，用数组下标对应。
## 具体实现
栈相关代码这里不再给出，有兴趣的去看关于栈的那篇文章
[栈 ADT 【数据结构与算法分析 c 语言描述】
](https://learnku.com/articles/21534)
```c
/**
 * 栈应用-平衡符号
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define error(str) fatal_error(str)
#define fatal_error(str) fprintf(stderr, "%s\n", str),exit(1)

const char close_char[] = ")]}";
const char open_char[] = "([{";

char *file_get_content(char path[]);
int check_symbol(char *path);
int is_close_symbol(char ch);
int is_open_symbol(char ch);
int is_match(char chopen, char chclose);

int main(int argc, char const *argv[])
{
    char path[] = "./stack.c";
    check_symbol(path);
    return 0;
}

int check_symbol(char *path)
{
    FILE *fp = NULL;
    stack stack;
    char ch;
    stack = create_stack();
    fp = fopen(path, "r+");
    if (NULL == fp)
        fatal_error("not a corrent path");
    while ( (ch = fgetc(fp)) != EOF) {
        printf("%c", ch);
        if (is_open_symbol(ch) > -1) {
            push(ch, stack);
        } else if (is_close_symbol(ch) > -1) {
            if ( is_empty( stack ) )
                fatal_error("error1");

            if ( !is_match( top(stack), ch ) )
                fatal_error("error2");

            pop(stack);
        }
    }
    if (!is_empty(stack))
        fatal_error("error");

    fclose(fp);
    dispose_stack(stack);
}

int is_close_symbol(char ch)
{
    int i = 0;
    while (close_char[i] != '\0') {
        if (ch == close_char[i])
            return i;
        i++;
    }
    return -1;
}

int is_open_symbol(char ch)
{
    int i = 0;
    while (open_char[i] != '\0') {
        if (ch == open_char[i])
            return i;
        i++;
    }
    return -1;
}

int is_match(char chopen, char chclose)
{
    return is_open_symbol(chopen) == is_close_symbol(chclose);
}
```