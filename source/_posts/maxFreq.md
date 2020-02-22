title:   Maximum Number of Occurrences of a Substring
date: 2020-2-20
tags: [Leetcode,暴力]
categories: algorithm

description: 子串的最大出现次数
---

## Problem description

  ### https://leetcode-cn.com/problems/maximum-number-of-occurrences-of-a-substring/


## Solution

暴力解决匹配，按照 minSize 进行匹配，但凡 maxSize匹配的其minSize 子串一定也是符合要求的，反过来亦然。如“aababcaab”的结果是 aab,其子串“ab”必然也至少出现了两次，可能大于两次 ，原因是这里还需要加上minSize 的限制确保子串预期一致，故每次遍历以最小滑动窗口进行遍历，降低时间复杂度。

如何保证子串中的不同字母的数目小于等于 maxLetters，通过 set 进行筛选。如果 set 的 size 大于 maxLettters 即不满足条件

在满足条件的情况下，将结果放到 map 中进行存储，最终进行比较子串其出现次数count。

时间复杂度： O(n)

空间复杂度：O(n)

## Code

```java
class Solution {
    fun maxFreq(s: String, maxLetters: Int, minSize: Int, maxSize: Int): Int {
        if (s.length < maxLetters || s.length < maxSize) {
            return 0
        }
        val map = HashMap<String, Int>()
        for (i in 0..s.length - minSize) {
            val substring = s.substring(i, i + minSize)
            if (isMatch(substring, maxLetters)) {
                map[substring] = map.getOrDefault(substring, 0) + 1
            }
        }
        return map.maxBy { it.value }?.value ?: 0
    }

    private fun isMatch(substring: String, maxLetters: Int): Boolean {
        val set = HashSet<Char>()
        for (c in substring) {
            set.add(c)
            if (set.size > maxLetters) {
                return false
            }
        }
        return set.size <= maxLetters
    }
}
```