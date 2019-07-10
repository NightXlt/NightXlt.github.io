title: Symmetric Tree
date: 2019-2-14
tags: [Leetcode,腾讯]
categories: algorithm
description: 　　
---
## Problem description
  ### Given a binary tree, check whether it is a mirror of itself (ie, symmetric around its center).

For example, this binary tree [1,2,2,3,4,4,3] is symmetric:

　　1
   /　 　\
  2 　 　2
 /　\  　/ 　\
3　4 4　　3
But the following [1,2,2,null,3,null,3] is not:
　1
   /　\
  2　2
   \　　\
   3　　3
## Solution
　　对称条件是每个节点（左子树的右孩子）和（右子树的左孩子）以及（左子树的左孩子）和（右子树的右孩子）相等。但没有写出代码怎么递归表示。采用了复制一颗同样的树p。将p树交换左右子树。再判断root和p是否相等。一定要复制同样的树，否则root会跟着p同时变化。

## Code
```java
 /**recursively 
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
       public boolean isSymmetric(TreeNode root) {
        if (root == null) {
            return true;
        }
        return isSymmetrics(root.left, root.right);
    }

    private boolean isSymmetrics(TreeNode left, TreeNode right) {
        if (left == null && right == null) {
            return true;
        }
        if (left == null || right == null) {
            return false;
        }
        if (left.val == right.val) {
            return isSymmetrics(left.left, right.right) && isSymmetrics(left.right, right.left);
        }
        return false;
    }

}
```
```java
/**swap
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isSymmetric(TreeNode root) {
        TreeNode p = cloneTree(root);
        swapNode(p);
        return isSame(p, root);
    }
    public static TreeNode cloneTree(TreeNode root){
        TreeNode node=null;
        if(root==null) return null;
        node=new TreeNode(root.val);
        node.left=cloneTree(root.left);
        node.right=cloneTree(root.right);

        return node;
    }
    private boolean isSame(TreeNode s, TreeNode t) {
        if (s == null && t == null) {
            return true;
        }
        if (s == null || t == null) {
            return false;
        }
        if (s.val != t.val) {
            return false;
        }
        return isSame(s.left, t.left) && isSame(s.right, t.right);
    }
    private void swapNode(TreeNode p) {
        if (p == null || (p.left == null && p.right == null)) {
            return;
        }
        TreeNode t = p.left;
        p.left = p.right;
        p.right = t;
        if (p.left != null) {
            swapNode(p.left);
        }
        if (p.right != null) {
            swapNode(p.right);
        }
    }

}
```
