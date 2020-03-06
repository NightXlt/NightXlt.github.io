title:    Combination
date: 2020-3-6
tags: [Leetcode,回溯]
categories: algorithm

description: 组合总和
---

## Problem description

https://leetcode-cn.com/problems/combinations/

## Solution

套上回溯的模板很快就可以出来，我在进行 DFS 时传入的是 1..n的数组有没办法去除这一次拷贝呢？既然我们遍历时需要遍历数组，而数组的元素和下标相差个 1，num[i] = i - 1; 那直接遍历 index就好了。可是时间复杂度还是很慢呀。还是需要进行剪枝。uh, 对了遍历索引时，从某位起后续的索引就不需要再进行遍历。比如我们遍历 1..n, 一开始只需要遍历到 n - 3即可，因为 n -3后面的元素从n - 2开始到 n没有四个元素了；Dequeue在添加一个元素后只需要遍历到 n - 2即可。同理添加三个元素后，遍历到 n 即可。这个最大访问索引 maxI，和 n,k,queue.size 是什么关系呢？ 

由定义得 最大索引加上 剩余未访问元素数目再 - 1等于 n.  maxI + k - queueSize - 1 = n, 那 maxI = n - k + queueSize + 1

时间复杂度： $O(kC_n^k)$

时间复杂度： $O(C_n^k)$

## Code

```java
Init: 
fun combine(n: Int, k: Int): List<List<Int>> {
        var res = mutableListOf<List<Int>>()
        fun DFS(nums: IntArray, start: Int, queue: Deque<Int>) {
            if (queue.size == k) {
                val list = queue.toList()
                res.add(list)
                return
            }
            for (i in start until nums.size) {
                    queue.add(nums[i])
                    DFS(nums, i + 1, queue)
                    queue.removeLast()
                }
        }
        DFS( (1..n).toList().toIntArray(), 0, ArrayDeque())
        return res
    }
```



```java
Advanced:
import java.util.*

class Solution {
    fun combine(n: Int, k: Int): List<List<Int>> {
        val res = mutableListOf<List<Int>>()
        fun DFS(start: Int, queue: Deque<Int>) {
            if (queue.size == k) {
                val list = queue.toList()
                res.add(list)
                return
            }
            for (i in start..(n - k + queue.size + 1)) {
                    queue.add(i)
                    DFS(i + 1, queue)
                    queue.removeLast()
                }
        }
        DFS( 1, ArrayDeque())
        return res
    }
}
```