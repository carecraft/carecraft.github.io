---
layout: post
title: 装饰模式
category : designpattern
author: Max
tags : [design patterns, decorator]
---


## 定义

《大话designpattern》：

>动态地给一个对象添加一些额外的职责，就增加功能来说，装饰模式比生成子类更为灵活。[DP]


## UML类图

![decoratorpattern](/img/in-post/designpatterns/decorator.jpg)

该模式中包含的角色及其职责如下：

* (Component)

  是定义一个对象接口，可以给这些对象动态的添加职责。

* (ConcreteComponent)

  是定义一个具体的对象，也可以给这个对象添加一些职责。

* 装饰抽象类(Decorator)

  继承Component类，从外类来扩展 Component 类的功能。但对于Component来说，是无需知道 Decorator 的存在的。

  如果只有一个 ConcreteComponent 类而没有抽象的 Component 类，那么 Decorator 类可以是 ConcreteComponent 的一个子类。

* 具体装饰对象(ConcreteDecorator)

  给 Component 添加职责。

  如果只有一个 ConcreteDecorator 类，那么就没有必要建立一个单独的 Decorator 类，而可以把 Decorator 和 ConcreteDecorator 的职责合并成一个类。

## 特点

装饰模式是利用SetComponent来对对象进行包装的。这样每个装饰对象的实现就和如何使用这个对象分离
开了，每个装饰对象只关心自己的功能，不需要关心如何被添加到对象链当中。

总结下来，装饰模式有效的把类的核心职责和装饰功能区分开了。而且可以去除相关类中重复的装饰逻辑。

