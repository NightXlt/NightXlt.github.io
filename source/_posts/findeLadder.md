title:     Word Ladder II
date: 2020-3-17
tags: [Leetcode,BFS]
categories: algorithm

description: 单词接龙 II
---

## Problem description

https://leetcode-cn.com/problems/word-ladder-ii/

## Solution

给定两个单词，一个作为开始，一个作为结束，还有一个单词列表。然后依次选择单词，只有当前单词到下一个单词只有一个字母不同才能被选择，然后新的单词再作为当前单词，直到选到结束的单词。输出这个的最短路径，如果有多组，则都输出。

一开始是想用回溯来做，深搜过程中判断当前单词和前一个单词和上个单词相差是否为 1，为 1 则进行深搜且回溯，最后将得到的结果集中筛选出最短路径。然而在一个 case 中发现了问题，如果 endWord 出现在中间的话而且 endWord 和之前的 word 相差会大于一就直接 continue 跳过，而后面又没有 endWord，这样会出现漏解的情况。问题出在按数组顺序进行深搜。那遍历完一个元素后应该怎么获取下一个元素进行深搜呢？有没什么办法找到所有和当前 word 相差一的多个 word呢？想想哈，暴力试试。遍历 26 个字母，将每个字母设置为当前word 中每个字母，然后判断设置后的 word 是否在 list 中存在。

Like this, 这样找到他的邻居，也避免深搜中 endWord 在中间的 case。

```kotlin
private ArrayList<String> getNeighbors(String node, Set<String> dict) {
    ArrayList<String> res = new ArrayList<String>();
    char chs[] = node.toCharArray();

    for (char ch = 'a'; ch <= 'z'; ch++) {
        for (int i = 0; i < chs.length; i++) {
            if (chs[i] == ch)
                continue;
            char old_ch = chs[i];
            chs[i] = ch;
            if (dict.contains(String.valueOf(chs))) {
                res.add(String.valueOf(chs));
            }
            chs[i] = old_ch;
        }

    }
```

可是在数据量大时导致了超时。为什么呢。我在遍历时会遍历很多无谓的分支，比如出现比当前 res 中的 list.size 大时,就直接不需要再遍历下去了。我们可以用一个 min 值记录最小的 list.size。如果 path 超过这个长度时就直接不再遍历下去。满怀欣喜改后还是超时。

![bfs 中重复节点](/images/ladder_bfs_redundance.jpg)

为啥呢？遍历中在不同层会存在一样的节点abc，其实如果在第 K 层我们发现这个节点在第 1..k-1层出现过那该节点就不需要访问了。我们可以使用一个 visited 的 set记录 1..k-1层出现过的节点，访问每一层时使用 visited 进行判断是否出现过，同时将新出现过的节点加入到一个全新的 subVisited 中，在该层访问完后，再将 subVisited 加入到 visited 中。为什么需要多此一举，不直接加入到 visited呢？因为假设在某一层中我们找到了endWord ，如果直接添加进 visited 中，那么后续的同层节点如果也有 endWord，但因为 visited 中已经包含 endWord就会导致无法添加该 path，导致漏解。

此外， 我们还是在寻找 min 时仍旧会遍历一些超长的路径，逐步找出最小的 min。那有没办法一步找出最小的 min 呢？而非逐步对 min 取 min(min, path.size)。 看了题解是用了尘封已久的 BFS，嗨呀，如果是 BFS 的话，就可以逐层遍历直至找到 endWord。

![bfs 图解](/images/ladder_bfs.jpg)

那就不需要用啥子 min 标识了，将 bfs 中queue 里面放path: 从 beginWord节点 到当前word节点前的路径如

"ab","if", {"cd","af","ib","if"}；
（1）第一层：[{"ab"}]
（2）第二层：[{"ab","af"}、{"ab","ib"}]
（3）第三层：[{"ab","af","if"}、{"ab","ib","if"}]

遍历 queue，逐个取出同层中的所有 path，再从path 中取出最后一个 word：上层遍历的 word，找出它的邻居，找出其他符合条件的 word 添加进入 一个新的 path 中。这里的条件指的是wordList 中包含该 word 以及visited 中不包含 word。如果找到了 endWord，将 isTraversalEnd 置为 true，将 path 添加进 res 中，然后继续遍历完同层元素即可终止遍历。

因此终止条件应该写为

```kotlin
while(queue.isNotEmpty() && !isTraversalEnd)
```

时间复杂度：$O(k^d)$, 每个节点至多有 k 个邻居节点，树高至多为 d

空间复杂度：$O(N)$, N: 节点个数

## Code

