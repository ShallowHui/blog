---
title: 剑指Offer-day3：从尾到头打印链表
date: 2021-03-15 22:29:42
tags: 数据结构与算法
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/data.jpg
description: 这个系列的文章就用来记录我在Leetcode上刷的剑指Offer算法题目。
---
## 题目

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

示例：

有这样一个链表：（1，3，2），则head指向1，输入head，返回[2,3,1]。

## 解题

注意这是一个单向链表，不是双向，更不是循环链表哦~

```java
/*
	链表的节点类
*/
class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { 
        val = x; 
    }
}
```

这是让我们求现有链表的逆序嘛，可以想到用栈，从头节点开始将链表的每个节点放入栈，再依次取出栈中元素，由于栈先进后出的特性，出来的元素就是逆序的了。

用栈是以空间换时间的方法，时间和空间复杂度均为O(n)。

我们可不可以不用额外的空间呢？可以，反正都是要进行遍历链表的操作，那么干脆就在在遍历链表的时候，修改每个节点的next指针，让其指向前一个节点，这样就完成了链表的反转，再顺序输出这个链表就是原来链表的逆序了。

但反转链表需要的操作有点麻烦，而且一般来说也不建议修改输入的数据。我后来突然醒悟了，不必去反转链表，题目让我们返回的是一个跟链表顺序相反的数组，那么我们直接就可以顺序遍历链表，将节点的值倒序填入数组就行了......

原来这么简单......

代码实现如下：

```java
class Solution {
    public int[] reversePrint(ListNode head) {
        ListNode node = head;
        int count = 0;
	//要先获取链表长度
        while(node != null){
            count++;
            node = node.next;
        }
        int[] arr = new int[count];
        node = head;
        for(int i = count-1 ; i >= 0 ; i--){
            arr[i] = node.val;
            node = node.next;
        }
        return arr;
    }
}
```

这个方法遍历了两次链表，时间复杂度为O(n)，空间复杂度为O(1)。