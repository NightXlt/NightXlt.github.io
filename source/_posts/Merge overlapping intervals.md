title:  Merge overlapping intervals
date: 2019-1-2
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### Given a collection of intervals, merge all overlapping intervals.
 ## Examples
``` java
Example 1:
Input: [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Explanation: Since intervals [1,3] and [2,6] overlaps, merge them into [1,6].
```
```java
Example 2:
Input: [[1,4],[4,5]]
Output: [[1,5]]
Explanation: Intervals [1,4] and [4,5] are considered overlapping.
```

## Solution
　　这道题和OJ上的看电视时间排序一题类似，以start递增排序，判断前一段区间的end是否大于后一段区间的start来进行合并。尽管AC了，但对象排序颇为耗时。看了解析，将start,end拆开放在两个数组。用一个索引记录区间。当 i <length - 1且 start[i+1] <= end[i].

## Code

```java
 class Solution {
    static class Interval {
        int start;
        int end;

        Interval() {
            start = 0;
            end = 0;
        }

        Interval(int s, int e) {
            start = s;
            end = e;
        }
    }

    public List<Interval> merge(List<Interval> intervals) {
        int lenth = intervals.size();
        if (lenth == 0 || lenth == 1) {
            return intervals;
        }
        int starts[] = new int[lenth];
        int ends[] = new int[lenth];
        for (int i = 0; i < lenth; i++) {
            Interval interval = intervals.get(i);
            starts[i] = interval.start;
            ends[i] = interval.end;
        }
        Arrays.sort(starts);
        Arrays.sort(ends);
        List<Interval> results = new ArrayList<>();
        results.add(intervals.get(0));
        Interval currentInterval, nextInterval;
        int cur = 0;
        for (int i = 1; i < lenth; i++) {
            currentInterval = results.get(cur);
            nextInterval = intervals.get(i);
            if (currentInterval.end >= nextInterval.start) {
                currentInterval.end = Math.max(currentInterval.end, nextInterval.end);
                results.set(cur, currentInterval);
            } else {
                results.add(nextInterval);
                cur++;
            }
        }
        return results;
    }

    public static void main(String[] args) {
        List<Interval> list = new ArrayList<>();
        list.add(new Interval(1, 3));
        list.add(new Interval(2, 6));
        list.add(new Interval(8, 10));
        list.add(new Interval(15, 18));
        new Solution().merge(list);
    }
}

```