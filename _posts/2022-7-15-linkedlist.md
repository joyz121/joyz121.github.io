---
layout: post
title: "数据结构：（一）链表"
subtitle: "链表C++实现"
author: "Joy"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.2
tags:
  - 数据结构
---

线性表树是由零或多个数据元素组成的有限序列。假设线性表的数据集合是{a1,a2,…,an}，除a1外，每个元素有且只有一个前驱；除an外，每个元素有且只有一个后继。

## 链表

链表中的元素由两部分组成：

- 存储数据——"数据域"
- 存储直接后续位置的指针——"指针域"

将这两部分组成的元素的存储结构称为“结点“

![node](/img/in-post/node.png)

使用结构体定义结点：

```c++
typedef struct ListNode
{
    double value;//数据域
    ListNode* next;//指针域
    ListNode(double value=0,ListNode* next=nullptr)
    {
        this->value=value;
        this->next=next;
    }
}ListNode;
```

## 单链表

​        链表由n个结点通过指针域相互链接构成。在链表的第一个结点之前会额外增设一个结点——头结点，头结点的数据域一般不存放数据；在链表的最后一个结点增设一个尾节点，尾结点的指针域存放nullptr。

![linked_list](/img/in-post/linked_list.png)

链表的实现方式如下：

```c++
typedef struct LinkedList
{
	ListNode* head,*tail;
	int len;
}LinkedList;
```

链表遍历：

```c++
void PrintList(LinkedList* list)
{
	ListNode* p=list->head->next;//当前指向的节点
	while(p!=nullptr)
	{
		cout<<p->value<<" ";
		p=p->next;
	}
}
```

按位置查找结点：

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

插入结点：先找到插入结点位置的前驱结点，然后令插入结点的 next 为前驱结点的 next；前驱结

点的 next 为插入结点。

![insert_node](/img/in-post/insert_node.png)

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

删除结点：

![delete_node](/img/in-post/delete_node.png)

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
