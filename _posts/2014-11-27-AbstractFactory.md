---
layout: post
title: 抽象工厂模式
category : designpattern
author: Max
tags : [design patterns, abstract factory]
---


## 定义

《大话designpattern》：

>抽象工厂模式，提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。[DP]

与简单工厂模式和工厂方法模式相同，是创建型模式。

## UML类图

![抽象工厂模式结构图](http://images.cnblogs.com/cnblogs_com/zhenyulu/Pic46.gif)

该模式中包含的角色及其职责如下：

* 抽象工厂接口（AbstractFactory）

  是抽象工厂模式的核心，包含所有的产品创建的抽象方法，与应用系统商业逻辑无关。

* 具体工厂（ConcreteFactory）

  直接在客户端的调用下创建具有特定实现的产品对象，有选择合适的产品对象的逻辑，而这个逻辑
  是与应用系统的商业逻辑紧密相关的。

* 抽象产品（AbstractProduct）

  抽象工厂模式所创建的对象的父类，或它们共同拥有的接口。有可能有多种不同的实现。

* 具体产品（Concrete Product）

  具体的产品，对抽象产品的具体分类的实现。抽象工厂模式所创建的任何产品对象都是某一个具体
  产品类的实例。这是客户端最终需要的东西，其内部一定充满了应用系统的商业逻辑。

## 特点

抽象工厂模式有良好的封装性。每个产品的实现类不是高层模块要关心的，要关心的是接口，是抽象。
高层模块不关心对象是如何创建出来的，只要知道工厂类是谁，就能创建出一个需要的对象。

抽象工厂模式产品族内的约束为非公开状态，调用工厂类的高层模块无需知道。

但是，抽象工厂模式严重违反了开闭原则，产品族扩展非常困难。以UML类图所示结构为例，如果要增
加一个产品C，抽象类AbstractCreator要增加一个方法createProductC()，然后，所有与该抽象类
有关的代码都要修改，如本例中的两个实现类。

## 参考

1. [抽象工厂模式-技术之大，在乎你我心中](http://www.cnblogs.com/cbf4life/archive/2009/12/23/1630612.html)

2. [抽象工厂模式-随风的专栏](http://blog.csdn.net/ipqxiang/article/details/1955677)
