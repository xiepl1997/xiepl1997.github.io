---
layout: post
title: 验证二叉搜索树
date: 2020-03-05
author: xiepl1997
tags: 敲敲敲
---

给定一个二叉树，判断其是否是二叉搜索树。  

一个二叉搜索树的特点是：  
* 节点的左子树只包含**小于**当前节点的数。
* 节点的右子树只包含**大于**当前节点的数。

示例：
```
输入：
	2
   / \
  1   3

输出：true


输入：
	5
   / \
  1   4
     / \
    3   6

输出：false
```

二叉搜索树有一个特点，就是中序遍历序列为递增序列，例如示例1的中序遍历为1-2-3；而示例2的中序遍历为1-5-3-4-6，不是递增，则其不是二叉搜索树。  

代码如下：
```java
//Definition for a binary tree node.
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}

class Solution {
    public boolean isValidBST(TreeNode root) {
        if(root == null)
            return true;
        Stack<TreeNode> stack = new Stack<>();
        TreeNode t = root;
        double temp = -Double.MAX_VALUE; //临时值，保留上一个值
        while(t != null || !stack.isEmpty()){
            if(t != null){
                stack.push(t);
                t = t.left;
            }
            else{
                t = stack.pop();
                if(temp < t.val)
                    temp = (double)t.val;
                else
                    return false;
                t = t.right;
            }
        }
        return true;
    }
}
```

算法时间复杂度O(N)  
空间复杂度O(N)