title:    Unique Paths III
date: 2020-4-13
tags: [Leetcode,回溯]
categories: algorithm

description: 不同路径 III
---

## Problem description

https://leetcode-cn.com/problems/unique-paths-iii/

## Solution

一道和迷宫的走法类似的题，当访问到 2时，怎么才能知道其他的方格已经访问完毕了。可以用一个 Int 值rest记录 剩余未访问的方格。每次访问一个 0 号方格后减一。回溯后再加一。差一点点，就是差很多啊。

## Code

```java
class Solution {
    var rows = 0
    var cols = 0
    val end = Array(2) { 0 }
    var result = 0
    var rest = 0
    fun uniquePathsIII(grid: Array<IntArray>): Int {
        if (grid.isEmpty()) return 0
        rows = grid.size
        cols = grid[0].size
        rest = rows * cols // 初始 rest 为 rows * cols
        var srcRow = 0
        var srcCol = 0
        for (i in grid.indices) // 遍历找到 rest,起始点和终止点
            for (j in grid[i].indices)
                when (grid[i][j]) {
                    -1 -> rest--
                    1 -> {
                        srcRow = i
                        srcCol = j
                    }
                    2 -> {
                        end[0] = i
                        end[1] = j
                    }
                }
        dfs(srcRow, srcCol, grid)
        return result
    }

    private fun dfs(row: Int, col: Int, grid: Array<IntArray>) {
        if (row == end[0] && col == end[1]) {
            if (rest == 1) result++
            return
        }
        rest--
        if (rest < 0) return
        grid[row][col] = -1
        if (row - 1 >= 0 && grid[row - 1][col] != -1) dfs(row - 1, col, grid)
        if (row + 1 < grid.size && grid[row + 1][col] != -1) dfs(row + 1, col, grid)
        if (col - 1 >= 0 && grid[row][col - 1] != -1) dfs(row, col - 1, grid)
        if (col + 1 < grid[0].size && grid[row][col + 1] != -1) dfs(row, col + 1, grid)
        grid[row][col] = 0
        rest++
    }

}
```

