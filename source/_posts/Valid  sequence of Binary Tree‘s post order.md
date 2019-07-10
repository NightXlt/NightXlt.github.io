title: Valid  sequence of Binary Tree‘s post order
date: 2019-2-21
tags: [Leetcode,剑指Offer]
categories: algorithm
description: 给定后序遍历的二叉序列是否合理
---
## Problem description
  ### 输入一个整形数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则返回true,否则返回false.假设输入的数组的数字都不相同
 ## Examples
``` java
Example 1:
Input:{5，7，6，9，11，10，8}
output:true
```
```java
Example 2:
Input:{4,8,1,7}
output: false
```
## Solution
在后序遍历得到的序列中，最后一个数字是树的根结点的值。数组中前面的数字可以分为两部分：第一部分是左子树结点的值，它们都比根结点的值小；第二部分是右子树结点的值，它们都比根结点的值大。 以数组{5，7，6，9，11，10，8）为例，后序遍历结果的最后一个数字8就是根结点的值。在这个数组中，前3个数字5、7和6都比8小，是值为8的结点的左子树结点；后3个数字9、11和10都比8大，是值为8的结点的右子树结点。 我们接下来用同样的方法确定与数组每一部分对应的子树的结构。这其实就是一个递归的过程。再来分析另一个整数数组{4，8，1，7）。后序遍历的最后一个数是根结点，因此根结点的值是7。故左子树为4，右子树的根节点为1.不大于7故返回false.

## Code

```java
     public boolean verifySequenceOfBST(int[] sequence, int low, int high) {

        if (sequence == null || low > high) {
            return false;
        }

        if (sequence.length == 0 || high == low) {
            return true;
        }

        int root = sequence[high - 1];
        int i, j;

        for (i = low; i < high - 1; i++) {//Remember to exclude the last one root node
            if (sequence[i] > root) {
                break;
            }
        }

        for (j = i; j < high - 1; j++) {//Remember to exclude the last one root node
            if (sequence[j] < root) {
                return false;
            }
        }
        boolean left = true;

        if (i > low) {//Remember to exclude the last one root node
            left = verifySequenceOfBST(sequence, low, i);
        }


        boolean right = true;

        if (i < high - 1) {//Remember to exclude the last one root node
            right = verifySequenceOfBST(sequence, i, high - 1);
        }

        return left && right;
    }
```