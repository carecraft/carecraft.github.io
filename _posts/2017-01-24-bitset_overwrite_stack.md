---
layout:     post
title:      "C++内存分配之堆栈"
subtitle:   "记一次栈越界错误"
category : basictheory
date:       2017-01-24
author:     "Max"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - C++
    - memory
    - Linux
---

## 问题描述


使用[bitset](http://www.cplusplus.com/reference/bitset/bitset/bitset/)时碰到一个问题，进程在bitset构造那里core dumped。源码如下：

```
#include <bitset>

int main ()
{
    std::bitset<0xffffffff> foo;
    foo.reset();
    return 0;
}
```

使用dmesg查询，得到信息如下：

```
a.out[21521]: segfault at 7ffdb1d3f518 ip 00000000004009ae sp 00007ffdb1d3f520 error 6 in a.out[400000+1000]
```

error code是一个字位组合，定义在内核[fault.c](http://lxr.free-electrons.com/source/arch/x86/mm/fault.c?v=4.9#L30)中:

```
/*
 * linux kernel v4.9
 * Page fault error code bits:
 *
 *   bit 0 ==    0: no page found       1: protection fault
 *   bit 1 ==    0: read access         1: write access
 *   bit 2 ==    0: kernel-mode access  1: user-mode access
 *   bit 3 ==                           1: use of reserved bit detected
 *   bit 4 ==                           1: fault was an instruction fetch
 *   bit 5 ==                           1: protection keys block access
 */
enum x86_pf_error_code {

        PF_PROT         =               1 << 0,
        PF_WRITE        =               1 << 1,
        PF_USER         =               1 << 2,
        PF_RSVD         =               1 << 3,
        PF_INSTR        =               1 << 4,
        PF_PK           =               1 << 5,
};
```

*注：低版本内核至少定义了bit0~2。*

因此得到一个线索——段错误原因6是内存写入越界，写到了一个无法对应物理内存页的地址。

## 根源探究

我们知道，程序使用的内存可以分为以下几类：

* **代码段**，编译好的程序在内存中的存储位置，是只读的。
* **BSS段**，存放初始化了的全局变量和静态变量。
* **数据段**，存放未经初始化的全局变量和静态变量。
* **堆**，可从这里动态的申请内存用于存放变量。
* **栈**，函数参数、局部变量以及其它函数相关信息的存储位置。

### 1. 栈Stack

栈是内存中用于存放函数的各种临时变量的特定区域。有以下显著特点：

* 当函数将局部变量push进栈中时，栈（向内存地址减小的方向）增长；反之，pop出栈后，栈缩小。
* 编译器自动管理，无需手动申请和释放内存。
* 因为CPU对栈内存的管理之高效，栈中变量的读写会非常快。
* 有大小限制(OS-dependent)。
* 栈变量仅在创建它们的函数运行时存在，函数结束，相应变量自动释放。
* 一经创建，不能调整大小。

关于栈大小，参考博客[《进程栈大小 与 线程栈大小》](http://blog.csdn.net/xhhjin/article/details/7579145)，有以下结论：
1. 进程的栈大小是在进程执行的时刻才能指定的，即不是在编译的时刻决定，也不是链接的时刻决定
2. 进程的栈大小是随机确定的，至少比线程的栈要大，但是不到线程栈大小的2倍
3. 线程栈的大小是固定的（可配置的），也就是ulimit -a显示的值

我测试环境的线程栈大小是10MB。

```
$ ulimit -a  
-t: cpu time (seconds)         unlimited
-f: file size (blocks)         unlimited
-d: data seg size (kbytes)     unlimited
-s: stack size (kbytes)        10240
-c: core file size (blocks)    0
-m: resident set size (kbytes) unlimited
-u: processes                  257160
-n: file descriptors           1024
-l: locked-in-memory size (kb) 64
-v: address space (kb)         unlimited
-x: file locks                 unlimited
-i: pending signals            257160
-q: bytes in POSIX msg queues  819200
-e: max nice                   0
-r: max rt priority            0
```

### 2. 堆Heap

堆是一种更加自由的内存区域，与栈相比，堆有以下显著特点：

* 手动管理内存

    申请和释放都由使用者自行负责，new/delete、malloc/free成对出现，逻辑不完整就可能出现内存泄露memory leak问题。

* 堆变量可以在全局范围访问

    生命周期是由使用者的申请和释放操作位置决定的。

* 无大小限制
* 相对于栈，访问速度较慢
* 可能产生碎片
* 堆变量可以动态的调整大小

    如通过realloc()函数。

## 解决方案

bitset<0xffffffff>至少需要2^32/8B=512MB的空间，远超默认栈大小10M。coredump问题由此产生。

改为从堆上申请空间，问题解决：

```
#include <bitset>

int main ()
{
    std::bitset<0xffffffff>* foo = new std::bitset<0xffffffff>;
    (*foo).reset();
    return 0;
}
```


## 参考

1. [Memory : Stack vs Heap](http://gribblelab.org/CBootcamp/7_Memory_Stack_vs_Heap.html)
