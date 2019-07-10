title:  Add the two non-negative numbers
date: 2019-1-4
tags: [Leetcode,字节跳动]
categories: algorithm
description: Add the two  negative numbers and return it as a linked list.
---
## Problem description
  ### You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.You may assume the two numbers do not contain any leading zero, except the number 0 itself.
 ## Examples
``` java
Example 1:
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

## Solution
　　看到这道超级熟悉的题目，写两个点。1：看清题目，一开始以为是正序的单链表，就觉得直接算不好算必须放到数组中。后来再看了下题目是逆序，这就好处理啦。2：还是循环少套if的问题，降低时间复杂度。创立一个头结点，省去判断，返回时，返回其next.效率提升了一半。

## Code

```java
 /**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
        public static ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode result = new ListNode(0);
        ListNode p = result;
        int carry = 0;
        while (l1 != null || l2 != null || carry != 0) {
            int sum = ((l1 == null) ? 0 : l1.val) + ((l2 == null) ? 0 : l2.val) + carry;
            carry = sum / 10;
            sum = sum % 10;
            ListNode node = new ListNode(sum);
            node.next = null;
            p.next = node;
            p = node;
            l1 = (l1 == null) ? null : l1.next;
            l2 = (l2 == null) ? null : l2.next;
        }
        return result.next;
    }

}
```