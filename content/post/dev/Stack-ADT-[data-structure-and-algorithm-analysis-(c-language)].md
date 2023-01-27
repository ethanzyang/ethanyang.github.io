---
title: "栈 ADT [数据结构与算法分析 c 语言描述]"
date: 2018-12-24 19:07:00
slug: "Stack-ADT-[data-structure-and-algorithm-analysis-(c-language)]"
tags: ["c", "数据结构与算法分析"]
categories: [Dev]
draft: false
---

## 栈模型
后进先出表
## 栈实现
需要实现
* 创建空栈
* 置空栈
* 入栈
* 出栈
* 获取栈顶
* 销毁栈
### 链表实现
结构体存放栈元素跟指向栈下一个结构体的指针
入栈 一个新的节点添加到 header->next, 这里是栈顶
出栈 弹出 header->next 即第一个节点（栈顶

```c
/**
 * 栈-链表实现
 */

#include <stdio.h>
#include <stdlib.h>

#define Error(Str) FatalError(Str)
#define FatalError(Str) fprintf(stderr,"%s\n",Str),exit(1)
// #define element_type int;
typedef int element_type;
typedef struct node
{
    element_type element;
    struct node *next;
} * ptr_to_node;
typedef ptr_to_node stack;

int is_empty(stack s);
stack create_stack();
void dispose_stack( stack s);
void make_empty(stack s);
void push(element_type x, stack s);
element_type top(stack s);
void pop(stack s);
void test();

int main(int argc, char const *argv[])
{
    test();
    return 0;
}

int is_empty(stack s)
{
    return s->next == NULL;
}

stack create_stack()
{
    stack s;
    s = (stack)malloc(sizeof(struct node));
    if (NULL == s)
        FatalError("out of space");
    s->next = NULL;
    make_empty(s);
    return s;
}

void dispose_stack(stack s)
{
    make_empty(s);
    free(s);
}

void make_empty(stack s)
{
    if (NULL == s)
        Error("must crated a stack");
    else
        while (!is_empty(s)) {
            pop(s);
        }
}

void push( element_type x, stack s)
{
    stack temp_cell;
    temp_cell = (stack)malloc(sizeof(struct node));
    if (NULL == temp_cell)
        FatalError("out of space");
    temp_cell->element = x;
    temp_cell->next = s->next;
    s->next = temp_cell;
}

element_type top(stack s)
{
    if (!is_empty(s))
        return s->next->element;
    Error("empty stack");
    return 0;
}

void pop(stack s)
{
    stack temp;
    temp = s->next;
    s->next = temp->next;
    free(temp);
}

void print_stack(stack s)
{
    stack p;
    p = s->next;
    while (NULL != p) {
        printf("\t%d\n", p->element);
        p = p->next;
    }
}

void test()
{
    stack s;
    int act;
    while (1) {
        printf("\n\tplease input a intger and choose a option.\n");
        printf("\t\t1.create a empty stack.\n");
        printf("\t\t2.push a element to stack.\n");
        printf("\t\t3.pop a element from stack.\n");
        printf("\t\t4.get the top element of stack.\n");
        printf("\t\t5.print stack.\n");
        printf("\t\t6.dispose stack.\n");
        printf("\t\t0.exit\n");

        scanf("%d", &act);
        if (act == 0)
            break;
        switch(act) {
            case 1:
                s = create_stack();
                printf("create stack success\n");
                break;
            case 2: {
                int x;
                printf("please input a intger.\n");
                scanf("%d", &x);
                push(x, s);
                printf("after push\n");
                print_stack(s);
                break;
            }
            case 3:
                pop(s);
                printf("pop a element success\n");
                break;
            case 4:
                printf("stack top:%d\n", top(s));
                break;
            case 5:
                print_stack(s);
                break;
            case 6:
                dispose_stack(s);
                printf("dispose stack success\n");
                break;
        }
    }
}
```

### 栈数组实现
用一个包含栈顶在数组下标（top）跟数组长度以及数组指针组成的结构体来存放栈
入栈：将元素存入数组的 top + 1位置，栈顶 + 1,即是：`arr[  ++top ]`
出栈：同理 弹出栈顶元素并 top--，即是：`arr[ top-- ]`

