---
title: Leetcode / Math
mathjax: false
date: 2021-12-27 17:26:38
tags: leetcode
---

> ...

<!-- more -->

## 矩阵

> - 54/剑指29  螺旋矩阵
>【矩阵为空时需特判，定义四个边界左,上,右,下,
>顺时针打印的顺序：从左到右；从上到下；从右到左；从下到上
每个方向的打印需要做三件事：打印当前下标的值；边界收缩；判断打印是否结束】
> - 59  螺旋矩阵-II
> - 剑指04/ 240 二维数组中的查找/搜索二维矩阵 II（右上角，二叉搜索树）
> - 74. 搜索二维矩阵
【右上角或左下角开始，二叉搜索树；二分，二维矩阵映射为一维】

## 数学

- 204 统计质数

```java
public boolean isPrimes(int n){
    for(int i = 2; i * i <= n; i++){
        if(n % i == 0) return false;
    }
    return true;
}
```

- 367 有效的完全平方数 / 69 Sqrt(x) 【二分】
- 365 水壶问题【找最大公约数】

```java
// 辗转相除法
public int gcd(int x, int y) {
    int remainder = x % y;
    while (remainder != 0) {
        x = y;
        y = remainder;
        remainder = x % y;
    }
    return y;
}
```

- 650 **只有两个键的键盘**【数学 - 因数分解】

```java
class Solution {
    public int minSteps(int n) {
        int ans = 0;
        for (int i = 2; i * i <= n; i++) {
            while (n % i == 0) {
                ans += i;
                n /= i;
            }
        }
        if (n != 1) ans += n;
        return ans;
    }
}
```
- 263 丑数
- 264/剑指49 丑数【因数只含2、3、5的数】