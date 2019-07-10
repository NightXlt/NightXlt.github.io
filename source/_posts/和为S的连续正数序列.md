title: 和为S的连续正数序列
date: 2019-3-8
tags: [Leetcode,剑指Offer]
categories: algorithm
description: 双指针
---
## Problem description
  ### 输入一个正数s,打印出所有和为s的连续正数序列（至少含有两个数）。例如输入15，由于1+2+3+4+5=4+5+6=7+8=15；所以打印出三个连续序列1~5,4~6,7~8；

## Solution
从递增数组中两个数和为s的数得到启示，我们也可以设置两个指针，一个指向当前序列的最小的数，一个指向当前序列最大的数。

设置mid变量，赋值为(1+s)/2，因为和为s的序列至少包括两个数，所以只需small指针遍历到s即可

1. 设置两个指针，一个为small指向当前正数序列中最小的数，一个为big指向当前正数序列中最大的数；
2. 若是当前的正数序列之和大于S，那么缩小序列范围，让small指针不停往前走，直到等于S或者small > mid；
3. 若是当前的正数序列之和小于S，那么扩大序列范围，让big指针不停往前走，直到和为S停止；



## Code
```java
 void findContinuousSequence(int s) {
        if (s < 3) {
            return;
        }
        int small = 1;
        int big = 2;
        int mid = (1 + s) / 2;
        int curSum = small + big;//当前和不从0开始计算
        while (small < mid) {
            if (curSum == s) {//加上small后和是否相等
                print(small, big);
            }
            while (curSum > s && small < mid) {
                curSum -= small;
                small++;
                if (curSum == s) {//减去small后和是否相等
                    print(small, big);
                }
            }
            big++;
            curSum += big;
        }
    }
	    void print(int small, int big) {
        StringBuilder s = new StringBuilder();
        for (int i = small; i <= big; i++) {
            s.append(i).append(" ");
        }
        System.out.println(s.toString());
    }

```