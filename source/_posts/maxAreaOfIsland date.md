title: maxAreaOfIsland
date: 2018-12-31
tags: [Leetcode,字节跳动]
categories: algorithm 
description: Calculate the max area of island.
---
## 题目描述
  ### Given a non-empty 2D array grid of 0's and 1's, an island is a group of 1's (representing land) connected 4-directionally (horizontal or vertical.) You may assume all four edges of the grid are surrounded by water.Find the maximum area of an island in the given 2D array. (If there is no island, the maximum area is 0.)
 ## Examples
``` java
Example 1:
Input: 
[[0,0,1,0,0,0,0,1,0,0,0,0,0],
 [0,0,0,0,0,0,0,1,1,1,0,0,0],
 [0,1,1,0,1,0,0,0,0,0,0,0,0],
 [0,1,0,0,1,1,0,0,1,0,1,0,0],
 [0,1,0,0,1,1,0,0,1,1,1,0,0],
 [0,0,0,0,0,0,0,0,0,0,1,0,0],
 [0,0,0,0,0,0,0,1,1,1,0,0,0],
 [0,0,0,0,0,0,0,1,1,0,0,0,0]]

Output: 6 

```
```java
Example 2:
Input: 
[[0,0,0,0,0,0,0,0]]

Output: 0

```
```java
Example 3:
Input : {0, 1},
           {1, 1}
Output: 3		  
```

## Solution
　　　　注意四个方向的DP，还有DFS中为了降低空间复杂度可以置现有的grid[i][j] = 0;时间复杂度O（mn）（m:岛屿个数，n节点数） ，空间复杂度O(1)
	
## Code

```java
class Solution {
 public int maxAreaOfIsland(int[][] grid) {                                                     
     int maxArea = 0;                                                                           
     for (int i = 0; i < grid.length; i++) {                                                    
         for (int j = 0; j < grid[i].length; j++) {                                             
             if (grid[i][j] != 0) {                                                             
                 maxArea = Math.max(maxArea, DFS(grid, i, j));                                  
             }                                                                                  
         }                                                                                      
     }                                                                                          
     return maxArea;                                                                            
 }                                                                                              
                                                                                                
 public int DFS(int[][] grid, int i, int j) {                                                   
     if (i >= 0 && i < grid.length && j >= 0 && j < grid[i].length && grid[i][j] != 0) {        
         grid[i][j] = 0;                                                                        
         return 1 + DFS(grid, i + 1, j) + DFS(grid, i - 1, j) + DFS(grid, i, j - 1) + DFS(grid, i, j+1);
     }
     return 0;                                                                                  
 }                                                                                              
                                                                                                  
}                                                           
```