---
layout: post
title: TemplateMethod
category : design patterns
tagline:
tags : [design patterns, template method]
---
{% include JB/setup %}

## 定义

《大话设计模式》：

>**模版方法**模式，定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模版方法使得子类
可以不改变一个算法的结构即可重定义该算法的某些特定步骤。[DP]

## UML类图

![模版方法模式结构图](http://p.blog.csdn.net/images/p_blog_csdn_net/lenotang/EntryImages/20080911/8c7d65ea9a3a41b7a2aa58eea79346d3.jpg)

## 特点

模板方法模式属于行为模式的一种。通过把不变行为搬移到父类，去除子类中的重复代码来体现其优势。

各个模式之间都有联系，模板方法也不例外。模板中的那些虚方法实际上使用了工厂方法设计模式，
将父类的执行逻辑延迟到子类；
有的时候算法骨架存在不止一种，则可以使用策略模式。
