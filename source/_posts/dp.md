---
title: Leetcode / dp
mathjax: false
date: 2021-12-27 15:09:43
tags: leetcode
---

> 记录前边问题结果，利用递推公式求解，空间换时间。

<!-- more -->

## 一维数组dp

> 509  斐波那契数/70. 爬楼梯
> 338  比特位计数(dp + 位运算)
> 55  跳跃游戏
> 45  跳跃游戏 II（dp/贪心）
> 198  打家劫舍
> 213 打家劫舍 II
> 121  最佳买卖股票时机(一次买卖)
> 122  买卖股票的最佳时机 II(多次买卖)
> 264/剑指49 丑数【数学】

### 子序列与子串

子序列：子序列不要求元素连续。
> 300  最长递增子序列
> 516 最长回文子序列
> 1143/剑指95 最长公共子序列

子串：要求数组元素必须连续
> 53/剑指42 连续子数组的最大和 (子串最大和)
> 674 最长递增子串
> 5  最长回文子串
> 最长公共子串

## 二维数组dp

> 62  不同路径
> 63  不同路径 II
> 剑指47  礼物的最大价值(最大路径和) / 64  最小路径和
> 120  三角形最小路径和

## dp背包

- 给定一个背包容量 target，一个数组 nums(物品)，能否按一定方式选取 nums 中的元素得到target。背包容量 target 和物品 nums 的类型可能是数，也可能是字符串
- target可能题目已经给出(显式)，也可能需要从题目的信息中挖掘出来(非显式)(常见的非显式 target 比如sum/2 等)
- 选取方式有常见的以下几种：每个元素选一次/每个元素选多次/选元素进行排列组合
  1、0/1背包：每个元素最多选取一次, 外循环 nums,内循环 target, target 倒序且 target>=nums[i];
  2、完全背包：每个元素可以重复选择, 外循环 nums,内循环 target, target 正序且 target>=nums[i];
  3、组合背包：背包中的物品要考虑顺序, 外循环target,内循环 nums, target 正序且 target>=nums[i];
  4、分组背包：不止一个背包，需要遍历每个背包, 三重循环：外循环背包bags,内部两层循环根据题目的要求转化为1,2,3三种背包类型的模板

而每个背包问题要求的也是不同的，按照所求问题分类，又可以分为以下几种：
1、最值：要求最大值/最小值

```
dp[i] = max/min(dp[i], dp[i-nums]+1)或dp[i] = max/min(dp[i], dp[i-num]+nums);
```

2、存在：是否存在…，满足…

```
// boolean
dp[i]=dp[i]||dp[i-num];
```

3、组合：求所有满足……的排列组合

```java
dp[i]+=dp[i-num];
```

```java
// 二维dp背包，自底向上
public int knapsack_bottom_up(int[] weights,int[] values, int capacity){
    int n = weights.length;
    // dp[i][w]: 对于第i个物品，总重不超过x的最大价值
    int[][] dp = new int[n][capicity];
    for(int i = 0; i < n; i++) dp[i][0] = 0;
    for(int j = 0; j < capacity; j++) dp[0][j] = 0;
    for(int x = 1; x < n; x++){
        for(int y = 1; y < capacity; y++){
            int w = weights[x];
            int v = values[x];
            if(y < w) dp[x][y] = dp[x-1][y]; // 第i个物品的重量w超出限制y
            else dp[x][y] = Math.max(dp[x-1][y], dp[x-1][y-w]+v);
        }
    }
    return dp[n-1][capacity-1];
}
```