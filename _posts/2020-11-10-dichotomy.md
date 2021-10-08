---
layout: post
title: 【算法】一种通用的二分法写法
date: 2020-11-10
author: xiepl1997
tags: java 算法
---

二分法思路很简单，细节是魔鬼。  

二分法的细节问题有时候真的是让人脑壳痛，什么时候 right = len 而什么时候 right = len - 1，什么时候 mid + 1 什么时候 mid - 1，while 里面到底是 < 还是 <=，哎呀啊，想想就头痛噢。  

使用二分法主要是想在有序序列中找到：
* 一个确定的数
* 寻找左侧边界
* 寻找右侧边界

对于寻找左右边界的做法，主要是因为给定要找的数可能是多个，例如[1, 3, 3, 5, 7, 8]，当目标是找到 3 这个数的下标时，就可以对其进行左侧边界或者右侧边界的查找，左侧边界：1，右侧边界：2.  

现在对三种情况都用一种二分法的写法来完成！

### 寻找一个数（基本二分搜索）

这个场景是最简单的，当找到要找的那个数的时候，直接返回该数的索引就好，如果没有该数就返回-1.
```java
int binarySearch(int[] nums, int target) {
	int left = 0;
	int right = nums.length - 1;
	while (left <= right) {
		int mid = left + (right - left) / 2;
		if (nums[mid] == target) {
			// 直接返回
			return mid;
		} else if (nums[mid] < target) {
			left = mid + 1;
		} else if (nums[mid] > target) {
			right = mid - 1;
		}
	}
	// 直接返回
	return -1;
}
```

### 寻找左侧边界的二分搜索
```java
int binarySearch(int[] nums, int target) {
	int left = 0;
	int right = nums.length - 1;
	while (left <= right) {
		int mid = left + (right - left) / 2;
		if (nums[mid] == target) {
			// 别返回，锁定左侧边界
			right = mid - 1;
		} else if (nums[mid] < target) {
			left = mid + 1;
		} else if (nums[mid] > target) {
			right = mid - 1;
		}
	}
	// 最后要检查left越界的情况
	if (left >= nums.length || nums[left] != target)
		return -1;
	return left;
}
```

### 寻找右侧边界的二分搜索
```java
int binarySearch(int[] nums, int target) {
	int left = 0;
	int right = nums.length - 1;
	while (left <= right) {
		int mid = left + (right - left) / 2;
		if (nums[mid] == target) {
			// 别返回，锁定右侧边界
			left = mid + 1;
		} else if (nums[mid] < target) {
			left = mid + 1;
		} else if (nums[mid] > target) {
			right = mid - 1;
		}
	}
	// 最后要检查right越界的情况
	if (right < 0 || nums[right] != target)
		return -1;
	return right;
}
```