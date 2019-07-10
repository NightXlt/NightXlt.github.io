title: Rotate the list 
date: 2019-1-20
tags: [Leetcode,腾讯]
categories: algorithm
description: Rotate the list to the right by k places
---
## Problem description
  ### Given a linked list, rotate the list to the right by k places, where k is non-negative.
 ## Examples
``` java
Example 1:
Input: 1->2->3->4->5->NULL, k = 2
Output: 4->5->1->2->3->NULL
Explanation:
rotate 1 steps to the right: 5->1->2->3->4->NULL
rotate 2 steps to the right: 4->5->1->2->3->NULL
```
```java
Example 2:
Input: 0->1->2->NULL, k = 4
Output: 2->0->1->NULL
Explanation:
rotate 1 steps to the right: 2->0->1->NULL
rotate 2 steps to the right: 1->2->0->NULL
rotate 3 steps to the right: 0->1->2->NULL
rotate 4 steps to the right: 2->0->1->NULL
```
## Solution
　　陷入了惯性思维，以为不能修改链表。然后暴力解决。啊~差点超时。看了解析。我擦，原来可以操作链表![enter description here](/images/1547971023240.png)。
  
  将单链表变为单循环链表，同时记录链表元素个数num。只需要移动 num - k个元素即为所求。咦~是不是漏了什么？k > num？  不存在的。安排的妥妥的。  k = k % num.每逆序num个不是和没逆序一样吗？hhhhhhhhhhhhhh

## Code

```java
     public ListNode rotateRight(ListNode head, int k) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode p = head;
        int num = 1;
        while (p.next != null) {
            p = p.next;
            num++;
        }
        p.next = head;
        k %= num;
        for (int i = 0; i < num - k; i++) {
            p = p.next;
        }
        head = p.next;
        p.next = null;
        return head;
    }

```