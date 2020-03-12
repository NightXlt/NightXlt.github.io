title:     N-Queens
date: 2020-3-10
tags: [Leetcode,回溯]
categories: algorithm

description: N皇后
---

## Problem description

https://leetcode-cn.com/problems/n-queens/

## Solution

参考：https://leetcode-cn.com/problems/n-queens/solution/nhuang-hou-by-leetcode/

经典的八皇后问题，在一个米字中不能有两个皇后。老办法进行深搜逐行遍历，找到 n - 1行表明找到一个可行解，否则如果放置了一个元素A导致下面一行无法进行放置，先恢复状态 移除该元素A,尝试将A 放置到该行的其他位置，继而继续遍历。整理为伪代码即 

```
fun backtrack(row) {
	for (col in 0 until n)
		if (isUnderAttack(row, col))//如果该位置不能放 Queue 直接 continue
				continue;
		addQueen(row, col)
		if (row == n)
			addResult()
		else 
			backtrack(row + 1)
		removeQueen(row, col)
}
```

其中关键的代码在于怎么判断 isUnderAttack。有两个相关的数学知识

1. 每条主对角线的 row - column 为常数，其 min 为 1 - n, max：n - 1
2. 每条副对角线的 row + column 为常数，其 min 为 0，max 为 2n - 2

我们需要标识两条对角线上的相应常数值是否有 queen 放置在上面了。我们还需要一个 rows数组 去进行标识某列是否有 queen，不需要去判断同一行是否有两个queen，因为在深搜过程中，i行 j 列添加了 queen 后，便会紧接着深搜下一行的 queen 位置进行放置。如果没找到放置位置，回溯到了 i 行 j 列，在访问 j + 1列前 `注意移除放置在 (i,j)的 queen`。  此外需要一个 queens 数组记录各行queen 的位置便于最后添加 result。

第二天自己回看了提交记录中最优的解决方案，用到了位运算进行判断isUnderAttack 判断。那怎么进行记录呢？使用https://nightxlt.github.io/2020/02/10/Max_Length_Concatenated_String_with_Unique_Characters/

这道题中的位压缩，用一个标志A记录状态，标志A中每一位表示该位置上是否有放置 queen。判断能否放置 queen也相当牛匹， 既然都是某一位是否为 1， 那就三个标志进行分辨右移对应位数（column, row - column + n - 1, row + column，这样一来首位就是标志位。将三个数的首位标志位或一下然后和 1 进行与操作。 结果为1 表明已经放置了 queen。  此外可以通过 xor B进行添加 queen，再次 xor B进行移除 queen。

## Code

```kotlin
Init:
class Solution {
    private lateinit var rows: BooleanArray
    private lateinit var mainDig: BooleanArray
    private lateinit var secDig: BooleanArray
    private lateinit var queens: IntArray
    private lateinit var res: MutableList<List<String>>
    private var n: Int = 0

    fun solveNQueens(n: Int): List<List<String>> {
        this.n = n
        res = mutableListOf()
        rows = BooleanArray(n)
        queens = IntArray(n)
        mainDig = BooleanArray(3 * n)//声明为3n是因为后面需要对 row - column 的结果进行放大为正数
        secDig = BooleanArray(2 * n - 1)
        backTrack(0)
        return res
    }

    private fun backTrack(row: Int) {
        for (col in 0 until n) {
            if (isUnderAttack(row, col)) {
                continue //切记是 continue，并非是 return。因为后续的列可能存在满足条件的元素
            }
            addQueen(row, col)
            if (row == n - 1) {
                addRes()
            } else {
                backTrack(row + 1)
            }
            removeQueen(row, col)
        }
    }

    private fun addRes() {
        val rowString = mutableListOf<String>()
        for (row in 0 until n) {
            val string = StringBuilder()
            for (col in 0 until queens[row]) string.append('.')
            string.append('Q')
            for (col in queens[row] + 1 until n) string.append('.')
            rowString.add(string.toString())
        }
        res.add(rowString)
    }

    private fun addQueen(row: Int, col: Int) {
        rows[col] = true
        mainDig[row - col + 2 * n] = true
        secDig[row + col] = true
        queens[row] = col
    }

    private fun removeQueen(row: Int, col: Int) {
        rows[col] = false
        mainDig[row - col + 2 * n] = false // + 2n放大为正数，那其最大值对应的就是 3n - 1,故空间需要为 3n
        secDig[row + col] = false
        queens[row] = 0
    }

    private fun isUnderAttack(row: Int, col: Int): Boolean { //判断同行，主对角线，副对角线是否放置了 queen
        return rows[col] || mainDig[row - col + 2 * n] || secDig[row + col]
    }
}
```



```kotlin
Advanced:
class Solution {

    private var rows = 0
    private var mainDig = 0
    private var secDig = 0
    private var n = 0
    private var count = 0

    private fun backTrack(row: Int) {
        for (col in 0 until n) {
            val mainDigConstants = row - col + n - 1
            val secDigConstants = row + col
            if (rows shr col or (mainDig shr mainDigConstants) or (secDig shr secDigConstants) and 1 != 0) {
                continue
            }
            if (row == n - 1) {
                count++
                return
            }
            rows = rows xor (1 shl col)
            mainDig = mainDig xor (1 shl mainDigConstants)
            secDig = secDig xor (1 shl secDigConstants)
            backTrack(row + 1)
            rows = rows xor (1 shl col)
            mainDig = mainDig xor (1 shl mainDigConstants)
            secDig = secDig xor (1 shl secDigConstants)
        }
    }

    fun totalNQueens(n: Int): Int {
        this.n = n
        backTrack(0)
        return count
    }
}
```

