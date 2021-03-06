---
layout: post
title: 进程和线程
category : basictheory
author: Max
tags : [linux, process, thread]
---


<item>
<title>进程和线程</title>
<content:encoded>
<h2>1 差异</h2>

进程（process）和线程（thread）是操作系统的基本概念。书面的讲，进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动，是系统进行资源分配和调度的一个独立单位，具有资源独立、主从分明的特点；而线程是进程的一个实体，是CPU调度和分派的基本单位，是比进程更小的能独立运行的基本单位。线程自己基本上不拥有系统资源，只拥有一点在运行中必不可少的资源(如程序计数器，一组寄存器和栈)，但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源。

打个比方，<a href="http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html">进程就似一个工厂车间，线程是这个车间的工人</a>。

在操作系统层面，对进程和线程的支持是完全不同的。以Windows为代表的支持线程的操作系统，系统内部以线程为调度实体，进程相当于一堆数据结构；以Unix为代表的支持进程的操作系统，系统内部的调度实体是进程，线程基本上都在耍花招。因此，仅从多任务的调度效率上而言Windows占了很大优势。

<h2>2 Linux线程</h2>

Linux从2.6内核开始引入原生POSIX线程库(Native POSIX Thread Library)后，才真正成为一个可以傲视群雄的现代操作系统。NPTL是一种1:1的线程方案，一个线程会与内核的一个调度实体一一对应，线程的创建和回收都有内核负责。NPTL方案海引入了新的线程同步机制--快速用户空间互斥体(faster userspace mutex，缩写futex)。这个互斥锁完全再用户空间实现，规避了由用户空间向内核空间切换的代价，使锁的效率更高。

Linux内核本身的多任务调度实体被称为“内核线程”，与Windows还有很大差别。首先，Windows的调度实体就是线程，进程只是一堆数据结构；Linux将进程和线程做了同等对待，在内核一级没有差别，只是通过特殊的内存映射方法使得它们从用户的角度看来有了进城和线程的差别。其次，Windows至今也没有多进程的概念，创建进程的开销远大于创建线程的开销；Linux的开销则相差不大。最后，Windows与Linux的任务调度策略也不尽相同。Windows会随着线程越来越多而越来越慢，Linux却可遇持续运行很长时间。

<h2>3 进程的优势</h2>

进程是一个实体，每个进程都有它独立的地址空间。一般情况下要包括：代码区，数据区，堆栈。代码区的内容就是CPU要执行的代码；数据区存储变量和进程执行期间使用的动态分配的内存，也就是C程序员俗称的“堆”；堆栈区存储过程调用的指令和局部变量，也就是C程序员俗称的“栈”。

进程比线程出现的早，但依然有其不可忽略的优势。主要体现在以下四个方面：
1. 进程之间完全封闭，是非常典型的面向对象所追求的封装特性。而线程强调数据的共享，从本质上看就是对封装的一种破坏。
2. Linux系统提供了丰富可靠的进程间通信机制。而利用这种能力，进程之间完全可以像线程那样来交换数据或协调工作，完全不需要考虑复杂的同步机制，因为进程并不是通过共享数据来达到这些的。
3. 在Linux下进程的执行效率与线程的执行效率基本相当。
4. 进程特有主从分明的特点。父进程可以非常容易的了解到子进程出现问题退出了。而利用这种机制就可以采用一种非常简单的方案来缩短系统故障的修复时间。

<h3>1.4 系统调用fork()</h3>

当进程执行了fork()系统调用，子进程会复制父进程的所有内存页面，并将其载入操作系统为它所分配的那片独立内存中。不难想象，这个拷贝的动作对于CPU来说将会非常耗时。Linux早此处引入了一种叫做写时拷贝(Copy On Write，缩写COW)的机制，不干这种傻事。当fork发生时，子进程根本不会去拷贝父进程的内存页面，二是与父进程共享；当子进程或父进程需要修改一个内存页面时，Linux就将这个内存页面复制一份给修改者，然后再去修改。这样从用户的角度看，父子进程根本没有共享什么内存，符合进程资源独立的原则。这个花招就是COW。对Linux内核而言，线程和进程的差别就在于用不用COW。

采用了COW技术后，fork时，子进程还需要拷贝父进程的页面表。采用这种设计就是要模拟传统Unix系统fork时的效果。当然，这种拷贝的代价非常的小，对于CPU来说用不了几个时钟周期，类似nginx这类的高性能服务器系统就是受益于此。

</content:encoded>

</item>