```kotlin
import java.util.*
import kotlin.collections.HashSet

class Solution {
    fun findLadders(beginWord: String, endWord: String, wordList: List<String>): List<List<String>> {
        val dict = HashSet(wordList)
        val res:  MutableList<List<String>> = mutableListOf()
        if (!dict.contains(endWord)) { //确保能够找到 endWord,否则会死循环
            return res
        }

        val queue = ArrayDeque<MutableList<String>>()
        queue.add(mutableListOf(beginWord)) //存放每层路径
        val visited = HashSet<String>() //存储 1.。k-1层访问过的word
        visited.add(beginWord)
        var isTraversalEnd = false //是否找到 endWord
        while (!queue.isEmpty() && !isTraversalEnd) {
            val subVisited = HashSet<String>() //记录 k 层访问的 word
            val size = queue.size
            repeat(size) {
                val path = queue.poll() //获取到之前的 path
                val previousWord = path[path.size - 1] //获取上一层的 word
                val chars = previousWord.toCharArray() 
                for (i in 0 until chars.size) { //取出每个字符进行暴力遍历
                    val temp = chars[i] 
                    for (c in 'a'..'z') {
                        if (c == temp) { //如果 temp 和之前的一致就不需要遍历了，因为其已经在集合中了。
                            continue
                        }
                        chars[i] = c //暴力寻chars 的 neighbor
                        val nextWord = String(chars)
                        if (!dict.contains(nextWord) || visited.contains(nextWord)) {
                            continue
                        }
                        val newPath = path.toMutableList() //防止出现集合引用导致出现重复的情况，list 引用问题
                        newPath.add(nextWord)
                        if (nextWord == endWord) { //不能直接 return，同层需要遍历完毕
                            res.add(newPath)
                            isTraversalEnd = true
                        }
                        queue.add(newPath)
                        subVisited.add(nextWord)
                    }
                    chars[i] = temp //暴力遍历完后恢复 chars 的第 i 个字符
                }
            }
            visited.addAll(subVisited) //将 k 访问的 word 添加到 1..k-1层中的 visited 中
        }
        return res
    }

}
```

## Solution 2

第二种解法是使用的是双向 BFS + DFS. 通过双向 BFS记录每个节点的邻居节点。DFS 根据记录下的邻居进行深搜找到最短路径。

![double bfs 图解](/images/double_bfs.jpg)

双向 bfs 指的是在 BFS 过程中，会先后从不同方向进行 BFS，即从上至下、从下至上;当两个方向节点产生共同节点时，就产生了最短路径。 使用两个集合begin,end记录两个方向访问过的节点，每次从节点数少的集合进行遍历。当begin 节点数目为空时，表明不存在公共节点，直接返回 false。

时间复杂度：$O(k^{\frac{d}{2}})$, 双向遍历降低了一半树高，每个节点至多有 k 个邻居节点，树高至多为 d

空间复杂度：$O(N)$ N: 节点个数

## Code

```kotlin
import java.util.*
import kotlin.collections.HashSet

class Solution {
    fun findLadders(beginWord: String, endWord: String, wordList: List<String>): List<List<String>> {
        val dict = HashSet(wordList) //未访问过的节点集合
        val res: MutableList<List<String>> = mutableListOf()
        if (!dict.contains(endWord)) {
            return res
        }
        val begin = HashSet<String>()
        begin.add(beginWord) //从上之下集合
        val end = HashSet<String>()
        end.add(endWord) //从下至上集合
        val map = HashMap<String, MutableList<String>>() //存储每个节点的邻居节点
        if (doubleBfs(dict, begin, end, map, true)) {
            dfs(map, res, beginWord, endWord, ArrayDeque())
        }
        return res
    }

    private fun dfs(map: HashMap<String, MutableList<String>>, res: MutableList<List<String>>, beginWord: String, endWord: String, queue: ArrayDeque<String>) {
        queue.add(beginWord) //beginWord 可能不在集合中，需要单独添加
        if (beginWord == endWord) {
            res.add(queue.toList())
            queue.removeLast()
            return
        }
        if (map.containsKey(beginWord)) {
            map[beginWord]?.forEach {
                dfs(map, res, it, endWord, queue)
            }
        }
        queue.removeLast()
    }

    fun doubleBfs(dict: HashSet<String>, begin: Set<String>, end: Set<String>, map: HashMap<String, MutableList<String>>, isTopDown: Boolean): Boolean { //寻找每个节点的邻居节点
        if (begin.isEmpty()) { //当一个方向节点数为空仍未找到 中间联通节点时，返回 false
            return false
        }
        if (begin.size > end.size) { // 选取节点数小的方向进行遍历
            return doubleBfs(dict, end, begin, map, !isTopDown)
        }
        dict.removeAll(begin)
        dict.removeAll(end) //去除已访问节点
        var isTraversalEnd = false //是否遍历结束
        val visited = HashSet<String>() //记录本层新增节点（未在 begin 和 end 中出现过）
        for (word in begin) {
            val chars = word.toCharArray()
            for (i in 0 until chars.size) {
                val temp = chars[i]
                for (c in 'a'..'z') { //暴力枚举可能的邻居节点
                    if (c == temp) {
                        continue
                    }
                    chars[i] = c
                    val neighborWord = String(chars)
                    val key = if (isTopDown) word else neighborWord //根据访问顺序确定 key,value.确保 key 是从上之下方向中的上面，而value 是下面。
                    val value = if (isTopDown) neighborWord else word
                    val neighborWords = map.getOrDefault(key, mutableListOf())

                    if (end.contains(neighborWord)) { //如果有公共节点，遍历完本层即可结束遍历。值得注意的是，后续本层中的公共节点还是可以进入这个 if 循环进行添加
                        isTraversalEnd = true
                        neighborWords.add(value)
                        map[key] = neighborWords
                    }
                    if (isTraversalEnd || !dict.contains(neighborWord)) { //如果找到了目标节点或者dict 中不包含该邻居节点，跳过本次循环
                        continue
                    }
                    visited.add(neighborWord)
                    neighborWords.add(value)
                    map[key] = neighborWords
                }
                chars[i] = temp
            }
        }
        return isTraversalEnd || doubleBfs(dict, visited, end, map, isTopDown) //使用短路或进行判断是否找到公共节点，找到就直接返回，否则继续以本层新增加的节点作为下层的 begin 继续深搜
    }
}
```





