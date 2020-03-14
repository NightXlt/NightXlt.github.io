title:    Implement Trie (Prefix Tree)
date: 2020-3-13
tags: [Leetcode,回溯]
categories: algorithm

description: 前缀树
---

## Problem description

https://leetcode-cn.com/problems/implement-trie-prefix-tree/

## Solution

[前缀树](https://zh.wikipedia.org/wiki/Trie)有多个树枝，可以用 hashMap 存储其 children 减少多余的存储空间。因为在前缀树中进行 search 一个前缀是需要返回 false 的。比如前缀树中存储 了 apple,我们搜索app 是需要返回 false 的，故当我们遍历给定 word 结束时需要知道前缀树后续是否还有节点。故添加 一个 isFinished 字段进行标识，默认为 false，插入到字符串最后一个字母时会将其置为 true.

插入时间复杂度： $O(m)$, m 为 word长度

空间复杂度： $O(m)$

搜索时间复杂度： $O(m)$

空间复杂度： $O(1)$

查询前缀时间复杂度： $O(m)$

空间复杂度： $O(1)$

## Code

```java
class Trie {

    private var children = HashMap<Char, Trie>()
    private var isFinished = false

    /** Inserts a word into the trie. */
    fun insert(word: String) {
        var cur = this
        word.forEach {
            if (!cur.children.containsKey(it)) {
                cur.children[it] = Trie()
            }
            cur = cur.children[it]!!
        }
        cur.isFinished = true
    }

    /** Returns if the word is in the trie. */
    fun search(word: String): Boolean {
        var cur = this
        word.forEach {
            if (!cur.children.containsKey(it)) {
                return false
            }
            cur = cur.children[it]!!
        }
        return cur.isFinished
    }

    /** Returns if there is any word in the trie that starts with the given prefix. */
    fun startsWith(prefix: String): Boolean {
        var cur = this
        prefix.forEach {
            if (!cur.children.containsKey(it)) {
                return false
            }
            cur = cur.children[it]!!
        }
        return true
    }
}
```