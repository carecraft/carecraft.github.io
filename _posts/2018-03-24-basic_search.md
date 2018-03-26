---
layout:     post
title:      "常见查找算法（一）"
subtitle:   "顺序表查找法、线性索引概述"
category : basictheory
date:       2018-03-24
author:     "Max"
header-img: "img/post-bg-algorithms.jpg"
catalog:    true
tags:
    - algorithms
---

对常见查找算法做一个归纳。

# 1. 顺序表查找 Sequential Search

顺序查找又称线性查找，是最基本的查找技术。它的基本过程如下：从表中第一个（或最后一个）记录开始，逐个进行记录的关键字和给定值比较，
若相等，则查找成功；若查找到最后一个（或第一个）记录，关键字和给定值比较都不相等，则表中没有所查的记录，查找不成功。

```
func SequentialSearch(nums []int, key int) int {
    for i, num := range nums {
        if num == key {
            return i
        }
    }
    return -1
}
```
显然，时间复杂度为 O(n)，长度很大时，查找效率会及其低下。

# 2. 有序表查找

## 2.1 折半查找 Binary Search

折半查找又称二分查找，它的前提是线性表中关键字有序，且线性表采用顺序存储。

```
func BinarySearch(nums []int, key int) int {
    low, high := 0, len(nums)-1
    for low <= high {
        mid := (low + high)/2
        if key < nums[mid] {
            high = mid-1
        } else if key > nums[mid] {
            low=mid+1
        } else {
            return mid
        }
    }
    return -1
}
```

时间复杂度为 O(logn)。

## 2.2 插值查找 Interpolation Search

若不折半，而是根据要查找的关键字 key 与查找表中最大最小记录的关键字比较来分割，就得到了另一种有序表查找算法，插值查找法。

仅需更改一行代码如下：
```
        mid := low + (high - low)*(key - nums[low])/(nums[high] - nums[low])
```

从时间复杂度来看依然是 O(logn)，但对于表长较大且关键字分布又比较均匀的查找表来说，插值查找算法的平均性能比折半查找的性能要好得多。

## 2.3 斐波那契查找 Fibonacci Search

若不折半，而是利用黄金分割原理实现，就得到了斐波那契查找算法。

需已知斐波那契数列 `F = []int{0,1,1,2,3,5,8,13,21,34...}`:
```
func FibonacciSearch(nums []int, key int) int {
    length := len(nums)
    low, high := 0, length-1
    k := 0
    for length > F[k] {
        k++
    }
    for low <= high {
        mid := low + F[k-1] - 1
        if key < nums[mid] {
            high = mid-1
            k = k-1
        } else if key > nums[mid] {
            low=mid+1
            k = k-2
        } else {
            if mid < length {
                return mid
            }
            return length-1
        }
    }
    return -1
}
```

# 3. 线性索引查找

要保证记录全部是按照当中的某个关键字有序，其时间代价是非常高昂的，因此很多数据通常都是按照先后顺序存储的，如服务器日志、微博贴吧的帖子及回复等。对于这样的数据，可以通过建立索引来快速查找。

## 3.1 稠密索引

指在线性索引中，将数据集中的每个记录对应一个索引项，索引项按照关键码有序排列。

## 3.2 分块索引

把数据集的记录分成若干块，满足快内无序、块间有序。每块对应一个索引项，包含最大关键码，块记录个数，和用于指向块首数据元素的指针三个数据项。

## 3.3 倒排索引

以次关键码和记录号表为通用索引结构的方法就是倒排索引（inverted index），其中记录号表存储具有相同次关键字的所有记录的记录号。

这种索引方法源于实际应用中需要根据属性的值来查找记录，优点就是查找记录非常快。




