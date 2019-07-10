title: Simplify path
date: 2018-12-30
tags: [Leetcode,字节跳动]
categories: algorithm
description: Simplify Unix path
---
## Problem description
  ### Given an absolute path for a file (Unix-style), simplify it. 
 ## Examples
``` java
Example 1:
Input ： "/home/"
Output ： “/home”
```
```java
Example 2:
Input ： "/a/../../b/../c//.//"
Output ： “/c”
```
```java
Example 3:
Input ： "/a/./b/../../c/"
Output ： “/c”
```

## Solution
　　边界情况考虑好，当为/../，即已经访问到根目录时，再往前是无法访问的。返回“/”

## Code

```java
 class Solution {
     
    public String simplifyPath(String path) {
        String[] strings = path.split("/", 0);
        Stack<String> stack = new Stack<>();
        for (int i = 0; i < strings.length; i++) {
            if ( ".".equals(strings[i]) || "".equals(strings[i])) {
                continue;
            } else if ("..".equals(strings[i])) {
                if (!stack.empty()) {
                    stack.pop();
                }
            } else {
                stack.push(strings[i]);
            }
        }
        StringBuilder sb = new StringBuilder();
        while (!stack.isEmpty()) {
            sb.insert(0, stack.pop());
            sb.insert(0, '/');
        }
        if (sb.length() == 0) {
            sb.append('/');
        }
        return sb.toString();
    }

}
```