---
title: Two sum
urlname: LeetCode_TwoSum
mathjax: true
tags:
  - LeetCode
  - Two sum
categories: LeetCode
description: 闲置许久，终于开始写Leetcode啦，兴奋o(*￣▽￣*)ブ，希望能借助Leetcode学习一些算法相关的知识~~
abbrlink: 22353
date: 2019-03-30 23:43:00
---

闲置许久，终于开始写Leetcode啦，兴奋o(*￣▽￣*)ブ，希望能借助Leetcode学习一些算法相关的知识~~



**LeetCode第一题**：

#### question

难度：Easy

​						        Two sum
Given an array of integers, return indices of the two numbers such that they add up to a specific target.
You may assume that each input would have exactly one solution, and you may not use the same element twice.
Example:
Given nums = [2, 7, 11, 15], target = 9,
Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].

译文：
​                                                            两数之和
给定一个整数数组和一个目标值，找出数组中和为目标值的 两个 数。
你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。
示例:
给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]



#### solution

将时间复杂度缩短至O(n),只能以空间换时间，这里利用HashMap（查找时间复杂度为O(1)）

```java
public Vector<Integer> twoSum(Vector<Integer> nums, int target) {
    Vector<Integer> resultVector = new Vector<Integer>();
	 Map<Integer,Integer> map = new HashMap<Integer, Integer>();
	 int length = nums.size();
	 
	 for(int i = 0; i<length; i++) {
		 int complementation = target - nums.get(i);
		 if(map.containsKey(complementation)) {
			 resultVector.add(map.get(complementation));
			 resultVector.add(i);
			 return resultVector;
		 }
		 map.put(nums.get(i),i);
	 }
	 return resultVector;
   }
```

在网上还看到了一份大佬的优质解法，但一时找不到网址了，暂时无法分享，但后续会更新呦

附：LeetCode代码仓库链接

<a  href="https://github.com/CN-ZhangYue/LeetCode">https://github.com/CN-ZhangYue/LeetCode</a>





