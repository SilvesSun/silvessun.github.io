---
title: addTwoNumbers
date: 2018-03-01 16:58:40
tags:
- 链表
- LeetCode
categories:
- 算法
---

## 题目:
You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

> example
> Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.

## 分析
首先最容易想到的方法是直接获取到两个数, 相加的结果生成链表.
```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        # 首先取得两个数, 同时因为链表是倒序的, 即第一个节点就是链表的最低为, 所以初始base设置为1

        base = 1
        num1, num2 = 0, 0
        while  l1:
            # 取得当前节点的值
            num1 += l1.val*base

            # 然后指向下一节点, 同时将base*10
            l1 = l1.next
            base *= 10

        # num2同理
        base = 1
        while l2:
            num2 += l2.val*base
            l2 = l2.next
            base *= 10

        # 由于输出要求是反序的, 所以求得的结果也需要reverse

        r = map(int, str(num1 + num2))[::-1]

        # 返回值为表头, 并用一个临时变量来生成链表
        res = ListNode(r[0])

        tmp = res
        for i in r[1:]:
            next_node = ListNode(r[i])
            tmp.next = next_node
            tmp = next_node

        return res

```

上面这种解法直观, 但是耗时较长.
![解法1耗时](http://p3euxxfa8.bkt.clouddn.com/runtime1.png)

然后在LeetCode上Python的最优解法是, 整体思路是在遍历的过程中就将节点构建出来
```Python
class Solution(object):
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        lo = p = l1
        addi = 0
        while l1 and l2:
            p.val = l1.val + l2.val + addi
            l1 = l1.next
            l2 = l2.next
            addi = p.val / 10  
            p.val %= 10  
            q = p
            p = p.next

        if l2:
            p = l2
            q.next = p

        while addi > 0:
            if not p:
                p = ListNode(0)
                q.next = p   
            p.val = p.val + addi
            addi = p.val / 10
            p.val %= 10
            q = p
            p = p.next

        return lo

```

这个解法的耗时为99ms,