```c
/**
 * 栈-数组实现
 */

#include <stdio.h>
#include <stdlib.h>

#define error(str) fatal_error(str)
#define fatal_error(str) fprintf(stderr, "%s\n", str),exit(1)

#define MINI_STACK_SIZE ( 5 )
#define EMPTY_TOS ( -1 )

typedef int element_type;
struct stack_record;
typedef struct stack_record *stack;

struct stack_record
{
    int capacity;
    int top;
    element_type *arr;
};
/*typedef struct stack_record
{
    int capacity;
    int top;
    element_type *arr;
} *stack;*/
int is_empty(stack s);
int is_full(stack s);
stack create_stack(int max_len);
void dispose_stack(stack *s);
void make_empty(stack s);
void push(element_type x, stack s);
void pop(stack s);
element_type top(stack s);
element_type top_and_pop(stack s);

void print_stack(stack s);
void test();

int main(int argc, char const *argv[])
{
    test();
    return 0;
}

stack create_stack(int max_len)
{
    stack s;
    if (max_len < MINI_STACK_SIZE)
        error("max_len is too small.");
    s = (stack)malloc(sizeof(struct stack_record));
    if (NULL == s)
        fatal_error("out of space");

    s->capacity = max_len;
    s->arr = (element_type*)malloc(sizeof(element_type) * max_len);
    if (NULL == s->arr)
        fatal_error("out of space");
    make_empty(s);
    return s;
}

void dispose_stack(stack *s)
{
    if ( NULL != (*s) ) {
        free((*s)->arr);
        free((*s)); //free 后必须指向 NULL
        (*s)->arr = NULL;
        (*s) = NULL;
    }
}

int is_empty(stack s)
{
    return s->top == EMPTY_TOS;
}

int is_full(stack s)
{
    return s->top == s->capacity - 1;
}

void make_empty(stack s)
{
    s->top = EMPTY_TOS;
}

void push(element_type x, stack s)
{
    if (is_full(s))
        error("Full stack");
    else
        s->arr[ ++s->top ] = x;
}

void pop(stack s)
{
    if (is_empty(s))
        error("Empty stack");
    s->top--;
}

element_type top(stack s)
{
    if (!is_empty(s))
        return s->arr[ s->top];
    error("Empty stack");
    return 0;
}

element_type top_and_pop(stack s)
{
    if (!is_empty(s))
        return s->arr[ s->top-- ];
    error("Empty stack");
    return 0;
}

void print_stack(stack s)
{
    if (NULL == s)
        return;
    int i;
    for (i = s->top; i > EMPTY_TOS; i--) {
        printf("\t%d\n", s->arr[i]);
    }
}

void test()
{
    stack s;
    int act;
    while (1) {
        printf("\n\tplease input a intger and choose a option.\n");
        printf("\t\t1.create a empty stack.\n");
        printf("\t\t2.push a element to stack.\n");
        printf("\t\t3.pop a element from stack.\n");
        printf("\t\t4.get the top element of stack.\n");
        printf("\t\t5.print stack.\n");
        printf("\t\t6.dispose stack.\n");
        printf("\t\t0.exit\n");

        scanf("%d", &act);
        if (act == 0)
            break;
        switch(act) {
            case 1: {
                int max_len;
                printf("please input max_len of stack.\n");
                scanf("%d", &max_len);
                s = create_stack(max_len);
                printf("create stack success\n");
                break;
            }
            case 2: {
                int x;
                printf("please input a intger.\n");
                scanf("%d", &x);
                push(x, s);
                printf("after push\n");
                print_stack(s);
                break;
            }
            case 3:
                pop(s);
                printf("pop a element success\n");
                break;
            case 4:
                printf("stack top:%d\n", top(s));
                break;
            case 5:
                print_stack(s);
                break;
            case 6:
                dispose_stack(&s);
                printf("dispose stack success\n");
                break;
        }
    }
}
```