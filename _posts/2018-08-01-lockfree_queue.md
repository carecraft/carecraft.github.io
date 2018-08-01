---
layout:     post
title:      "A lock-free queue"
subtitle:   "无锁队列的一种实现"
category :  basictheory
date:       2018-08-01
author:     "Max"
header-img: "img/post-bg-algorithms.jpg"
catalog:    true
tags:
    - algorithms
    - lock-free
---

## 1. 数据结构

```
/** Ring producer status. */
struct prod_t
{
    uint32_t watermark;      /**< Maximum items before EDQUOT. Power of 2. */
    uint32_t sp_enqueue;     /**< True, if single producer. */
    uint32_t size;           /**< Size of ring. Power of 2. */
    uint32_t mask;           /**< Mask (size-1) of ring. */
    volatile uint32_t head;  /**< Producer head. */
    volatile uint32_t tail;  /**< Producer tail. */
};

/** Ring consumer status. */
struct cons_t
{
    uint32_t sc_dequeue;     /**< True, if single consumer. */
    uint32_t size;           /**< Size of the ring. Power of 2. */
    uint32_t mask;           /**< Mask (size-1) of ring. */
    volatile uint32_t head;  /**< Consumer head. */
    volatile uint32_t tail;  /**< Consumer tail. */
};

class CObjectUnLockQueue
{
    ...
    void   **m_ppObject;     /**< Object pointers of the ring. */
    struct prod_t m_stProd;
    struct cons_t m_stCons;
    ...
};
```

m_ppObject 当为队列存储对象指针的数组。在某一时刻，m_stProd.head（不含）与 m_stProd.tail（含） 之间为正在插入队列的对象，m_stCons.head（不含）与 m_stCons.tail（含）之间为正在取出队列的对象， m_stCons.head（含）与 m_stProd.tail（不含）之间为队列存储的对象。

```
m_stCons.tail     m_stProd.tail
 |                       |
---------------------------------------
 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 
---------------------------------------
         |                           |
   m_stCons.head                m_stProd.head
```

## 2. 入队操作

### 2.1 单生产者入队列

要向队列插入 n 个对象指针，则操作 m_stProd， 将 head 置为插入后可用位置的索引。在将对象写入队列后，将 tail 移动到可用位置的索引（同 head）即可。
```
    ...
    prod_head = m_stProd.head;
    prod_next = prod_head + n;
    m_stProd.head = prod_next;

    /* write entries in ring */
    const uint32_t size = m_stProd.size; 
    uint32_t idx = prod_head & m_stProd.mask; 
    if (likely(idx + n < size)) 
    { 
        for (i = 0; i < n; i++, idx++)
            m_ppObject[idx] = ppObject[i]; 
    } 
    else 
    { 
        for (i = 0; idx < size; i++, idx++)
            m_ppObject[idx] = ppObject[i]; 
        for (idx = 0; i < n; i++, idx++) 
            m_ppObject[idx] = ppObject[i]; 
    }
    
    asm volatile ("" : : : "memory");  // a compiler level memory barrier
    m_stProd.tail = prod_next;
    ...
```

### 2.2 多生产者入队列

与单生产者入队列相比，在操作 m_stProd.head 时可能受到其它线程的干扰，需要使用原子操作保证数据安全：

```
    do
    {
        prod_head = m_stProd.head;
        prod_next = prod_head + n;
        success = __sync_bool_compare_and_swap(&m_stProd.head, prod_head, prod_next);
    } while (unlikely(success == 0));
```

若有其他线程在当前线程之前开始插入（但还未完成写入操作，也未移动 m_stProd.tail），需等待其操作结束本线程再行继续：

```
    asm volatile ("" : : : "memory");
    while (unlikely(m_stProd.tail != prod_head))
        ;
    m_stProd.tail = prod_next;
```

## 3. 出队操作

### 3.1 单消费者出队列

与入队操作类似，要从队列取出 n 个对象指针，则操作 m_stCons， 先将 head 置为取出后队列首个元素的索引，然后将目标对象写出，最后将 tail 移动到队列首个元素的索引（同 head）即可。

```
    cons_head = m_stCons.head;
    cons_next = cons_head + n;
    m_stCons.head = cons_next;

    /* copy in table */
    uint32_t idx = cons_head & mask; 
    const uint32_t size = m_stCons.size; 
    if (likely(idx + n < size)) 
    { 
        for (i = 0; i < n; i++, idx++) 
            ppObject[i] = m_ppObject[idx]; 
    } 
    else 
    { 
        for (i = 0; idx < size; i++, idx++) 
            ppObject[i] = m_ppObject[idx]; 
        for (idx = 0; i < n; i++, idx++) 
            ppObject[i] = m_ppObject[idx]; 
    }

    asm volatile ("" : : : "memory");
    m_stCons.tail = cons_next;
```

### 3.2 多消费者出队列

与多消费者出队列相比，在操作 m_stCons.head 时可能受到其它线程的干扰，需要使用原子操作保证数据安全：

```
    do 
    {
        cons_head = m_stCons.head;
        cons_next = cons_head + n;
        success = __sync_bool_compare_and_swap(&m_stCons.head, cons_head, cons_next);
    } while (unlikely(success == 0));
```

同样，若有其他线程在当前线程之前开始出队（但还未完成写出操作，也未移动 m_stCons.tail），需等待其操作结束本线程再行继续：

```
    asm volatile ("" : : : "memory");
    while (unlikely(m_stCons.tail != cons_head))
        ;
    m_stCons.tail = cons_next;
```

## 4. 封装应用

### 4.1 内存池

申请一块大内存，按固定大小划分后放入队列，即可简单包装成一个无锁内存池。

```
class CUnLockBuffer
{
    ...
    char * m_stMemory; 
    CObjectUnLockQueue m_stMemQueue;
    ....
};
```