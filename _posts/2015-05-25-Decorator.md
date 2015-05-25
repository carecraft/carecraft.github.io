---
layout: post
title: 装饰模式
category : design patterns
tagline:
tags : [design patterns, decorator]
---
{% include JB/setup %}

## 定义

《大话设计模式》：

>[DP]

，是型模式。

## UML类图

![]()

该模式中包含的角色及其职责如下：

*

  是抽象工厂模式的核心，包含所有的产品创建的抽象方法，与应用系统商业逻辑无关。

* 具体工厂（ConcreteFactory）

  直接在客户端的调用下创建具有特定实现的产品对象，有选择合适的产品对象的逻辑，而这个逻辑
  是与应用系统的商业逻辑紧密相关的。

* 抽象产品（AbstractProduct）

  抽象工厂模式所创建的对象的父类，或它们共同拥有的接口。有可能有多种不同的实现。

* 具体产品（Concrete Product）


## 特点



## 参考

1. []()

2. []()
