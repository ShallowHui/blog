---
title: 剑指Offer-day2：二维数组中的查找
date: 2021-03-10 22:29:37
tags: 数据结构与算法
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/data.jpg
description: 这个系列的文章就用来记录我在Leetcode上刷的剑指Offer算法题目。
---
## 题目

在一个`n * m`的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

示例：

有如下二维数组：

	[
  		[1,   4,  7, 11, 15],
  		[2,   5,  8, 12, 19],
  		[3,   6,  9, 16, 22],
  		[10, 13, 14, 17, 24],
  		[18, 21, 23, 26, 30]
	]

给定 target = 5，返回`true`。

给定 target = 20，返回`false`。

## 解题

容易想到，暴力解法就是遍历这个二维数组，这样时间复杂度为O(mn)。

所以我们要注意到这个二维数组从左到右，从上到下均递增的性质。如果是在一维的递增序列中，我们可以用二分查找这个算法，那么二维呢？

其实，如果从右上角看示例中的这个二维数组，不难看出这有点像一颗二叉搜索树。15的左边11比15小，15的下边19比15大，11、19的左，下边也是。当然，并不是说这个二维数组是一棵真的二叉搜索树，但我们可以利用这个性质。

应该说，一个元素作为根结点，与它的同一行和同一列元素构成一颗二叉搜索树。所以，当要找的target不是根结点时，根据target与根结点的大小关系，我们可以放弃左子树或者右子树不用去查找，也就是抛弃了一行或者一列的元素不用去检索了。

具体代码实现如下：

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        int raw, col;
        if(matrix.length == 0){
            return false;
        }else{
            raw = 0;
            col = matrix[0].length-1;
        }
        while(col >= 0 && raw < matrix.length){
            if(target == matrix[raw][col]){
                return true;
            } else if(target < matrix[raw][col]){
                col--;
            } else{
                raw++;
            }
        }
        return false;
    }
}
```

**一共有n行，m列，所以算法时间复杂度为O(m+n)。**

+ while循环前面的if-else判断是为了通过Leetcode测试时一个输入为空数组的测试用例，~~有点坑哈~~。