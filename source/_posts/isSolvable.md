title:  Verbal Arithmetic Puzzle
date: 2020-5-10
tags: [Leetcode,回溯]
categories: algorithm

description: 口算难题
---

## Problem description

https://leetcode-cn.com/problems/verbal-arithmetic-puzzle/

## Solution

SEND + MORE = MONEY，移动下 SEND + MORE - MONEY = 0

将其转为 10 进制的数字，再合并同类项得到 
$$
S \* 1000 + E \* 91 - N \* 90 + D - M \* 9000 - O \* 900 + R \* 10 - Y = 0
$$
我们通过 map 记录 字符与系数的对应关系。此外还需要记录首项字符，避免出现前导 0 的情况。

回溯过程

- 判断是否已经遍历光了上述方程中的字符，遍历光了，返回sum是否等于 0
- 从`未访问的字符`中取出一项用来枚举其映射的数字
  - 枚举时，如果该数字已被访问或 （映射数字为 0 且 字符为首字符），则跳过
  - 将该数字标记为使用过防止重复使用
  - 进行深搜，如果该映射数字能够找到一组解则返回 true
  - 否则，回溯数字标记状态
  - 枚举所有数字仍未找到解时，则将该字符添加回`未访问字符`中,并返回 false

## Code

```kotl
import kotlin.math.pow

class Solution {
    fun isSolvable(words: Array<String>, result: String): Boolean {
        val map = HashMap<Char, Int>()
        val firstChar = HashSet<Char>()
        for (word in words) {
            firstChar.add(word[0])
            word.forEachIndexed { index, c ->
                map[c] = map.getOrDefault(c, 0) + 10.0.pow(word.length - 1 - index).toInt()
            }
        }
        firstChar.add(result[0])
        result.forEachIndexed { index, c ->
            map[c] = map.getOrDefault(c, 0) - 10.0.pow(result.length - 1 - index).toInt()
        }
        val usedChar = map.keys.toMutableList()
        return dfs(map, firstChar, usedChar, BooleanArray(10), 0)
    }

    fun dfs(map: HashMap<Char, Int>, firstChar: HashSet<Char>, usedChar: MutableList<Char>,
            usedDigits: BooleanArray, sum: Int): Boolean {
        if (usedChar.isEmpty()) return sum == 0
        val c = usedChar.removeAt(0)
        for (i in 0..9) {
            if (usedDigits[i]) continue
            if (i == 0 && firstChar.contains(c)) continue
            usedDigits[i] = true
            if (dfs(map, firstChar, usedChar, usedDigits, sum + map.getOrDefault(c, 0) * i)) return true
            usedDigits[i] = false
        }
        usedChar.add(0, c)
        return false
    }
}
```
