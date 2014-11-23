---
layout: post
title: Simple Factory
category : design patterns
tagline:
tags : [design patterns, simple factory]
---
{% include JB/setup %}

## 定义

简单工厂模式（Simple Factory Pattern）属于类的创新型模式，又叫静态工厂方法模式
（Static Factory Method Pattern）, 通过专门定义一个类来负责创建其他类的实例，
被创建的实例通常都具有共同的父类。

## UML类图

![简单工厂模式结构图](http://e.hiphotos.baidu.com/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26%3Bt%3Dgif/sign=697e92dba786c9171c0e5a6ba8541baa/08f790529822720ef04794d97bcb0a46f21fab0c.jpg)

该模式中包含的角色及其职责如下：

* 工厂（Creator）

  简单工厂模式的核心，它负责实现创建所有实例的内部逻辑。工厂类的创建产品类的方法可以
  被外界直接调用，创建所需的产品对象。

* 抽象产品（Product）

  简单工厂模式所创建的所有对象的父类，它负责描述所有实例所共有的公共接口。

* 具体产品（Concrete Product）

  是简单工厂模式的创建目标，所有创建的对象都是充当这个角色的某个具体类的实例。

## 特点

工厂类含有必要的判断逻辑，可以动态的决定在什么时候创建哪一个产品类的实例，使客户端可以免除
直接创建产品对象的责任，而仅仅"消费"产品。简单工厂模式通过这种做法实现了对责任的分割。

如果要对系统进行扩展，则需要增加实现产品接口的产品类。虽然无需对原有的产品类进行修改，但
却需要修改工厂类。所以简单工厂模式是不满足开放-封闭原则。

## 参考

1. [简单工厂模式-百度百科](http://baike.baidu.com/view/1227908.htm?fr=aladdin)

2. [简单工厂模式-vwinner的博客](http://blog.csdn.net/weiwenlongll/article/details/6918164)
