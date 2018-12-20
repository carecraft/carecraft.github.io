---
layout:     post
title:      "Composite"
subtitle:   "组合模式"
category :  designpattern
date:       2018-09-14
author:     "Max"
header-img: "img/post-bg-algorithms.jpg"
catalog:    true
tags:
    - design patterns
    - composite
---


## 定义

> 组合模式，将对象组合成树形结构以表示“部分-整体”的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。[DP]

组合模式是结构型模式之一。

## UML类图

![compositepattern](/img/in-post/designpatterns/composite.jpg)

* **Component**
    
    组合中的对象声明接口。在适当情况下，实现所有类共有接口的默认行为。声明一个接口用于访问和管理 Component 的子部件。

    通常用 Add 和 Remove 方法来提供增加或移除树叶或树枝的功能。

* **Leaf**
    
    在组合中表示叶节点对象，叶节点没有子节点。

    虽然实现 Add 和 Remove 方法没有意义，但可以消除叶节点和枝节点对象在抽象层次的区别，使它们具备完全一致的接口。

* **Composite**
    
    定义有枝节点行为，用来存储子部件，在 Component 接口中实现与子部件有关的操作，比如 Add 和 Remove 方法。

## 应用场景


1. 需求中是体现部分与整体层次的结构时
2. 希望用户可以忽略组合对象与单个对象的不同，统一地使用组合结构中的所有对象时

## 参考

1. 程杰《大话设计模式》


