---
layout: post
title: "LinkedList"
subtitle: "线性表的链式实现"
author: "Joy"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.2
tags:
  - 数据结构
---

# 线性表的链式实现

链表中的元素由两部分组成：

- 存储数据——"数据域"
- 存储直接后续位置的指针——"指针域"

将这两部分组成的元素的存储结构称为“结点“，我们使用结构体定义结点。

```c++
typedef struct ListNode
{
    double value;
    ListNode* next;
    ListNode(double value=0,ListNode* next=nullptr)
    {
        this->value=value;
        this->next=next;
    }
}ListNode;
```

链表由n个结点通过指针域相互链接构成。在链表的第一个结点之前会额外增设一个结点——头结点，头结点的数据域一般不存放数据；在链表的最后一个结点增设一个尾节点，尾结点的指针域存放nullptr。

链表的实现方式如下：

```c++
typedef struct LinkedList
{
    ListNode* head,*tail;
    int len;
}LinkedList;
```

## 链表的基本操作

- **打印链表**

打印链表中各结点数据（除头结点和尾结点外）

```c++
void PrintList(LinkedList* list)
{
    ListNode* p=list->head->next;
    while(p!=nullptr)
    {
        cout<<p->value<<" ";
        p=p->next;
    }
}
```

- **按位置查找结点**

```c++
ListNode* FromLocGetNode(LinkedList* list,int loc)
{
    ListNode* p=list->head;
    int i;
    //从头遍历，直到loc或p=nullptr
    for(i=0;(p&&i<loc);i++)
    {
        p=p->next;
    }
    if(!p||i>loc)//p=nullptr 或者loc输入的位置小于0
    {
        cout<<"第"<<loc<<"个元素不存在"<<endl;
        return nullptr;
    }
    else
    {
        cout<<"第"<<loc<<"个元素为："<<p->value<<endl;
        return p;//返回节点指针
    }
}
```

- **向链表中插入结点**

```c++
void InsertNode(LinkedList* list,int loc,double value)//loc->节点插入的位置
{
    ListNode* temp=new ListNode(value);
    ListNode* p=FromLocGetNode(list,loc-1);//寻找前一个节点
    if(p)//p!=nullptr
    {
        temp->value=value;
        temp->next=p->next;
        p->next=temp;
        list->len++;
    }
}
```

- **删除结点**

```c++
void DeleteNode(LinkedList* list,int loc)
{
    ListNode* p=FromLocGetNode(list,loc-1);//寻找删除点前一个节点
    ListNode* temp;
    if(p)
    {
        temp=p->next;
        p->next=temp->next;
        free(temp);//此时被删除数据已经不在链表中，要有个中间变量存储其地址
        list->len--;
    }
}
```

## 测试代码

```c++
int main()
{
    LinkedList* L=new LinkedList();//创建链表
    L->head=(ListNode*)malloc(sizeof(ListNode));//创建表头,直接new ListNode(),表头将会存入数据
    L->tail=(ListNode*)malloc(sizeof(ListNode));
    L->head->next=L->tail;
    ListNode* temp= L->head;
    double num;
    while(cin>>num)
    {
        ListNode *q=new ListNode(num);
        temp->next=q;
        temp=temp->next;
        L->len++;
    }
    InsertNode(L,2,32);
    PrintList(L);
    DeleteNode(L,6);
    PrintList(L);
    return 0;
}
```

