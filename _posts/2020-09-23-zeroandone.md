---
layout: post
title: 一个背包问题
date: 2020-09-23
author: xiepl1997
tags: 敲敲敲
---

背包问题可以说是动态规划的经典问题了，围绕背包问题能够衍生出很多类似的问题。动态规划看起来不是那么好解决，它涉及到重复子问题和最优子结构，还有状态转移方程的寻找。充分理解了动态规划背后的逻辑，就会理解到其实它真正的原理就是穷举，但它是聪明地进行穷举。  

今天遇到一道题，类似思路类似于背包问题。  

## 问题描述

//在计算机界中，我们总是追求用有限的资源获取最大的收益。   
//  
// 现在，假设你分别支配着 m 个 0 和 n 个 1。另外，还有一个仅包含 0 和 1 字符串的数组。   
//  
// 你的任务是使用给定的 m 个 0 和 n 个 1 ，找到能拼出存在于数组中的字符串的最大数量。每个 0 和 1 至多被使用一次。  
//  
//  
//  
// 示例 1:  
//  
// 输入: strs = ["10", "0001", "111001", "1", "0"], m = 5, n = 3  
//输出: 4  
//解释: 总共 4 个字符串可以通过 5 个 0 和 3 个 1 拼出，即 "10","0001","1","0" 。  
//  
//  
// 示例 2:  
//  
// 输入: strs = ["10", "0", "1"], m = 1, n = 1  
//输出: 2  
//解释: 你可以拼出 "10"，但之后就没有剩余数字了。更好的选择是拼出 "0" 和 "1" 。  
//  
//  
//  
//  
// 提示：  
//  
//  
// 1 <= strs.length <= 600  
// 1 <= strs[i].length <= 100  
// strs[i] 仅由 '0' 和 '1' 组成  
// 1 <= m, n <= 100  
  
首先定义好dp，dp[i][j][k] 表示在前 i 个字符串的情况下有 j 个 0 和 k 个 1 时能够拼出数组中的字符串的最大数量。  
则状态转移方程为 **dp[i][j][k] = max(dp[i - 1][j][k], dp[i - 1][j - zero_num][k - one_num] + 1)**  

代码如下：  
```java
    public int findMaxForm(String[] strs, int m, int n) {
        if (strs == null || strs.length == 0)
            return 0;
        int[][][] dp = new int[strs.length + 1][m + 1][n + 1];
        for (int i = 1; i <= strs.length; i++) {
            int zero = zerocount(strs[i - 1]);
            int one = strs[i - 1].length() - zero;
            for (int j = 0; j <= m; j++) {
                for (int k = 0; k <= n; k++) {
                    if (j >= zero && k >= one)
                        dp[i][j][k] = Math.max(dp[i - 1][j][k], dp[i - 1][j - zero][k - one] + 1);
                    else
                        dp[i][j][k] = dp[i - 1][j][k];
                }
            }
        }
        return dp[strs.length][m][n];
    }
    public int zerocount(String str) {
        int len = str.length();
        int count = 0;
        for (int i = 0; i < len; i++) {
            if (str.charAt(i) == '0')
                count++;
        }
        return count;
    }
```