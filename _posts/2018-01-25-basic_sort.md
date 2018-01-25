---
layout:     post
title:      "常见排序算法"
subtitle:   "冒泡、选择、插入、希尔、堆、归并、快速排序"
category : basictheory
date:       2018-01-25
author:     "Max"
header-img: "img/post-bg-algorithms.jpg"
catalog:    true
tags:
    - algorithms
---

对常见排序算法做一个归纳。

# 1. 冒泡排序（Bubble Sort)

冒泡排序是一种交换排序，它的基本思想是两两比较相邻记录的关键字，如果反序就交换，知道没有反序的记录为止。

给定一个乱序数组 nums[n]，简单实现非递增如下：
```go
func BubbleSort(nums []int) {
    length := len(nums)
    swapped := true
    for i := 0; i < length && swapped; i++ {
        swapped = false
        for j := length - 1; j > i; j-- {
            if nums[j] > nums[j-1] {
                swapped = true
                nums[j], nums[j-1] = nums[j-1], nums[j]
            }
        }
    }
}
```
# 2. 简单选择排序（Simple Selection Sort）

简单选择排序就是通过 n-i 次关键字的比较， 从 n-i+1 个记录中选出关键字最小的记录，并和第 i（1<=i<=n）个记录交换。
```go
func SelectSort(nums []int) {
    length := len(nums)
    for i := 0; i < length; i++ {
        max := i
        for j := i; j < length; j++ {
            if nums[j] > nums[max] {
                max = j
            }
        }
        if max != i {
            nums[i], nums[max] = nums[max], nums[i]
        }
    }
}
```

# 3. 直接插入排序（Straight Insertion Sort）

直接插入排序的基本操作是将一个记录插入到已经排好序的有序表中，从而得到一个新的、记录数增 1 的有序表。

```go
func InsertSort(nums []int) {
    length := len(nums)
    for i := 1; i < length; i++ {
        if nums[i] > nums[i-1] {
            temp := nums[i]
            j := i-1
            for nums[j] < temp {
                nums[j+1] = nums[j]
                j--
            }
            nums[j] = temp
        }
    }
}
```

# 4. 希尔排序（Shell Sort）

从时间上来讲，希尔排序是突破 O(n^2) 复杂度的第一批算法之一，之后才相继出现更加高效的算法；从原理上来讲，希尔排序是将相距某个增量的记录组成子序列分别进行直接插入排序的过程，使整个记录序列基本有序。当增量为 1 的直接插入排序后，整个序列有序。

```go
func ShellSort(nums []int) {
    length := len(nums)
    increment := length
    for increment > 1 {
        increment = increment/3 +1;
        for i := increment; i < length; i++ {
            if nums[i] > nums[i-increment] {
                temp := nums[i]
                j := i - increment
                for j >= 0 && nums[j] < temp {
                    nums[j+increment] = nums[j]
                    j -= increment
                }
                nums[j+increment] = temp
            }
        }
    }
}
```
# 5. 堆排序（Heap Sort）

堆是具有下列性质的完全二叉树：每个节点的值都大于或等于其左右孩子节点的值，称为大顶堆；或者每个节点的值都小于或等于其左右孩子节点的值，称为小顶堆。

堆排序就是利用堆进行排序的方法——将带排序的序列构造成一个小顶堆，此时整个序列的最小值就是堆顶的根节点，将其与堆数组的末尾元素交换，将剩余的 n-1 个序列重新构造成堆，然后反复执行。

```go
func HeapSort(nums []int) {
    length := len(nums)
    for i := length/2-1; i >= 0; i-- {
        HeapAdjust(nums, i, length-1)
    }
    for i := length-1; i > 0; i-- {
        nums[0], nums[i] = nums[i], nums[0]
        HeapAdjust(nums, 0, i-1)
    }
}

func HeapAdjust(nums []int, s int, e int) {
    temp := nums[s]
    for j := 2*s+1; j <= e; j = 2*j+1 {
        if j < e && nums[j] > nums[j+1] {
            j++
        }
        if temp <= nums[j] {
          break
        }
        nums[s] = nums[j]
        s = j
    }
    nums[s] = temp 
}
```

# 6. 归并排序（Mergeing Sort）

假设初始序列含有 n 个记录，则可以看成是 n 个有序的子序列，每个子序列的长度为 1， 然后两两归并，得到 ⌈n/2⌉ 个长度为 2 或 1 的有序子序列，再两两归并，如此重复，直到得到一个长度为 n 的有序序列为止。这种排序方法称为 2 路归并排序。

```go
func MergeSort(nums []int) {
    length := len(nums)
    TR := make([]int, length)
    for k := 1; k < length; {
        MergePass(nums, TR, k, length)
        k *= 2
        MergePass(TR, nums, k, length)
        k *= 2
    }
}

/* 将 SR[] 中相邻长度为 s 的子序列两两归并到 TR[] */
func MergePass(SR []int, TR []int, s int, n int) {
    
}
```