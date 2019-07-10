title: The zigzag level order traversal
date: 2019-1-5
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### Given a binary tree, return the zigzag level order traversal of its nodes' values. (ie, from left to right, then right to left for the next level and alternate between).
 ## Examples
``` java
Example 1:
Input: [3,9,20,null,null,15,7]
Output:: [
  [3],
  [20,9],
  [15,7]
]
```


## Solution
　　层次遍历的封装吧，层次遍历细节处理注意，起初分为奇偶层处理，奇数层：先加入左子树，后加入右子树。偶数层：先加入右子树，后加入左子树。但没有意识到偶数层逆序加入后，它下面的奇数层无法再顺序访问。
 　　无需在加入队列中特殊化处理，在加入List时做顺序逆序处理。

## Code

```java
 /**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        List<List<Integer>> treeNodes = new ArrayList<>();
        if (root == null) {
            return treeNodes;
        }
        Queue<TreeNode> queue = new ArrayDeque<>();
        queue.add(root);
        TreeNode p;

        boolean isReverse = false;
        while (!queue.isEmpty()) {
            int size = queue.size();
            List<Integer> treeNode = new ArrayList<>();

            while (size-- != 0) {
                p = queue.poll();
                if (p.left != null) {
                    queue.add(p.left);
                }
                if (p.right != null) {
                    queue.add(p.right);
                }
                if (!isReverse) {
                    treeNode.add(p.val);
                } else {
                    treeNode.add(0, p.val);
                }
            }
            isReverse = !isReverse;
            treeNodes.add(treeNode);
        }
        return treeNodes;
    }


}
```