title: Restore  IP address
date: 2018-12-30
tags: [Leetcode,字节跳动]
categories: algorithm
description: Restore  IP address，returning all possible valid IP address combinations.
---
## 题目描述
  ### Given a string containing only digits, restore it by returning all possible valid IP address combinations.
 ## Examples
``` java
Example 1:
Input: "25525511135"
Output: ["255.255.11.135", "255.255.111.35"]
```

## Solution
　　直接暴力DP
　　两点：
 　　　　1. 每个节点不能超过255
 　　　　2. substring（i, j）, j是取不到的
 　　　　3. 不允许出现以0为首长度>1的IP地址，即使是03

  						

## Code

```java
 class Solution {
         public List<String> restoreIpAddresses(String s) {
        int length = s.length();
        List<String> list = new ArrayList<>();
        String s1, s2, s3, s4;
        for (int i = 1; i < 4 && i < length - 2; i++) {
            for (int j = i + 1; j < i + 4 && j < length - 1; j++) {
                for (int k = j + 1; k < j + 4 && k < length; k++) {
                    s1 = s.substring(0, i);
                    s2 = s.substring(i, j);
                    s3 = s.substring(j, k);
                    s4 = s.substring(k, length);
                    if (isValid(s1) && isValid(s2) && isValid(s3) && isValid(s4)) {
                        s1 = s1 + '.' + s2 + '.' + s3 + '.' + s4;
                        list.add(s1);
                    }
                }
            }
        }
        return list;
    }

    private boolean isValid(String s) {
        int length = s.length();
        if (length > 3 || length == 0 || Integer.parseInt(s) > 255 || (s.charAt(0) == '0' && length > 1)) {
            return false;
        }
        return true;
    }

}
```