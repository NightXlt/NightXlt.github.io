title: Find the lowest common ancestor (LCA) 
date: 2019-1-5
tags: [Leetcode,字节跳动]
categories: algorithm
description: Find the lowest common ancestor (LCA) of two given nodes in the tree.
---
## Problem description
  ### Given a binary tree, find the lowest common ancestor (LCA) of two given nodes in the tree.According to the definition of LCA on Wikipedia: “The lowest common ancestor is defined between two nodes p and q as the lowest node in T that has both p and q as descendants (where we allow a node to be a descendant of itself).” All of the nodes' values will be unique.p and q are different and both values will exist in the binary tree.
 ## Examples
``` java
Example 1:
Input: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
Output: 3
Explanation: The LCA of nodes 5 and 1 is 3.
```
```java
Example 2:
Input: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
Output: 5
Explanation: The LCA of nodes 5 and 4 is 5, since a node can be a descendant of itself according to the LCA definition.
```

## Solution
　　思路是先找到根节点到两个节点的路径，转换为求两个链表的公共节点。代码量不少啊...
  　　看了解析，有一个时间复杂度和该方法一致，但更为简介的算法。自底向上遍历结点，一旦遇到结点等于p或者q，则将其向上传递给它的父结点。父结点会判断它的左右子树是否都包含其中一个结点，如果是，则父结点一定是这两个节点p和q的LCA（前提是p,q唯一），传递父结点到root。如果不是，我们向上传递其中的包含结点p或者q的子结点，或者NULL(如果子结点不包含任何一个)。该方法时间复杂度为O(N)。

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
    boolean getPath(TreeNode root, TreeNode target, List<TreeNode> path) {
        path.add(root);
        if (root.val == target.val) {
            return true;
        }
        boolean found = false;
        if (root.left != null)
            found = getPath(root.left, target, path);
        if (!found && root.right != null)
            found = getPath(root.right, target, path);
        if (!found)
            path.remove(root);
        return found;
    }

    public TreeNode getLastAncestor(List<TreeNode> listP, List<TreeNode> listQ) {
        int pLength = listP.size();
        int qLength = listQ.size();
        TreeNode pNode = null;
        TreeNode qNode = null;
        for (int i = Math.min(pLength,qLength) - 1; i >= 0; i--) {
            pNode = listP.get(i);
            qNode = listQ.get(i);
            if (pNode == qNode) {
                return pNode;
            }
        }
        return qNode;
    }

    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        List<TreeNode> listP = new ArrayList<>();
        List<TreeNode> listQ = new ArrayList<>();
        getPath(root, p, listP);
        getPath(root, q, listQ);
        return getLastAncestor(listP, listQ);
    }
}
```
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
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root == null || root == p || root == q) return root;
        
        TreeNode l = lowestCommonAncestor(root.left, p, q);
        TreeNode r = lowestCommonAncestor(root.right, p, q);
        
        if(l == null && r == null) return null;
        if(l != null && r != null) return root;
        else return l = (l != null)? l:r; 
    }
}
```