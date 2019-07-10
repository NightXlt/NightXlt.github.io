title: Find  the cycle begins node
date: 2019-1-4
tags: [Leetcode,字节跳动]
categories: algorithm
description: Given a linked list, return the node where the cycle begins
---
## Problem description
  ### Given a linked list, return the node where the cycle begins. If there is no cycle, return null. To represent a cycle in the given linked list, we use an integer pos which represents the position (0-indexed) in the linked list where tail connects to. If pos is -1, then there is no cycle in the linked list.
  ### Note: Do not modify the linked list.Solve it without using extra space?
 ## Examples
``` java
Example 1:
Input: head = [3,2,0,-4], pos = 1
Output: tail connects to node index 1
Explanation: There is a cycle in the linked list, where tail connects to the second n
```
```java
Example 2:
Input: head = [1,2], pos = 0
Output: tail connects to node index 0
Explanation: There is a cycle in the linked list, where tail connects to the first node.
```
```java
Example 3:
Input: head = [1], pos = -1
Output: no cycle
Explanation: There is no cycle in the linked list.
```

## Solution
　　![cycle_list](/images/cycle_list.PNG)
  Y为环的起点，Z为slow走一步与fast走两步的第一次相遇之点。当第一次相遇时slow走过的距离：a+b，fast走过的距离：a+b+c+b。因为fast的速度是slow的两倍，所以fast走的距离是slow的两倍，有 2(a+b) = a+b+c+b，可以得到a=c。所以先找到slow与fast相遇的Z点。在令slow从X开始走，fast从Z点同步走相遇即为Y点
  一个小细节与上道题的差异：在第一次相遇之点的判断与上题不同。while (fastNode != null && fastNode.next != null) 与上题的while (fastNode.next != null && fastNode.next.next != null) 不同。而且都不能随意替换。这道题如果换成后者会结果出错。要具体问题具体分析条件，差异主要体现在链表结点数目为偶数时，前者会取到length / 2 + 1,后者会取到 length/2 - 1. 

## Code

```java
 /**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
      public ListNode detectCycle(ListNode head) {
        if (head == null) {
            return null;
        }
        ListNode fastNode = head, slowNode = head;
        while (fastNode != null && fastNode.next != null) {
            fastNode = fastNode.next.next;
            slowNode = slowNode.next;
            if (fastNode == slowNode) {
                break;
            }
        }
        if (fastNode == slowNode && fastNode.next != null) {
            slowNode = head;
            while (slowNode != fastNode) {
                slowNode = slowNode.next;
                fastNode = fastNode.next;
            }
            return fastNode;
        }
        return null;
    }
}
```