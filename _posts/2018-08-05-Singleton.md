---
layout:     post
title:      "Singleton"
subtitle:   "单例模式"
category :  designpattern
date:       2018-08-05
author:     "Max"
header-img: "img/post-bg-algorithms.jpg"
catalog:    true
tags:
    - design patterns
    - singleton
---


## 定义

> 单例模式，保证一个类仅有一个实例，并提供一个访问它的全局访问点。[DP]

单例模式是创建型模式之一。

## 实现

### 懒汉式

有一个简单实现如下：

```c++
class CSingleton
{
private:
    CSingleton(){}  // 构造函数私有
    static CSingleton *m_pInstance;
public:
    static CSingleton * GetInstance()
    {
        if (m_pInstance == NULL) //判断是否第一次调用
            m_pInstance = new CSingleton();
        return m_pInstance;
    }
};
```

这种在第一次被引用时才会将自己实例化的，被称之为懒汉式单例类。这种模式虽然可以保证类实例的唯一性，并且提供全局可访问点，但是却存在多线程访问的安全性问题。

因此有人提出了 Double-Check Locking（双重锁定）方法解决此问题：

```c++
class CSingleton
{
private:
    CSingleton(){}
    static CSingleton *m_pInstance;
    static std::mutex m;
public:
    static CSingleton * GetInstance()
    {
        if (m_pInstance == NULL)      //判断是否第一次调用
        {
            std::unique_lock lock(m); //析构函数自动 unlock
            if (m_pInstance == NULL)  //第二次判断
            {
                m_pInstance = new CSingleton();
            }
        }
        return m_pInstance;
    }
};
```

如果内存访问严格按照语句先后顺序进行，那么以上代码堪称完美解决了所有问题。但是，在某些内存模型中，或者是由于编译器的优化以及运行时优化等原因，可能使得 m_pInstance 虽然已经不是 NULL 但是其所指对象还没有完成构造。这种情况下，另一个线程如果调用 GetInstance() 就有可能使用到一个不完全初始化的对象，从而引发错误。

### 饿汉式

还有一种构建方式，在自己被加载时就将自己实例化。这种方式不需要开发人员显式的编写线程安全代码，即可解决多线程环境下的不安全问题，但却会提前占用系统资源。

```c++
class CSingleton
{
protected:
    CSingleton(){}
private:
    static CSingleton* p;
public:
    static CSingleton* GetInstance();
};
CSingleton* CSingleton::p = new CSingleton;
CSingleton* CSingleton::GetInstance()
{
    return p;
}
```

### c++11

C++11 提供了更加方便的组件用以实现单例模式：

```c++
static unique_ptr<CSingleton> CSingleton::instance;
static std::once_flag CSingleton::create;
CSingleton& CSingleton::GetInstance()
{
    std::call_once(create, [=]{ instance = make_unique<CSingleton>(); });
    return instance;
}
```

从 C++11 开始，在一个线程开始 local static 对象的初始化后并初始化完成前，其他线程执行到这个local static对象的初始化语句就会等待，直到该local static 对象初始化完成。因此，我们将会得到一份最简洁也是效率最高的单例模式的 C++11 实现：

```c++
CSingleton& CSingleton::GetInstance()
{
    static CSingleton s_Instance;
    return s_Instance;
}
```

### 重用

上述代码 CSingleton 实现是无法重用的。一般来说，重用方法有三种：组合、派生以及模板。对 CSingleton 的重用仅仅是对 GetInstance() 函数的重用，因此通过从 CSingleton 派生以继承该函数的实现是一个很好的选择。而 GetInstance() 函数如果能根据实际类型更改返回类型则更好了。因此奇异递归模板（CRTP，The Curiously Recurring Template Pattern）模式是一个非常好的选择。

```c++
template <typename T>
class CSingleton
{
public:
    static T& GetInstance()
    {
        static T s_Instance;
        return s_Instance;
    }

protected:
    Singleton(void) {}
    ~Singleton(void) {}

private:
    Singleton(const Singleton& rhs) {}
    Singleton& operator = (const Singleton& rhs) {}
};
```

## 参考

1. [C++程序员们，快来写最简洁的单例模式吧](http://www.cnblogs.com/zxh1210603696/p/4157294.html)
2. [C++单例模式](https://blog.csdn.net/qq_35280514/article/details/70211845)
3. [面试中的Singleton](http://www.cnblogs.com/loveis715/archive/2012/07/18/2598409.html)
