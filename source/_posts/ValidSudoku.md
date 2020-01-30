title: ValidSudoku
date: 2019-2-3
tags: [Leetcode,腾讯]
categories: algorithm
description: 　　
---
## Problem description
  ### Determine if a 9x9 Sudoku board is valid. Only the filled cells need to be validated according to the following rules:

1. Each row must contain the digits 1-9 without repetition.
2. Each column must contain the digits 1-9 without repetition.
3. Each of the 9 3x3 sub-boxes of the grid must contain the digits 1-9 without repetition.

![sudo](/images/sudu.png)
The Sudoku board could be partially filled, where empty cells are filled with the character '.'.
 ## Examples
``` java
Example 1:
Input:
[
  ["5","3",".",".","7",".",".",".","."],
  ["6",".",".","1","9","5",".",".","."],
  [".","9","8",".",".",".",".","6","."],
  ["8",".",".",".","6",".",".",".","3"],
  ["4",".",".","8",".","3",".",".","1"],
  ["7",".",".",".","2",".",".",".","6"],
  [".","6",".",".",".",".","2","8","."],
  [".",".",".","4","1","9",".",".","5"],
  [".",".",".",".","8",".",".","7","9"]
]
Output: true
```
```java
Example 2:
Input:
[
  ["8","3",".",".","7",".",".",".","."],
  ["6",".",".","1","9","5",".",".","."],
  [".","9","8",".",".",".",".","6","."],
  ["8",".",".",".","6",".",".",".","3"],
  ["4",".",".","8",".","3",".",".","1"],
  ["7",".",".",".","2",".",".",".","6"],
  [".","6",".",".",".",".","2","8","."],
  [".",".",".","4","1","9",".",".","5"],
  [".",".",".",".","8",".",".","7","9"]
]
Output: false
Explanation: Same as Example 1, except with the 5 in the top left corner being 
    modified to 8. Since there are two 8's in the top left 3x3 sub-box, it is invalid.
```
## Solution
　　用3个HashSet，分别保存第i行、第i列和第i个3x3的九宫格（i个九宫格也是从0开始）中的元素，每处理一个元素，若不为空，将正在处理的当前元素，添加到所属的行、列以及3x3的九宫格中，若添加失败，表明所属的行、列或者3x3九宫格中有重复元素，返回false；若全部扫描完，返回true。九宫格的行列转换代码没想出来。
  
  （ i / 3 ） x 3 得到九宫格的起始行数即九宫格左上角块所处行。（ i / 3 ） * 3  + j / 3得到九宫格中任一对应行（PS: 0 <= j <= 8）。
  
 ( i % 3)  x 3 得到九宫格起始列数即九宫格左上角所处列。i % 3 * 3 + j % 3得到九宫格中任一对应列（PS: 0 <= j <= 8）。
## Code

```java
     public boolean isValidSudoku(char[][] board) {
        //最外层循环，每次循环并非只是处理第i行，而是处理第i行、第i列以及第i个3x3的九宫格
        for (int i = 0; i < 9; i++) {
            HashSet<Character> row = new HashSet<>();
            HashSet<Character> col = new HashSet<>();
            HashSet<Character> subBoxes = new HashSet<>();
            for (int j = 0; j < 9; j++) {
                if ('.' != board[i][j] && !row.add(board[i][j]))
                    return false;
                if ('.' != board[j][i] && !col.add(board[j][i]))
                    return false;
                int m = i / 3 * 3 + j / 3;
                int n = i % 3 * 3 + j % 3;
                if ('.' != board[m][n] && !subBoxes.add(board[m][n]))
                    return false;
            }
        }
        return true;
    }

```