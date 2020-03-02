title:   Number of Valid Words for Each Puzzle
date: 2020-2-26
tags: [Leetcode,位运算]
categories: algorithm

description: 猜字谜
---

## Problem description

## https://leetcode-cn.com/problems/number-of-valid-words-for-each-puzzle/

## Solution

参考https://leetcode-cn.com/problems/number-of-valid-words-for-each-puzzle/solution/wei-yun-suan-mapying-she-by-jameywoo/

题目中word 匹配 puzzle 需满足两个条件。

1. word 的首字母必在 puzzle 字符串内
2. word 的所有字母必在 puzzle 字符串内，即 word 是 puzzle 的`子集`

直观的解法直接暴力解决，时间复杂度达到 $10^9$

> 模式串：aaabbbbcccc 的模式串为 abc.即 unique string

思考出 通过位运算 提取出 word 和 puzzle 中的模式串进行比较但是脑海中 比较时的遍历是不可避免。

咦，我们除了将 puzzle 和 word 进行一一比较外有没其他办法比较得出满足条件的word 呢？ 

题目 中有留下puzzles[i].length == 7这个点，这 意味着 puzzle 的子集最大为 $2^7$, 题解中紧扣子集这一点，既然 word 是 puzzle 的子集（字符串），那 word 的模式串必是 puzzle 模式串的子集。找出 puzzle 的子集后，如果该子集包含 puzzle 的首字母，就根据映射关系 取出 该子集对应 word中出现的次数 进行累加。这里的映射关系通过 map建立。

此外最后最为 tricky 的一点即怎么寻找 puzzle 的子集，题解中的做法神乎其神。

```
for (int j = k; j; j = (j - 1)&k) {}
```

k 为模式串， 当 j > 0时表明还有子集，通过 j = (j - 1) & 初始值 k 逐个求出子集 j。

如 abc，对应二进制模式串k = 111， j = 111, 110,  101, 100, 011, 010, 001, 000. 这样就将模式串的所有子串找到了。我们知道 j & j-1 会将 j 最右侧的 1 置为 0，而 j-1 & k 会逐个求出k 的子集。因为每次减一的过程，势必会在原有的基础上产生 1 的变动，但需要保证是 k 的子集故通过& k 进行保证。只能感叹，大佬 666 啊。	更加灵活的是可以应用在求数组元素的子集，令数组下标作为 k 值 = 111...111,1表示选中该数组下标对应的元素，每次减一进行遍历，直至 k == 0, 因此 每次我们需要遍历 k 中的二进制 1 的位置选取对应数组元素，求数组元素子集问题的时间复杂度为 $O(n * 2^n)$

时间复杂度：$O(words.length⋅words[i].length+puzzles.length()) = O(5 * 10^6)$

空间复杂度：要额外的空间存放哈希表以及答案数组， $O(words.length+puzzles.length) = O(10^5)$

## Code

```java
class Solution {
    fun findNumOfValidWords(words: Array<String>, puzzles: Array<String>): List<Int> {
        val map = HashMap<Int, Int>()
        words.forEach { s ->
            var flag = 0
            s.forEach {
                flag = flag or (1 shl (it - 'a'))
            }
            map[flag] = map.getOrDefault(flag, 0) + 1
        }
        val list = mutableListOf<Int>()
        puzzles.forEach { s ->
            var flag = 0
            list.add(0)
            s.forEach {
                flag = flag or (1 shl (it - 'a'))
            }
            var i = flag
            while (i > 0) {
                if (i and (1 shl (s[0] - 'a')) != 0) {
                    list[list.lastIndex] += map.getOrDefault(i, 0)
                }
                i = (i - 1) and flag
            }
        }
        return list
    }
}
```