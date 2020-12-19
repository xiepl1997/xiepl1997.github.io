---
layout: post
title: 剑指offer.41 数据流中的中位数
date: 2020-12-19
author: xiepl1997
tags: 敲敲敲
---

这道题涉及到对堆这个数据结构的使用，落实到代码上实际使用的是优先队列（优先队列底层可以通过堆来实现）。  

> 题目描述

如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数
值排序之后中间两个数的平均值。  

例如，  

[2,3,4] 的中位数是 3  

[2,3] 的中位数是 (2 + 3) / 2 = 2.5  

设计一个支持以下两种操作的数据结构：  
void addNum(int num) - 从数据流中添加一个整数到数据结构中。  
double findMedian() - 返回目前所有元素的中位数。  

示例：  
输入：  
["MedianFinder","addNum","addNum","findMedian","addNum","findMedian"]
[[],[1],[2],[],[3],[]]  
输出：[null,null,null,1.50000,null,2.00000]  

> 解析

这道题需要设计一个数据结构来完成快速的中位数的获取。  
当然了，一个最基础的想法就是每次插入新数据之后都对数据进行排序，然后获取中位数。但这样的代价太大了，明显不可取。  

为此，使用两个堆来存储不断流入的数据流，一个大根堆和一个小根堆。  

其中，大根堆中的数据要始终小于小根堆中的数据，大根堆的堆顶数据是该堆中数据的最大值，小根堆的堆顶是该堆中数据的最小值。当数据总数为奇数时，小根堆数据数量比大根堆数量多1，当总数为偶数时，两个堆数据量一致。  

当要获取数据中位数时：
* 如果数据总数为奇数，则取小根堆堆顶。
* 如果数据总数为偶数，则取（大根堆堆顶 + 小根堆堆顶）/ 2

> code
```java
class MedianFinder {
    // 使用大根堆和小根堆
    private PriorityQueue<Integer> min;
    private PriorityQueue<Integer> max;
    private int count;

    /** initialize your data structure here. */
    public MedianFinder() {
        min = new PriorityQueue<>();
        max = new PriorityQueue<>((Integer a, Integer b) -> {return b - a;});
        count = 0;
    }
    
    public void addNum(int num) {
        count++;
        // 如果是奇数个，则小根堆多一个，偶数个则一样多
        if (count % 2 == 1) {
            max.offer(num);
            min.offer(max.poll());
        } else {
            min.offer(num);
            max.offer(min.poll());
        }
    }
    
    public double findMedian() {
        if (count % 2 == 0)
            return (min.peek() + max.peek()) / 2.0;
        return (double)min.peek();
    }
}
```