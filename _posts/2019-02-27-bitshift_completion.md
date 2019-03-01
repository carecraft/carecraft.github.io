---
layout:     post
title:      "位移操作与补齐"
category : basictheory
date:       2019-02-27
author:     "Max"
header-img: "img/post-bg-unix-linux.jpg"
catalog:    true
tags:
    - C++
---

## 问题描述

解码操作时，需要从buffer某处获取一个 2 字节的长度，结果得到一个非预期值。源码如下：

```
#define READ_SHORT(p)   ((p[0] << 8) | p[1])

int main ()
{
    char buffer[2] = {0x80, 0x81}；
    char *p = buffer;
    in len = READ_SHORT(p);
    ...
}
```

上述代码 len 长度为 -127。但若将 p 的类型改为 `unsigned char *`， 则 len 的结果为 32897。

## 根源探究

### 移位运算

对于一个位表示为 $ [x_{\omega-1},x_{\omega-2},...,x_0] $ 的操作数 `x`，C表达式 `x<<k` 会生成一个值，其位表示为 $ [x_{\omega-k-1},x_{\omega-k-2},...,x_0,0,...,0] $ ，也即是说，`x` 向左移动 `k` 位，丢弃最高 `k` 位，并在右端补 `k` 个 0。

相应的右移运算 `x>>k` 行为有点微秒。一般而言，机器支持两种形式的右移：逻辑右移和算数右移。逻辑右移在左端补 `k` 个 0，得到的结果是 $ [0,...,0,x_{\omega-1},x_{\omega-2},...,x_k] $。算数右移是在左端补 `k` 个 最高有效位的值，得到的结果是 $ [x_{\omega-1},...,x_{\omega-1},x_{\omega-1},x_{\omega-2},...,x_k] $。

C语言标准并没有明确定义对于有符号数应该使用哪种类型的右移，然而，几乎所有的编译器/机器组合都对有符号数使用算数右移，对无符号数使用逻辑右移。

与 C 相比，Java 对如何右移有明确的定义。表达式 `x>>k` 会将`x`算数右移`k`个位置，表达式 `x>>k` 会对`x`进行逻辑右移。

### 整数表示

假设有一个整数数据类型有 $\omega$ 位，我们可以用位向量 $ \vec x = [x_{\omega-1},x_{\omega-2},...,x_0] $ 表示其每一位。把 $ \vec x $ 看做一个二进制表示的数，就获得了 $ \vec x $ 的无符号表示。我们用一个函数 $ B2U_{\omega} $（Binary to Unsigned 的缩写，长度为 $\omega$）来表示：
$$
    B2U_{\omega}(\vec x) \doteq \sum_{i=0}^{\omega-1} x_i 2^i
$$

对于有符号数而言，几乎所有的机器都使用了补码（two's-complement）的形式来表示。在这个定义中，将字的最高有效位解释为负权（negative weight）：
$$
    B2T_{\omega}(\vec x) \doteq -x_{\omega-1} 2^{\omega-1} + \sum_{i=0}^{\omega-2} x_i 2^i
$$

由定义可知，补码的范围是不对称的，也即是 $ \vert TMin \vert = \vert TMax \vert + 1 $, $TMin$ 没有与之对应的正数；最大的无符号数值刚好比补码的最大值的两倍大一点，也即是 $ UMax_{\omega} = 2 TMax_{\omega} + 1 $。

### 类型转换

C 语言允许在各种不同的数字数据类型之间做强制类型转换。

#### 字长相同

对于大多数 C 语言的实现，处理同样字长的有符号数和无符号数之间相互转换的一般规则是：数值可能会改变，但是位模式不变。

定义 `short int v = -12345; unsigned short uv = (unsigned short)v;`，发现 `uv =  53191`，v 的 16 位补码与 uv 的 16 位无符号表示都是 `1100111111000111`。

给定 $ 0 \leq x \leq UMax_{\omega} $ 范围内的一个整数 $x$，定义函数 $ U2B_{\omega}(x) $ 给出 $x$ 的唯一的 $\omega$ 位无符号表示。相似的，当满足 $ TMin_{\omega} \leq x \leq TMax_{\omega} $，定义函数 $ T2B_{\omega} $ 给出 $x$ 的唯一的 $\omega$ 位补码表示。因此：

$$
T2U_{\omega} (x) \doteq B2U_{\omega}(T2B_{\omega}(x)) = x + x_{\omega-1} 2^{\omega}

U2T_{\omega} (u) \doteq B2T_{\omega}(U2B_{\omega}(u)) = u - u_{\omega-1} 2^{\omega}
$$

则上述转换示例可以表示为 $T2U_{16}(-12345) = -12345 + 2^{16} = 53191$。

#### 扩展数字

要将一个无符号数转换为一个更大的数据类型，我们只要简单的在表示的开头添加 0。这种运算也被称作零扩展。

要将一个补码数字转换为一个更大的数据类型，我们需要执行一个符号扩展，在表示中添加最高有效位的值。

给出如下示例：
```
short sx = -12345;        /* -12345,     编码为 cf c7 */
unsigned short usx = sx;  /* 53191,      编码为 cf c7 */
int x = sx;               /* -12345,     编码为 ff ff cf c7 */
unsigned ux = usx;        /* 53191,      编码为 00 00 cf c7 */
unsigned ux2 = sx;        /* 4294954951, 编码为 ff ff cf c7 */
```

注意最后一个转换，当把 short 转换为 unsigned 时，我们先改变大小，再改变符号。也就是说， `(unsigned)sx` 等价于 `(unsigned)(int)sx`，而不等价于`(unsigned)(unsigned short)sx`。这个规则是 C 语言标准要求的。

#### 截断数字

当将一个 $\omega$ 位表示为 $ [x_{\omega-1},x_{\omega-2},...,x_0] $ 的数截断为一个 $k$ 位数字时，我们会丢弃高 $\omega-k$位，得到 $ [x_{k-1},x_{k-2},...,x_0] $。
用数学的方式可以表示为：
$$
B2U_k([x_{k-1},x_{k-2},...,x_0]) = B2U_{\omega}([x_{\omega-1},x_{\omega-2},...,x_0]) mod 2^k
$$

补码截断也具有相似的属性，只不过要将最高位转换为符号位。数学表示如下：
$$
B2T_k([x_{k-1},x_{k-2},...,x_0]) = U2T_k(B2U_{\omega}([x_{\omega-1},x_{\omega-2},...,x_0]) mod 2^k)
$$


## 问题解释

在程序中经常遇到不同类型的数据进行运算，若一个运算符两侧的数据类型不同，则先自动进行类型转换，使两者具有同一类型，然后进行运算。故原始代码可分解为如下步骤：
```
char buffer[2] = {0x80, 0x81}；
char *p = buffer;
int p0 = p[0] << 8;       /* 编码为 ff ff 80 00 */
int p1 = (int)p[1];       /* 编码为 ff ff ff 81 */
int len = p0 | p1;        /* 编码为 ff ff ff 81 */
```

此时 $len = B2T_{32}(0xffffff81) = -127 $。若将 p 的类型改为 `unsigned char *`：
```
unsigned char *p = buffer;
unsigned int p0 = p[0] << 8;          /* 编码为 00 00 80 00 */
unsigned int p1 = (unsigned int)p[1]; /* 编码为 00 00 00 81 */
unsigned int up = p0 | p1;            /* 编码为 00 00 80 81 */
int len = (int)up;                    /* 编码为 00 00 80 81 */
```
此时 $len = B2T_{32}(0x8081) = 32897 $。


## 参考

1. 《深入理解计算机系统》 Randal E. Bryant, David R. O'Hallaron
