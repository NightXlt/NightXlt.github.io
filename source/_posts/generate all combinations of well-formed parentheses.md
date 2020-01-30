title:  Generate all combinations of well-formed parentheses.
date: 2019-1-25
tags: [Leetcode,腾讯]
categories: algorithm
description: 　　
---
## Problem description
  ### Given n pairs of parentheses（括号）, write a function to generate all combinations of well-formed parentheses.
 ## Examples
``` java
Example 1:
Input: 3
Output: 
[
  "((()))",
  "(()())",
  "(())()",
  "()(())",
  "()()()"
]
```
## Solution
　　和Combinations of a Phone Number类似，都是用回溯法，不过具体运用有些差异。如需要左右指针确定添加左括号还是右括号。先添加左括号深搜下去，直至left == n(PS:左右括号数目均为n)，再回溯添加右括号。

## Code

```java
 class Solution {
       public List<String> generateParenthesis(int n) {
        List<String> strings = new ArrayList<>();
        group(strings, "", 0, 0, n);
        return strings;
    }

    public void group(List<String> strings, String str, int left, int right, int n) {
        if (str.length() == n * 2) {
            strings.add(str);
        }
        if (left < n) {
            group(strings, str + '(', left + 1, right, n);
        }
        if (right < left) {
            group(strings, str + ')', left, right + 1, n);
        }
    }
}
```