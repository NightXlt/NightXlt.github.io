title: Sort a linked list
date: 2019-1-4
tags: [Leetcode,字节跳动]
categories: algorithm
description: Sort a linked list in O(nlogn) time using constant space complexity.
---
## Problem description
  ### Sort a linked list in O(n log n) time using constant space complexity.
 ## Examples
``` java
Example 1:
Input: 4->2->1->3
Output: 1->2->3->4
```
```java
Example 2:
Input: -1->5->3->4->0
Output: -1->0->3->4->5
```

## Solution
　　O(nlogn)用归并排序，快排一开始自己以为简单单链表是无法实现的,后来发现也是可以实现的。先说归并，先通过快慢指针找到中间节点，切记要将中间节点的next域设为null，再分别分治处理。值得一提的是归并的时间复杂度在链表的存储结构中降为了O(1),好神奇，有木有.快排则是采用了类似剑指Offer上的快排写法。这种写法避免了传统严奶奶版的双向移动，毕竟单链表往前动不了。

## Code

```java
 /**
 * Merge_Sort
 * 
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
       public ListNode sortList(ListNode head) {

        if (head == null || head.next == null) {
            return head;
        }

        ListNode fastNode = head, slowNode = head;

        while (fastNode.next != null && fastNode.next.next != null) {
		//cannot use while (fastNode != null && fastNode.next.!= null) {
		//This would be a stack overflow error when node number are even.
            fastNode = fastNode.next.next;
            slowNode = slowNode.next;
        }

        fastNode = slowNode;
        slowNode = slowNode.next;
        fastNode.next = null;
        head = sortList(head);
        slowNode = sortList(slowNode);
        return mergeTwoLists(head, slowNode);
    }
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

}
```

```c++
//  Quick_Sort
class Solution {
public:
    ListNode *quickSortList(ListNode *head) {
        // IMPORTANT: Please reset any member data you declared, as
        // the same Solution instance will be reused for each test case.
        //链表快速排序
        if(head == NULL || head->next == NULL)return head;
        qsortList(head, NULL);
        return head;
    }
    void qsortList(ListNode*head, ListNode*tail)
    {
        //链表范围是[low, high)
        if(head != tail && head->next != tail)
        {
            ListNode* mid = partitionList(head, tail);
            qsortList(head, mid);
            qsortList(mid->next, tail);
        }
    }
    ListNode* partitionList(ListNode*low, ListNode*high)
    {
        //链表范围是[low, high)
        int key = low->val;
        ListNode* loc = low;
        for(ListNode*i = low->next; i != high; i = i->next)
            if(i->val < key)
            {
                loc = loc->next;
                swap(i->val, loc->val);
            }
        swap(loc->val, low->val);
        return loc;
    }
};
```