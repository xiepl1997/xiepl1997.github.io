---
layout: post
title: 回文子串的数量（manacher）
date: 2020-08-06
author: xiepl1997
tags: 敲敲敲
---

之前写过一篇关于马拉车算法求最长回文子串的博客，讲到马拉车算法能在O(N)的时间复杂度下求得最长回文子串，今天遇到一个问题：**求字符串中的回文子串的数量**，这同样可以用马拉车算法来计算。特此记录一下。  

>问题描述

输入: "abc"
输出: 3
解释: 三个回文子串: "a", "b", "c".

输入: "aaa"
输出: 6
说明: 6个回文子串: "a", "a", "a", "aa", "aa", "aaa".

>代码

对于马拉车算法的原理可以参考 <https://xiepl1997.github.io/2019/09/07/manacher.html>,这里就不多赘述。那为什么马拉车能够用在这个问题上呢，因为它维护了一个数组Len，这个数组保存的是以每个字符为中心的最长回文串的半径，这就能利用上了，例如“aba”，对于“b”来说以它为中心的回文串有两个，“b”和“aba”，在数组Len中能计算得到它的回文半径，也就是说能得到2，这个2既是以“b”为中心的最长回文串的半径，同时也是以“b”为中心的回文串的个数。  

代码如下：
```java
class Solution {
    public int countSubstrings(String s) {
        //马拉车算法
        if(s == null || s.length() == 0)
            return 0;
        int res = 0;
        String temp = "@#";
        for(int i = 0; i < s.length(); i++){
            temp += s.charAt(i);
            temp += "#";
        }
        temp += "?";
        int[] Len = new int[temp.length()];
        int mid = 0;
        int right = 0;
        for(int i = 1; i < Len.length - 1; i++){
            if(i < right)
                Len[i] = Math.min(Len[mid * 2 - i], right - i);
            else
                Len[i] = 1;
            while(temp.charAt(i + Len[i]) == temp.charAt(i - Len[i]))
                Len[i]++;
            if(i + Len[i] > right){
                right = i + Len[i];
                mid = i;
            }
        }
        for(int v : Len)
            res += v / 2;
        return res;
    }
}
```