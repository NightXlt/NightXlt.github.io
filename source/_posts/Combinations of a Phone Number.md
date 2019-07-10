title: Combinations of a Phone Number
date: 2019-1-16
tags: [Leetcode,字节跳动]
categories: algorithm
description:   　　
---
## Problem description
  ### Given a string containing digits from 2-9 inclusive, return all possible letter combinations that the number could represent.Although the above answer is in lexicographical order, your answer could be in any order you want.
 ## Examples
``` java
Example 1:
Input: "23"
Output: ["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
```
## Solution
　　起初是通过HashMap存储数字和拼音映射，但深搜那里实在搞不定，检索了相关算法发现参考算法是回溯。那回溯和DFS有什么差异呢？
  　　其实回溯就是深搜的一种实现，深搜是无序的，而回溯是有序的。比如这道题必须是2对应的字符在前面，3的字符在后面。深搜已访问过的节点不再访问，所有点仅访问一次。而回溯不一样，已访问过的节点可能会再次访问，也可能存在未曾访问过的节点。这道题一个值得注意的点是拼接字符串不能用stringBuilder,其会将一次深搜的结果拼在一起，如第一个ex中，结果会变为ad,ade,adef...这不是我需要的结果。所以必须使用string。每次拼接会new一次。

## Code
```java
 class Solution {
        public List<String> letterCombinations(String digits) {
        List<String> strings = new ArrayList<>();
        if (digits == null || digits.length() == 0) {
            return strings;
        }
        int len = digits.length();
        String[] dict = {"abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};
        String arr[] = new String[len];
        for (int i = 0; i < len; i++) {
            arr[i] = dict[digits.charAt(i) - '0' - 2];
        }
        group(arr, 0, "", strings);
        
        return strings;
    }

    private String group(String[] arr, int startIndex, String stringCombination, List<String> strings) {
        char[] chars = arr[startIndex].toCharArray();
        for (int i = 0; i < chars.length; i++) {
            if (startIndex == arr.length - 1) {
                strings.add(stringCombination + chars[i]);
            } else {
                group(arr, startIndex + 1, stringCombination + chars[i], strings);
            }
        }
        return stringCombination;
    }

}
```