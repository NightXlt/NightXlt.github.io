title:   Design Underground System
date: 2020-3-29
tags: [Leetcode]
categories: algorithm

description: HashMap中标识Key的做法
---

## Problem description

https://leetcode-cn.com/problems/design-underground-system/

## Solution

看似普普通通的一道题，在竞赛时死活没刷出来，感觉被其中的映射关系绕糊涂了。结束后看了下题解。AHA，神奇。对于始发站和进站时间使用两个 HashMap 进行记录，key 是 id, 方便在下站时根据 id 站拿到始发站的时间和站名。 之前执拗在使用map 中包 map 的做法将其省略为一个，想着能省下空间，我擦，写起来时太痛苦了。开始想着一个变量写着简便些，结果简便个毛线。对于 time 和 count 也各使用一个 map，这里的 key 就比较 tricky 了，通过将始发站和出站中间拼上一个\#作为标识。这样计算平均时也相当方便，直接将startStation \# endStation 作为 key，直接取数据即可。这道题指的注意的是这里的 key 设置并不遵循常理单一变量设置，想来 Key 只要唯一即可，真的妙啊。

## Code

```java
class UndergroundSystem {
    val startStations = HashMap<Int, String>()
    val startT = HashMap<Int, Int>()
    val time = HashMap<String, Int>()
    val count = HashMap<String, Int>()

    fun checkIn(id: Int, stationName: String, t: Int) {
        startStations[id] = stationName
        startT[id] = t
    }

    fun checkOut(id: Int, stationName: String, t: Int) {
        val s = startStations.getOrDefault(id, "") + '#' + stationName
        time[s] = time.getOrDefault(s, 0) + t - startT.getOrDefault(id, 0)
        count[s] = count.getOrDefault(s, 0) + 1
    }

    fun getAverageTime(startStation: String, endStation: String): Double {
        val s = "$startStation#$endStation"
        return time[s]!! * 1.0 / count[s]!!
    }
}
```