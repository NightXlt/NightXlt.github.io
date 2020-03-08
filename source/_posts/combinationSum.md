title:    Combination Sum
date: 2020-3-3
tags: [Leetcode,回溯]
categories: algorithm

description: 组合总和
---

## Problem description

https://leetcode-cn.com/problems/combination-sum/

## Solution

题目中关键点：

1. 每个数字可以被重复取
2. 组合的和为指定值

做回溯题，需要画出递归树进行分析，参考[该链接递归树](https://leetcode-cn.com/problems/combination-sum/solution/hui-su-suan-fa-jian-zhi-python-dai-ma-java-dai-m-2/).

剪枝的情况

1. 对数组进行排序，排序的目的是之后寻找时可以方便剪枝，当 sum+cur > target 时，后面比 cur 还大的数就不需要考虑了。
2. 当 sum > target 时 或者遍历完数组时，sum < target， 就直接返回

深搜有两种情况

1. 包含自己，毕竟自己也可以重复添加进组合中
2. 不包含自己，访问下一元素前需要将当前元素从 Deque 中去除。

使用 Dequeue 便于移除队尾元素。

## Code

```java
import java.util.*
class Solution {
    fun combinationSum(candidates: IntArray, target: Int): List<List<Int>> {
        val res = mutableListOf<List<Int>>()
        Arrays.sort(candidates)
        fun DFS(cur: Int, sum: Int, lists: Deque<Int>) {
            if (cur == candidates.size || sum > target) {
                return
            }
            if (sum == target) {
                val list = lists.toList()
                res.add(list)
                return
            }
            if (sum + candidates[cur] > target) {
                return
            }
            lists.add(candidates[cur])
            DFS(cur, sum + candidates[cur], lists) //以 cur 为起始，再次深搜 cur
            lists.removeLast()
            DFS(cur + 1, sum, lists) //不添加 cur，深搜下一节点
        }
        DFS(0, 0, ArrayDeque())
        return res
    }
}
```