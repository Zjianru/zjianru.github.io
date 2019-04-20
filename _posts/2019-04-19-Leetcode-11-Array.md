---
layout:     post
title:      Leetcode 每日一题
subtitle:   Container With Most Water
date:       2019-04-19
author:     Alessio
header-img: img/PostBack_03.jpg
catalog: true
tags:
    - Leetcode
---
## 问题描述
>
> Given n non-negative integers a1, a2, ..., an , where each represents a point at coordinate (i, ai). 
>
> n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). 
>
>Find two lines, which together with x-axis forms a container, such that the container contains the most water.
>
> Note: You may not slant the container and n is at least 2.

示意图如下

![Container With Most Water](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/07/17/question_11.jpg)

Leetcode 官方也给出了样例：

> `Input: [1,8,6,2,5,4,8,3,7]`
>
> `Output: 49`

## 思考过程

我们首先来完成暴力解法

这道题实际是要我们求出最大面积，我们大可以一步步遍历下去：

1. 设定一个游标变量指向数组首坐标 记作当前位置 `current`

2. 选取当前位置的下一个坐标 记为 `next`

3. `current` 和 `next` 选取最小值作为基准 与两值的正坐标差做乘积  存入结果集中

4. 循环遍历，取完所有可能情况

5. 在结果集中寻找最大值

在这里给出暴力解法代码
```java
	public int maxAreaInBruteForce(int[]height){
		Set<Integer> resultSet = new HashSet<Integer>();
		for (int i = 0; i < height.length; i++)
		{
			for (int j = i + 1; j < height.length; j++)
			{
				if (height[i] < height[j]){
					int h = height[i] > height[j] ? height[i] : height[j];
					int info = h * (j - i);
					resultSet.add(info);
				}
			}
		}
			return findMax(resultSet);
	}

	private int findMax(Set<Integer> resultSet){
		int result = 0;
		for (int i : resultSet){
			if (result < i) {
				result = i;
			}
		}
		return result;
	}
}
```
## 优化方案
这样的时间复杂度肯定是不合格的  我们来改进暴力解法

1. 我们使用了两个变量作为指针，从一端遍历到另一端，那么可不可以设置两个端点，同时从两边向中间遍历呢？

2. 在暴力解法里，我们使用 java 中的 set 数据结构来完成结果数据的筛选，不在“第一时间”处理结果  并且在对 set 里的结果进行处理时，使用了一次 for 循环  这个开销能不能节省呢?

3. 能不能在节省时间开销的同时  也减少空间的占用（减少无关对象的创建）？


基于上述思考
我完成了优化解法，代码如下：
```java
public int maxArea(int[] height) {
    int left = 0, right = height.length - 1;
    int maxArea = 0;
    while (left < right && left >= 0 && right <= height.length - 1) {
        maxArea = Math.max(maxArea, Math.min(height[left], height[right]) * (right - left));
        if (height[left] > height[right]) {
            right--;
        } else {
            left++;
        }
    }
    return maxArea;
}
```