---
title: 剑指Offer-day4：矩阵中的路径
date: 2021-05-31 22:28:04
tags: 数据结构与算法
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/data.jpg
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/data.jpg
description: 这个系列的文章就用来记录我在Leetcode上刷的剑指Offer算法题目。
---
## 题目

给定一个 m x n 二维字符网格 board 和一个字符串单词 word 。如果 word 存在于网格中，返回 true ；否则，返回 false 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

示例：

输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"

输出：true

如图所示：

![二维网格](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/reflection/matrix.PNG)

## 解题

根据题目要求，同一个单元格不能被重复使用，可以想到，在开始寻找一个符合word字符串的路径的时候可以用深度优先遍历。当然，只用一次深度优先遍历肯定是找不全二维网格中所有可能路径的，毕竟深度优先遍历是每个网格只遍历一次，有些路径是要走到之前遍历过的网格的。

所以思路是，先用两个for循环遍历每一个网格，一旦网格中的字符是word字符串的首字符，就从这个网格开始进行深度优先遍历，寻找可能存在的路径。

代码实现如下：

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        int m = board.length, n = board[0].length;
        int[][] arr = new int[m][n];
        for (int i = 0 ; i < m ; i++) {
            for (int j = 0 ; j < n ; j++) {
                if (DFS(board,word,0,arr,i,j)) {
                    return true;
                }
            }
        }
        return false;
    }

    public boolean DFS(char[][] board, String word, int index, int[][] path, int i, int j) {
        if (board[i][j] != word.charAt(index)) {
            return false;
        }else {
            path[i][j] = index + 1;
	    //下面四个if判断表示往当前网格可达的东南西北四个方向遍历，并且这个递归是一旦寻找到了一个可行性方案，就递归返回结果了
            if (++index < word.length()) {
                if (j < board[0].length-1 && path[i][j+1] == 0) {
                    if (DFS(board,word,index,path,i,j+1))
                        return true;
                }
                if (i < board.length-1 && path[i+1][j] == 0) {
                    if (DFS(board,word,index,path,i+1,j))
                        return true;
                }
                if (j > 0 && path[i][j-1] == 0) {
                    if (DFS(board,word,index,path,i,j-1))
                        return true;
                }
                if (i > 0 && path[i-1][j] == 0) {
                    if (DFS(board,word,index,path,i-1,j))
                        return true;
                }
                path[i][j] = 0;
                return false;
            }else {
                return true;
            }
        }
    }
}
```

时间复杂度是O(m×n×3^k)，m是二维网格的行数，n是列数，k是word字符串的长度。