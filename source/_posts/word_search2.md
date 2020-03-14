title:    Word Search II
date: 2020-3-14
tags: [Leetcode,回溯]
categories: algorithm

description: 单词搜索 II
---

## Problem description

https://leetcode-cn.com/problems/word-search-ii/

## Solution

尽管使用上题的代码逐个遍历，可以通过但执行用时太大了。怎么进行优化呢？目前的解题思路是通过将 words 中每个 word在 board 中深搜，那可以在一次深搜过程中找到所有的 word 吗？答案是可以的，不过需要使用一个 `前缀树`作为数据结构存储 word.

关于前缀树的定义，可参考：https://nightxlt.github.io/2020/03/13/prefix_tree/

为了方便添加 word 在前缀树的结构上进行了两处调整

1. 去除了搜索功能，因为本题需要进行逐步深搜
2. 使用字符串 word 字段记录该树枝上的完整字符串 替代 标识字符结束字段，这样可以更方便在遍历到一个字符串时，直接进行添加。

每当遍历到 board 的一个字符时判断其是否在树中存在。不存在则直接返回。为了避免判断 an和 ang 时的重复遍历前缀树，我们在深搜时将子节点做为新一轮深搜的起始节点。

时间复杂度：  $O(m \* n  \* 4 \* l)$ ,m*n 是 board 的大小， board 上每个点都会被访问到，4：每个顶点有四个邻居，l: max( words[i].length), 至多深搜至max( words[i].length) 次。

空间复杂度：$O(k)$ k: words.size

## Code

```java
class Solution {
    fun dfs(board: Array<CharArray>, node: Trie, row: Int, column: Int, res: MutableList<String>) {
        if (row < 0 || row == board.size || column < 0 || column == board[row].size) {
            return
        }
        val c = board[row][column]
        if (c == '.' || !node.children.containsKey(c)) {
            return
        }
        var trieNode = node
        trieNode.children[c]?.run {
            trieNode = this
        }
        trieNode.word?.run { //如果当前 board 已匹配到一个 word，添加其进入 res，并将其置空防止重复添加
            res.add(this)
            trieNode.word = null
        }
        board[row][column] = '.'
        dfs(board, trieNode, row, column + 1, res) //将子节点作为新一轮深搜起点
        dfs(board, trieNode, row - 1, column, res)
        dfs(board, trieNode, row + 1, column, res)
        dfs(board, trieNode, row, column - 1, res)
        board[row][column] = c
    }

    fun findWords(board: Array<CharArray>, words: Array<String>): List<String> {
        if (board.isEmpty() || words.isEmpty()) {
            return emptyList()
        }
        val res = mutableListOf<String>()
        val root = Trie()
        words.forEach {
            root.insert(it)
        }
        for (i in 0 until board.size) {
            for (j in 0 until board[0].size) {
                dfs(board, root, i, j, res)
            }
        }
        return res
    }

}
class Trie {

    var children = HashMap<Char, Trie>()
    var word: String? = null

    /** Inserts a word into the trie. */
    fun insert(word: String) {
        var cur = this
        word.forEach {
            if (!cur.children.containsKey(it)) {
                cur.children[it] = Trie()
            }
            cur = cur.children[it]!!
        }
        cur.word = word
    }
}
```