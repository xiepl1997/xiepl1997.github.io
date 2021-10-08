---
layout: post
title: 【算法】不用乘、除、取余操作实现除法
date: 2021-09-27
author: xiepl1997
tags: 算法
---

来自剑指offerⅡ的一道题，如何在不使用乘号、除号、取余符号的情况下实现整数除法操作。  

首先想到的是被除数循环减除数，减到不能再减了，减的次数就是最终答案。这也是除法的本质，但是能不能快一点不用一个一个减呢？答案是可以的，我们可以通过位移被除数的操作来判断可减去除数的数量，代码如下。
```java
class Solution {
	// a为被除数，b为除数
    public int divide(int a, int b) {
        if (a == 0)
            return 0;
        if (a == b)
            return 1;
        if (a == Integer.MIN_VALUE && b == -1)
            return Integer.MAX_VALUE;
        boolean f = (a ^ b) < 0;
        long a1 = Math.abs((long)a);
        long b1 = Math.abs((long)b);
        int res = 0;
        for (int i = 31; i >= 0; i--) {
            if ((a1 >> i) >= b1) {
                a1 -= (b1 << i);
                res += (1 << i);
            }
        }
        return f ? -res : res;
    }
}
```