---
layout:     post
title:      "字符串匹配算法小结"
subtitle:   "分治、动态规划、贪心、回溯 归纳笔记"
category : basictheory
date:       2017-07-12
author:     "Max"
header-img: "img/post-bg-algorithms.jpg"
catalog:    true
tags:
    - algorithms
---

## 1. 概述

经常碰到类似这样的问题，要求找出文本 T[1..n]=abcabaabcabac 中模式 P[1..m]=abaa 的出现/有效位移。汇总网上常用算法做一些记录。

## 2. 朴素算法

最简单直接的想法，就是对 n+m-1 个可能的每一个有效位移 s 检查是否满足条件 P[1..m]=T[s+1..s+m]。

```golang
[]int func NaiveStringMatch(T,P []byte) {
    result := make([]int, 0)
    n, m := len(T), len(P)
    for s := 0; s <= n-m; s++ {
        if P[:m] == T[s:s+m] {
            result = append(result, s)
        }
    }
    return result
}
```
过程的运行时间为 O((n-m+1)m) 。这种匹配字符串匹配法的效率不高，其原因在于对于 s 的每个值，我们获得的关于文本的信息在考虑 s 的其它值时完全被忽略了。
