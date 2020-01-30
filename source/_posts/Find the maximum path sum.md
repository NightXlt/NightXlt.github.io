title: Find the maximum path sum
date: 2019-1-24
tags: [Leetcode,腾讯]
categories: algorithm
description: 　　
---
## Problem description
  ### Given a non-empty binary tree, find the maximum path sum.For this problem, a path is defined as any sequence of nodes from some starting node to any node in the tree along the parent-child connections. The path must contain at least one node and does not need to go through the root.
 ## Examples
``` java
Example 1:
Input: [1,2,3]

       1
      / \
     2   3

Output: 6
```
```java
Example 2:
Input: [-10,9,20,null,null,15,7]

   -10
   / \
  9  20
    /  \
   15   7

Output: 42
```
## Solution
　　每个节点的左右子树的最大路径和为左右子树的最大路径和的最大值加上当前节点值。取其中最大值即为二叉树的路径和。
## Code
```java
 class Solution {
    int res = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        findMaxPath(root);
        return res;
    }

    public int findMaxPath(TreeNode node) {
        if (node == null) {
            return 0;
        }
        int left = findMaxPath(node.left);
        int right = findMaxPath(node.right);
        res = Math.max(res, Math.max(0, left) + Math.max(0, right) + node.val);
        return Math.max(0, Math.max(left, right)) + node.val;
    }

}
```