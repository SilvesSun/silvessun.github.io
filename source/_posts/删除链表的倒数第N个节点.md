---
title: 删除链表的倒数第N个节点
date: 2018-04-11 14:46:51
tags:
- 链表
- LeetCode
categories:
- 算法
---

## 题目
给定一个链表，删除链表的倒数第 n 个节点并返回头结点。

例如，
```
给定一个链表: 1->2->3->4->5, 并且 n = 2.

当删除了倒数第二个节点后链表变成了 1->2->3->5.
```

说明:

给的 n 始终是有效的。

尝试一次遍历实现。

## 解析
使用两个指针, 第一个指针比第二个指针先遍历n个节点, 这样当第一个指针到达尾部时, 第二个指针指向的位置正好是要删除的节点的上一位置

代码实现:
```py
class ListNode(object):
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution(object):
    def removeNthFromEnd(self, head, n):
        """
        :type head: ListNode
        :type n: int
        :rtype: ListNode
        """
        first_point = head
        second_point = head

        i = 0
        while i < n:
            first_point = first_point.next
            i += 1
        if not first_point:
            return head.next

        while first_point.next:
            first_point = first_point.next
            second_point = second_point.next

        second_point.next = second_point.next.next
        return head
```

结果:
![](http://p3euxxfa8.bkt.clouddn.com/2c70b1239c32dc81135421c980583b76.png)
