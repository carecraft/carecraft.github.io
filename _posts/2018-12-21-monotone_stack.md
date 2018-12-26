---
layout:     post
title:      "单调栈和单调队列"
subtitle:   "monotone stack/queue"
category : basictheory
date:       2018-12-21
author:     "Max"
header-img: "img/post-bg-algorithms.jpg"
catalog:    true
tags:
    - algorithms
---

## 1. 单调栈

### 1.1 概述

单调栈是指一个栈内部的元素具有严格单调性的一种数据结构，分为单调递增栈和单调递减栈，具有如下性质：
1. 满足栈的后进先出特性
2. 从栈顶到栈底的元素具有严格的单调性

对于单调递增栈，若当前进栈元素为 e，从栈顶开始遍历元素，把小于（或等于） e 的元素弹出栈，直到遇到一个大于 e 的元素或者栈为空为止，然后再把 e 压入栈中。类似的，对于单调递减栈，则每次弹出的是大于（或等于） e 的元素。

### 1.2 应用

利用单调栈，我们可以轻易找到第一个更大或更小的元素，解决一些看似复杂的问题。

以[柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)问题为例。

首先假定已经找到的这个最大矩形，其左边界下标为 L，右边界下标为 R，矩形高度为 

$$
H=\min \left\{ heights[i] | L \leq i \leq R \right\}
$$

若柱 L-1 存在且大于或等于高度 H，则该矩形可向左扩展，与该矩形面积最大的假定不符。可见，左边界 L 是从右往左第一个高度小于 H 的柱的右边的柱。同理，右边界 R 是从左往右第一个高度小于 H 的柱的左边的柱。

如此，利用单调栈，对于每一个确定高度 $heights[i]$，我们都可以唯一确定它的最大矩形，从中选出最大值即可。

```
func largestRectangleArea(heights []int) int {
    l := len(heights)
    if l == 0 {
        return 0
    }
    left, right := make([]int, l), make([]int, l)
    left[0], right[l-1] = 0, l-1 
    h := []int{0}
    for i:=1; i<l; i++ {
        for len(h) > 0 && heights[h[len(h)-1]] >= heights[i] {
            h = h[:len(h)-1]
        }
        if len(h) == 0 {
            left[i] = 0
        } else {
            left[i] = h[len(h)-1]+1
        }
        h = append(h, i)
    }
    
    h = []int{l-1}
    for i:=l-2; i>=0; i-- {
        for len(h) > 0 && heights[h[len(h)-1]] >= heights[i] {
            h = h[:len(h)-1]
        }
        if len(h) == 0 {
            right[i] = l-1
        } else {
            right[i] = h[len(h)-1]-1
        }
        h = append(h, i)
    }
    
    area := 0
    for i:=0; i<l; i++ {
        if temp := (right[i] - left[i] + 1)*heights[i]; temp > area {
            area = temp
        }
    }
    return area
}
```

## 2. 单调队列

### 2.1 概述

单调队列是指一个队列内部的元素具有严格单调性的一种数据结构，分为单调递增队列和单调递减队列。单调队列满足两个性质：
1. 满足队列先入先出的特性
2. 从队首到队尾的元素具有严格的单调性

对于单调递增队列，若当前进队列元素为 e，从队尾开始遍历元素，把大于（或等于） e 的元素移出队列，直到遇到一个小于 e 的元素或者队列为空为止，然后再把 e 入队。类似的，对于单调递减队列，则每次移出的是小于（或等于） e 的元素。

### 2.2 应用

单调队列可以维护一个滑动窗口的最大值或最小值。

以[滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum/)为例，展示单调队列的用法。

```
func maxSlidingWindow(nums []int, k int) []int {
    if len(nums) == 0 {
        return nil
    }
    window := list.New()
    var ret []int
    for i, v := range nums {
        if i >= k && window.Front().Value.(int) <= i-k {
            window.Remove(window.Front())
        }
        for window.Len() > 0 && nums[window.Back().Value.(int)] <= v {
            window.Remove(window.Back())
        }
        window.PushBack(i)
        if i >= k-1 {
            ret = append(ret, nums[window.Front().Value.(int)])
        }
    }
    return ret
}
```