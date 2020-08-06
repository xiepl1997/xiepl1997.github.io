---
layout: post
title: 下一个排列
date: 2019-09-11
author: xiepl1997
cover: 'assets/img/20190911.png'
tags: 敲敲敲
---

### 题目描述
实现获取下一个排列的函数，算法需要将给定的数字序列重新排列成字典序中下一个更大的排列，如果不存在则将数字重新排列成最小的排列（即升序排列），必须原地修改。  
例如  
`1,2,3 --> 1,3,2`  
`1,5,8,6,7,3,2,1 --> 1,5,8,7,1,2,3,6`  

为了得到下一个排列，首先找到从右往左的第一对nums[i-1] < nums[i]的一对数，然后在nums[i-1]的右边的序列中按照从右往左的顺序找到第一个大于nums[i-1]的元素nums[j]，题目中要求的下一个更大的排列，所以将nums[i-1]与nums[j]交换，由于nums[i-1]是通过从右往左找到第一对nums[i-1] < nums[i]来确定的，所以在nums[i-1]的右边的序列是降序的。在交换了nums[i-1]和nums[i]后，还需要对nums[i-1]右边的序列进行翻转，使其变成升序子序列。代码如下：
```java
/**
	 * 下一个排列
	 * 从右往左找到第一对nums[i]>nums[i-1]，然后再从最右边往前找，找到第一个大于nums[i-1]的元素nums[j]，交换nums[i-1]和nums[j]，
	 * 最后再将nums[i-1]之后的元素反转。
	 * @param nums
	 */
	public void nextPermutation(int[] nums) {
		int i = nums.length-1;
		int flag = 0; //用于标记序列是否是全降序，0是，1不是
		for(; i > 0; i--){
			if(nums[i] > nums[i-1]){
				i--;
				flag = 1;
				break;
			}
		}
		if(flag == 1) {
			int j = nums.length - 1;
			for (; j >= 0; j--) {
				if (nums[j] > nums[i]) {
					int temp = nums[j];
					nums[j] = nums[i];
					nums[i] = temp;
					break;
				}
			}
			for (int k = i + 1, l = nums.length - 1; k < nums.length; k++, l--) {
				if (k > (i + nums.length) / 2) {
					break;
				}
				int temp = nums[k];
				nums[k] = nums[l];
				nums[l] = temp;
			}
		}
		else{
			for(int k = 0,l = nums.length-1; k < nums.length; k++, l--){
				if(k > (nums.length-1)/2){
					break;
				}
				int temp = nums[k];
				nums[k] = nums[l];
				nums[l] = temp;
			}
		}
	}
```
时间复杂度为O(N)