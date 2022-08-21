---
title: 剑指Offer-day1：数组中的重复元素
date: 2021-03-04 15:15:15
tags: 数据结构与算法
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/data.jpg
description: 这个系列的文章就用来记录我在Leetcode上刷的剑指Offer算法题目。
---
## 前言

这个系列的文章就用来记录我在Leetcode上刷的剑指Offer算法题目。

## 题目

找出数组中重复的数字。

在一个长度为`n`的数组`nums`里的所有数字都在`0～n-1`的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

示例：

	输入：
	[2, 3, 1, 0, 2, 5, 3]
	输出：2 或者 3

## 解题

找重复元素很容易想到暴力解法，即遍历每个元素看它是否与其它元素重复。容易知道暴力解法的时间复杂度为O(n^2)。

进一步优化思路，要注意到题目中的说明，长度为n的数组中每个元素值为0~n-1，也就是说，x为数组中的元素，而访问nums[x]不会越界。

所以，我们可以使用哈希的思想进行解题，具体做法也不用使用哈希表这个数据结构，而是用一个布尔数组来实现哈希的效果。

**初始化一个长度相等的布尔数组flag，默认值全为false。然后用i循环遍历nums，flag[nums[i]]就可以看成是nums[i]的哈希值，如果当遍历nums[i]时，发现flag[nums[i]]的值已经为true了，说明之前已经有重复的元素进行"哈希"了。**

代码如下：

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        int n = nums.length;
        boolean[] flag = new boolean[n];
        for(int i = 0 ; i < n ; i++){
            if(flag[nums[i]]){
                return nums[i];
            }
            flag[nums[i]] = true;
        }
        return -1; //返回-1是找不到重复元素
    }
}
```

这个方法的时间和空间复杂度均为O(n)，还可以进一步改进。

**前面说了，nums[x]不会越界，那我们遍历nums，将nums中的元素放回与它的值相对应的数组下标位置，即nums[i]=i，如果在放回nums[i]的时候，发现nums[i]的值与要放回位置上的元素值相同，就说明元素重复了。这可以看成是一种原地排序。**

代码如下：

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        for(int i = 0 ; i < nums.length ; i++){
            while(nums[i] != i){
                if(nums[i] == nums[nums[i]]){
                    return nums[i];
                }
		//放回的时候要交换
                int temp = nums[i];
                nums[i] = nums[temp];
                nums[temp] = temp;
            }
        }
        return -1;
    }
}
```

时间复杂度仍为O(n)，空间复杂度优化成了O(1)。