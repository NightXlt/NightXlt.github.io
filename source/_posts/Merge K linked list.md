title: Merge K linked list
date: 2019-1-5
tags: [Leetcode,字节跳动]
categories: algorithm
description: Merge k sorted linked lists 
---
## Problem description
  ###  Merge k sorted linked lists and return it as one sorted list. Analyze and describe its complexity.
 ## Examples
``` java
Example 1:
Input:
[
  1->4->5,
  1->3->4,
  2->6
]
Output: 1->1->2->3->4->4->5->6
```

## Solution
　　暴力解决，先比较出两个，再比剩下的。虽然过了。但时间惨不忍睹。这样时间复杂度为O(n^2)
　　![merge_sort](/images/mergesort.png)
  　　这张图是不是很像哈希表的链地址解决冲突呀～，每个索引对应一个链表，果然画图还是解决算法最高效的方法。接下来一趟归并操作猛如虎，注意的是归并排序中的一个小细节，排序中只剩两个元素时，归并（这两个元素所指的链表）。一个元素时，直接返回。

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
    
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        } else if (l2 == null) {
            return l1;
        }
        ListNode mergeNode = null;
        if (l1.val <= l2.val) {
            mergeNode = l1;
            mergeNode.next = mergeTwoLists(l1.next, l2);
        } else {
            mergeNode = l2;
            mergeNode.next = mergeTwoLists(l1, l2.next);
        }
        return mergeNode;
    }

    public ListNode mergeSort(ListNode[] lists, int left, int right) {
        if (left < right) {
            int mid = (left + right) / 2;
            ListNode leftNode = mergeSort(lists, left, mid);
            ListNode rightNode = mergeSort(lists, mid + 1, right);
            return mergeTwoLists(leftNode, rightNode);
        }
        return lists[left];
    }

    public ListNode mergeKLists(ListNode[] lists) {
        if (lists.length == 0) {
            return null;
        }
        return mergeSort(lists, 0, lists.length - 1);
    }

}
```