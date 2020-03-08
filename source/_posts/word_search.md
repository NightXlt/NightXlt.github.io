title:    Word Search
date: 2020-3-7
tags: [Leetcode,回溯]
categories: algorithm

description: 单词搜索
---

## Problem description

https://leetcode-cn.com/problems/word-search/

## Solution

参考 ：https://leetcode-cn.com/problems/word-search/solution/de-acjing-li-zong-jie-yi-xia-zhe-ge-you-de-dfshui-/

这和一年前做过的最大岛屿数量那道题很类似，朝四个方向进行遍历。不过那道题更像深搜，这道题更像是回溯。两题的共同点是开始进行深搜前需要改变当前节点状态防止重复访问。

不同点这道题遍历完四个方向后需要恢复状态，避免其他节点访问到该节点时状态异常（被置为‘ ’）。恢复状态是回溯中独有的一个特征。`回溯需要保证访问节点前后状态一致。`牛匹的是题解中不使用多余空间进行存储访问过的字符串，仅需累计比较当前 word[cur]和 board[row] [column] 一致判断即可得到之前的节点也是一致的。此外我们访问节点的 cur 是从 0 until word.length，当心越界

这道题的剪枝

1.  || 短路或，当找到一个正确结果后，就不需要再进行深搜
2. 当前元素 word 中的元素和 board 的对应元素是否一致，如果不一致，及早结束后续无谓比较

时间复杂度：  $O(m \* n  \* 4 \* l)$ ,m*n 是 board 的大小， board 上每个点都会被访问到，4：每个顶点有四个邻居，l: word.length, 至多深搜至word.length 次。

空间复杂度：$O(1)$

## Code

```java
class Solution {
    fun exist(board: Array<CharArray>, word: String): Boolean {
        if (board.isEmpty()) {
            return false
        }
        fun dfs(row: Int, column: Int, cur: Int): Boolean {
            if (word[cur] != board[row][column]) {
                return false
            }
            if (cur == word.length - 1) {
                return true
            }
            val c = board[row][column]
            board[row][column] = ' '
            val res = (column < board[0].size - 1 && dfs(row, column + 1, cur + 1))
                    || (row < board.size - 1 && dfs(row + 1, column, cur + 1))
                    || (row > 0 && dfs(row - 1, column, cur + 1))
                    || (column > 0 && dfs(row, column - 1, cur + 1))
            board[row][column] = c
            return res
        }
        for (i in 0 until board.size) {
            for (j in 0 until board[0].size) {
                if (dfs(i, j, 0)) {
                    return true
                }
            }
        }
        return false
    }
}
```