title:    Combination Sum II
date: 2020-3-4
tags: [Leetcode,回溯]
categories: algorithm

description: 组合总和II
---

## Problem description

## https://leetcode-cn.com/problems/combination-sum-ii/submissions/

## Solution

参考：https://leetcode-cn.com/problems/combination-sum-ii/solution/hui-su-suan-fa-jian-zhi-python-dai-ma-java-dai-m-3/

和上一道题类似，自己一开始在上一步删除了第二次的 DFS，将第一次的 DFS 操作改为DFS(cur + 1, sum + candidates[cur], lists)，以为这样就可以 O了，candidates = [10,1,2,7,6,1,5], target = 8,。但是这个结果却出现了重复的元素，debug 时发现原来是1 重复出现了。咦， 怎样规避这个重复出现的 1 呢？我想想哈。数组是排过序的，那么重复的元素必然是聚集在一块的，加个 if (cur > 0 && candidates[cur] == candidates[cur - 1]) return；但这么一加竟然 结果集都为空了。想想也是哈，毕竟我们是进行深搜，如果中间有重复的后面的元素就会得不到遍历了。那还有没其他办法可以解决重复时跳过但不影响遍历后续元素呢。用循环的 `continue`可以做到。

在for 循环体内进行剪枝，判断 sum + 当前值是否大于target，大于就进行 return；  如果 (i > start && candidates[i] == candidates[i - 1]) 是的话我们进行 continue; 对于这个判断也可以从递归树去进行理解。将当前DFS 方法体内的循环体重从 i..length 的节点看为兄弟节点，如果在同一层上出现了重复节点必然会出现重复路径。想想哈，直观感受就是

```java
[1,2,2,5]
      1
    /   \
   2    2
 /     /
5    5  
```

此外需要注意的就是，for 循环外的两个 if 判断不能颠倒，颠倒了会出现漏解的情况，本题与上题中的if 判断是相反的，上一道题访问第 n.length - 1个元素时，先对自己再进行一次深搜，毕竟可以重复添加嘛，因为`下标没有加一`也保证了数组访问安全，而且不会出现漏解的情况；本题是访问第 n - 1个元素后，会将 n - 1下标的值加到 sum 里，`将下标加一后`再深搜一次，优先判断sum 值是否等于，等于就可以添加进res，不满足也会因为其下标达到 n 而直接返回不会出现越界情况。可以简单理解为之所以调整顺序是因为本次深搜时下标加一了。

这道题 是基本回溯法的模板，通过 for 循环控制将所有元素选中，未选中的情况都包含其中，可以说不失为暴力法，但是我们通过剪枝减少了无谓访问，比如通过排序 + 当前 sum > target时，就没必要再往后面走下去了；同为兄弟节点时，出现重复节点时，可跳过这次遍历等剪枝办法。

## Code

```java
import java.util.*

class Solution {
    fun combinationSum2(candidates: IntArray, target: Int): List<List<Int>> {
        val res = mutableListOf<List<Int>>()
        Arrays.sort(candidates)
        fun DFS(start: Int, sum: Int, lists: Deque<Int>) {
            if (sum == target) {
                val list = lists.toList()
                res.add(list)
                return
            }
            if (start == candidates.size || sum > target) {
                return
            }
            for (i in start until candidates.size) {
                if (sum + candidates[start] > target) {
                    break
                }
                if (i > start && candidates[i] == candidates[i - 1]) {
                    continue
                }
                lists.add(candidates[i])
                DFS(i + 1, sum + candidates[i], lists)
                lists.removeLast()
            }
        }
        DFS(0, 0, ArrayDeque())
        return res
    }
}
```