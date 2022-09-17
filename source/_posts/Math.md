---
title: Leetcode / Math
mathjax: false
date: 2021-12-27 17:26:38
tags: leetcode
---

> ...

<!-- more -->

## 矩阵

- 54/剑指29  螺旋矩阵 / 59  螺旋矩阵-II
>【矩阵为空时需特判，定义四个边界左,上,右,下,
>顺时针打印的顺序：从左到右；从上到下；从右到左；从下到上
每个方向的打印需要做三件事：打印当前下标的值；边界收缩；判断打印是否结束】

- 48 旋转图像
> 【位置对应关系】对于矩阵中第 i 行的第 j 个元素，在旋转后，它出现在倒数第 i 列的第 j 个位置。因此对于矩阵中的元素 $matrix[row][col]$，在旋转后，它的新位置为 $matrix\_new[col][length - row - 1]$
> 【翻转代替旋转】水平翻转: `swap(matrix[row][col], matrix[length-row-1][col])` 对角线翻转: `swap(matrix[row][col], matrix[col][row])`

- 剑指04/ 240 二维数组中的查找/搜索二维矩阵 II（右上角，二叉搜索树）
- 74 搜索二维矩阵【右上角或左下角开始，二叉搜索树；二分，二维矩阵映射为一维】

## 数学

- 204 统计质数

```java
// 判断质数
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
// 两数最大公约数，gcd，greatest common divisor
// 辗转相除法

public int gcd(int a, int b) {
    if (a % b == 0) return b;
    return gcd(b, a % b);
}

public int gcd(int x, int y) {
    int remainder = x % y;
    while (remainder != 0) {
        x = y;
        y = remainder;
        remainder = x % y;
    }
    return y;
}

// 两数最小公倍数，lcm，least common multiple
// 最小公倍数=两数乘积/最大公约数
public int lcm(int a, int b) {
    return a * b / gcd(a, b);
}

// 多个数的gcd。lcm：先求出前两个数的gcd/lcm(a)，再用(a)与下一个数求出它们的gcd/lcm(b)，不断循环直到计算完所有数。

```

- 650 **只有两个键的键盘**【数学 - 因数分解】

```java
// 质因数分解
public void decompose(int n) {
    List  ans = new ArrayList();
    for (int i = 2; i * i <= n; i++) {
        while (n % i == 0) {
            ans.add(i);
            n /= i;
        }
    }
    if (n != 1) ans.add(n);
    return;
}
```
- 263 丑数
- 264/剑指49 丑数2【因数只含2、3、5的数】


- 991 坏了的计算器（只能 -1或 \* 2）
>贪心：当Y > X 且 Y 为奇数时，我们可以得到当前最优序列中倒数第二个数为 Y+1，递归求解到 Y+1 的最小操作次数。
当Y > X 且 Y为偶数时, 我们可以得到当前最优序列中倒数第二个数为Y/2，递归求解到Y/2的最小操作次数。
>逆向思维：从 Y 到 X，当 Y 大于 X 时，如果Y是奇数，执行 +1，否则执行 /2。之后，需要执行 X - Y 次加法操作得到 X。


- 470 用 Rand7() 实现 Rand10()

> 已知 rand_N() 可以等概率的生成[1, N]范围的随机数
那么：(rand_X() - 1) × Y + rand_Y() ==> 可以等概率的生成[1, X \* Y]范围的随机数
即实现了 rand_XY()

> 如何利用一个小范围随机数，得到一个大范围等概率随机数？
采用随机数的 k 进制，对于 randN，采用 N 进制，即：(randN - 1) \* N + randN 得到了一个 N \* N 范围的等概率随机数，如果还不够大，可以继续在 randN 或生成的 randN * N 上使用这个

> 如何利用一个小范围随机数，得到一个确定范围的等概率随机数？
先采用随机数的 k 进制，得到一个不小于确定范围的随机数 randK，然后**对超过确定范围数进行拒绝**。 

> 对于随机数 randN，只要 K 是 N 的约数（或者说 N 是 K 的整数倍），都可以通过 randN 一步得到 randK：randK = (randN % K) + 1;

- 7  整数反转