---
layout:     post
title:      "linux内核PID管理"
subtitle:   "management of PID in linux kernel"
category :  basictheory
date:       2017-03-22
author:     "Max"
header-img: "img/post-bg-algorithms.jpg"
catalog:    true
tags:
    - linux
---

## 1. 概念

什么是 PID ？

PID 是内核内部对进程的一个标识符，用来唯一标识一个进程（task）、进程组（process group）、会话（session）。PID 及其对应进程存储在一个哈希表中，方便依据 PID 快速访问进程 task_struct。

因此Linux 内核在设计管理ID的数据结构时，要充分考虑以下因素：

1. 如何快速地根据进程的 task_struct、ID 类型、命名空间找到 PID ？
2. 如何快速地根据 PID、命名空间、ID 类型找到对应进程的 task_struct ？
3. 如何快速地给新进程在可见的命名空间内分配一个唯一的 PID ？


*注：本文所附内核源码版本为 v4.11.6。*

## 2. 数据结构

### 2.1 进程ID

内核中 PID 的定义[include/linux/pid.h](http://elixir.free-electrons.com/linux/v4.11.6/source/include/linux/pid.h#L57)如下：
```
struct pid
{
	atomic_t count;
	unsigned int level;
	/* lists of tasks that use this pid */
	struct hlist_head tasks[PIDTYPE_MAX];
	struct rcu_head rcu;
	struct upid numbers[1];
};
```

* count 是该数据结构被引用的次数。

* level 是该 pid 在 pid\_namespace 中所处层级。当 level=0 时表示是 global namespace，即最高层。

* tasks[i] 指向 PID 对应的 task_struct。PIDTYPE\_MAX 是 pid 的类型数。一个或多个进程可以组成一个进程组，进程组ID（PGID）为进程组领导进程 (process group leader)的PID；一个或多个进程组可以组成一个会话，会话ID（SID）为会话领导进程（session leader）的PID。PIDTYPE\_MAX定义在[include/linux/pid.h](http://elixir.free-electrons.com/linux/v4.11.6/source/include/linux/pid.h#L11)中：
  ```
  enum pid_type
  {
  	PIDTYPE_PID,
  	PIDTYPE_PGID,
  	PIDTYPE_SID,
  	PIDTYPE_MAX
  };
  ```



* rcu 用于保证数据同步，具体机制另行讨论。

* numbers[1] 域是一个可扩展 upid 结构体。 一个 PID 可以属于不同的 namespace ， numbers[0] 表示 global namespace，numbers[i] 表示第 i 层 namespace，i 越大所在层级越低。upid 结构体定义[include/linux/pid.h](http://elixir.free-electrons.com/linux/v4.11.6/source/include/linux/pid.h#L50)如下:
  ```
  struct upid {
  	  /* Try to keep pid_chain in the same cacheline as nr for find_vpid */
  	  int nr;
  	  struct pid_namespace *ns;
  	  struct hlist_node pid_chain;
  };
  ```
  + nr是pid的值， 即 task\_struct 中 pid_t pid 域的值。

  + ns指向该 pid 所处的 namespace。

  + pid\_chain 是 pid\_hash 哈希表节点。linux内核将所有进程的upid都存放在一个哈希表（pid\_hash）中，以方便查找和统一管理。通过 pid\_chain 能够找到该 upid 所在 pid_hash 中的位置。



## 2.2 进程命名空间

再来看 pid 命名空间 pid_namespace 的定义（[include/linux/pid_namespace.h](http://elixir.free-electrons.com/linux/v4.11.6/source/include/linux/pid_namespace.h#L30)）

```
struct pid_namespace {
	struct kref kref;
	struct pidmap pidmap[PIDMAP_ENTRIES];
	struct rcu_head rcu;
	int last_pid;
	unsigned int nr_hashed;
	struct task_struct *child_reaper;
	struct kmem_cache *pid_cachep;
	unsigned int level;
	struct pid_namespace *parent;
	struct user_namespace *user_ns;
	struct ucounts *ucounts;
	struct work_struct proc_work;
	kgid_t pid_gid;
	int hide_pid;
	int reboot;	/* group exit code if this pidns was rebooted */
	struct ns_common ns;
};
```
* kref 表示指向 pid_namespace 的个数。

* pidmap 结构体表示分配pid的位图，pidmap[PIDMAP\_ENTRIES] 域存储了该 pid\_namespace 下 pid 已分配情况。pidmap 结构体定义（[include/linux/pid_namespace.h](http://elixir.free-electrons.com/linux/v4.11.6/source/include/linux/pid_namespace.h#L13)）如下：
  ```
  struct pidmap {
         atomic_t nr_free;
         void *page;
  };
  
  
  #define BITS_PER_PAGE		(PAGE_SIZE * 8)
  #define BITS_PER_PAGE_MASK	(BITS_PER_PAGE-1)
  #define PIDMAP_ENTRIES		((PID_MAX_LIMIT+BITS_PER_PAGE-1)/BITS_PER_PAGE)
  
  // include/linux/threads.h
  #define PID_MAX_DEFAULT (CONFIG_BASE_SMALL ? 0x1000 : 0x8000)
  /*
   * A maximum of 4 million PIDs should be enough for a while.
   * [NOTE: PID/TIDs are limited to 2^29 ~= 500+ million, see futex.h.]
   */
  #define PID_MAX_LIMIT (CONFIG_BASE_SMALL ? PAGE_SIZE * 8 : \
  	(sizeof(long) > 4 ? 4 * 1024 * 1024 : PID_MAX_DEFAULT))
  ```
  
  + nr_free 表示还能分配的 pid 的数量。
  
  + page 指向的是存放 pid 的物理页。

* rcu 同样用于保证数据同步。

* last_pid 是最后一个已分配的 pid。

* nr_hashed 统计该命名空间已分配PID个数。

* child\_reaper指向的是一个进程。 该进程的作用是当子进程结束时为其收尸（回收空间）。global namespace 中child\_reaper 指向 init\_task。

* pid_cachep 域指向分配 pid 的 slab 的地址。

* level 表示该命名空间所处层级。

* parent 指向该命名空间的父命名空间。

### 2.3 进程描述符

进程描述符 task_struct 是 linux 内核中最重要的概念之一，其结构复杂（[include/linux/sched.h](http://elixir.free-electrons.com/linux/v4.11.6/source/include/linux/sched.h#L483)），详情另行分析。其中，与 pid 有关的域如下:

```
struct task_struct {
	···
	pid_t                 pid;
	pid_t                 tgid;
	struct task_struct   *group_leader;
	struct pid_link       pids[PIDTYPE_MAX];
	struct nsproxy       *nsproxy;
	···
};
```
* pid 即是该进程的 id。使用 fork 或 clone 系统调用时产生的进程均会由内核分配一个新的唯一的PID值。
* tgid 是线程组 id。在一个进程中，如果以 CLONE_THREAD 标志来调用 clone 建立的进程就是该进程的一个线程，它们处于一个线程组。处于相同的线程组中的所有进程都有相同的 TGID；线程组组长的 TGID 与其 PID 相同；一个进程没有使用线程，则其 TGID 与 PID 也相同。
* group\_leader 除了在多线程的模式下指向主线程外， 当一些进程组成一个群组时（PIDTYPE_PGID)， 该域指向该群组的leader。
* pids[PIDTYPE\_MAX] 指向了和该 task_struct 相关的 pid 结构体。其定义（[include/linux/pid.h](http://elixir.free-electrons.com/linux/v4.11.6/source/include/linux/pid.h#L69)）如下：
  ```
  struct pid_link
  {
      struct hlist_node node;
      struct pid *pid;
  };
  ```

* nsproxy指针指向namespace相关的域。其定义（[include/linux/nsproxy.h](http://elixir.free-electrons.com/linux/v4.11.6/source/include/linux/nsproxy.h#L30)）如下：
  ```
  struct nsproxy {
	  atomic_t count;
	  struct uts_namespace *uts_ns;
	  struct ipc_namespace *ipc_ns;
	  struct mnt_namespace *mnt_ns;
	  struct pid_namespace *pid_ns_for_children;
	  struct net 	     *net_ns;
	  struct cgroup_namespace *cgroup_ns;
  };
  ```
  + 与其它命名空间不同。此处 pid\_ns\_for\_children 指向该进程的子进程会使用的 pid\_namespace，该进程本身所属的 pid\_namespace 可以通过 task\_active\_pid\_ns 方法获得。
  + nsproxy 被所有共享命名空间的 task_struct 共享，随命名空间的复制而复制。

### 2.4 数据结构关系

举例来说，在level 2 的某个命名空间上新建了一个进程，分配给它的 pid 为45，映射到 level 1 的命名空间，分配给它的 pid 为 134；再映射到 level 0 的命名空间，分配给它的 pid 为289，对于这样的例子，如下图所示为其表示：

![pid_namespace_example](/img/in-post/pid_manage/pid_namespace.png)
*结构图示例，图片来源[郭海林的博客](http://www.cnblogs.com/hazir/p/linux_kernel_pid.html)*

## 3. 方法/函数

现在就可以解决本文开篇提出的问题了。

### 3.1 查询PID

1. 获取与 task_struct 相关的 pid 结构体实例

    [include/linux/sched.h](http://elixir.free-electrons.com/linux/v4.11.6/source/include/linux/sched.h#L1056)
  
    ```
    static inline struct pid *task_pid(struct task_struct *task)
    {
    	return task->pids[PIDTYPE_PID].pid;
    }
    
    static inline struct pid *task_tgid(struct task_struct *task)
    {
    	return task->group_leader->pids[PIDTYPE_PID].pid;
    }
    
    static inline struct pid *task_pgrp(struct task_struct *task)
    {
    	return task->group_leader->pids[PIDTYPE_PGID].pid;
    }
    
    static inline struct pid *task_session(struct task_struct *task)
    {
    	return task->group_leader->pids[PIDTYPE_SID].pid;
    }
    ```

2. 获取与 task_struct 相关的 PID 命名空间

    [kernel/pid.c](http://elixir.free-electrons.com/linux/v4.11.6/source/kernel/pid.c#L544)
  
    ```
    struct pid_namespace *task_active_pid_ns(struct task_struct *tsk)
    {
        return ns_of_pid(task_pid(tsk));
    }
    ```
  
    [include/linux/pid.h](http://elixir.free-electrons.com/linux/v4.11.6/source/include/linux/pid.h#L134)
    
    ```
    static inline struct pid_namespace *ns_of_pid(struct pid *pid)
    {
        struct pid_namespace *ns = NULL;
        if (pid)
    	      ns = pid->numbers[pid->level].ns;
        return ns;
    }
    ```

3. 获取 pid 实例中的 PID

    [kernel/pid.c](http://elixir.free-electrons.com/linux/v4.11.6/source/kernel/pid.c#L519)
    
    ```
    pid_t pid_nr_ns(struct pid *pid, struct pid_namespace *ns)
    {
    	struct upid *upid;
    	pid_t nr = 0;
    
    	if (pid && ns->level <= pid->level) {
    		upid = &pid->numbers[ns->level];
    		if (upid->ns == ns)
    			nr = upid->nr;
    	}
    	return nr;
    }
    ```
    获取指定 ns 中的 PID时，需注意，由于 PID 命名空间的层次性，父命名空间能看到子命名空间的内容，反之则不能。因此，函数  中  需要确保当前命名空间的 level 小于等于产生局部 PID 的命名空间的 level。
    
    此外，内核还封装有直接获取初始命名空间、当前命名空间对应 PID 的方法：
    ```
    static inline pid_t pid_nr(struct pid *pid)
    {
    	pid_t nr = 0;
    	if (pid)
    		nr = pid->numbers[0].nr;
    	return nr;
    }
    
    pid_t pid_vnr(struct pid *pid)
    {
    	return pid_nr_ns(pid, task_active_pid_ns(current));
    }
    ```

### 3.2 查找 task_struct

1. 获得 pid 实体。

    根据 PID 以及指定命名空间计算在 pid\_hash 数组中的索引，然后遍历散列表找到所要的 upid， 再根据内核的   container\_of 机制找到 pid 实例。代码（[kernel/pid.c](http://elixir.free-electrons.com/linux/v4.11.6/source/kernel/pid.c#L365)）如下：
  
    ```
    struct pid *find_pid_ns(int nr, struct pid_namespace *ns)
    {
    	  struct upid *pnr;
    
    	  hlist_for_each_entry_rcu(pnr,
    			  &pid_hash[pid_hashfn(nr, ns)], pid_chain)
    		  if (pnr->nr == nr && pnr->ns == ns)
    	  		  return container_of(pnr, struct pid,
    					  numbers[ns->level]);
       
    	       return NULL;
    }
    ```
    由此，也可以根据当前命名空间下的局部 PID 获取对应的 pid实例：
    ```
    struct pid *find_vpid(int nr)
    {
      	return find_pid_ns(nr, task_active_pid_ns(current));
    }
    ```

2. 根据 pid 及 PID 类型获取 task_struct

    ```
    struct task_struct *pid_task(struct pid *pid, enum pid_type type)
    {
    	struct task_struct *result = NULL;
    	if (pid) {
    		struct hlist_node *first;
    		first = rcu_dereference_check(hlist_first_rcu(&pid->tasks[type]),
    					      lockdep_tasklist_lock_is_held());
    		if (first)
    			result = hlist_entry(first, struct task_struct, pids[(type)].node);
    	}
    	return result;
    }
    ```

### 3.3 分配PID

1. 为新进程 task_struct 分配 pid 域

    新的进程使用 alloc_pid 方法分配 pid，代码（[kernel/pid.c](http://elixir.free-electrons.com/linux/v4.11.6/source/kernel/pid.c#L296)）如下：
    
    ```
    struct pid *alloc_pid(struct pid_namespace *ns)
    {
    	struct pid *pid;
    	enum pid_type type;
    	int i, nr;
    	struct pid_namespace *tmp;
    	struct upid *upid;
    	int retval = -ENOMEM;
    
        // 从命名空间分配一个 pid 结构体
    	pid = kmem_cache_alloc(ns->pid_cachep, GFP_KERNEL);
    	if (!pid)
    		return ERR_PTR(retval);
    
        // 初始化进程在各级命名空间的 PID，直到全局命名空间（level 为0）为止
    	tmp = ns;
    	pid->level = ns->level;
    	for (i = ns->level; i >= 0; i--) {
    		nr = alloc_pidmap(tmp);  //分配一个局部PID
    		if (nr < 0) {
    			retval = nr;
    			goto out_free;
    		}
    
    		pid->numbers[i].nr = nr;
    		pid->numbers[i].ns = tmp;
    		tmp = tmp->parent;
    	}
    
        // 若为命名空间的初始进程
    	if (unlikely(is_child_reaper(pid))) {
    		// 0
    		if (pid_ns_prepare_proc(ns))
    			goto out_free;
    	}
    
    	get_pid_ns(ns);
    	atomic_set(&pid->count, 1);
    	for (type = 0; type < PIDTYPE_MAX; ++type)
    		INIT_HLIST_HEAD(&pid->tasks[type]); // // 初始化 pid->task[] 结构体，值为NULL
    
    	upid = pid->numbers + ns->level;
    	spin_lock_irq(&pidmap_lock);
    	if (!(ns->nr_hashed & PIDNS_HASH_ADDING))
    		goto out_unlock;
    	for ( ; upid >= pid->numbers; --upid) {
    		// 将每个命名空间经过哈希之后加入到散列表中
    		hlist_add_head_rcu(&upid->pid_chain,
    				&pid_hash[pid_hashfn(upid->nr, upid->ns)]);
    		upid->ns->nr_hashed++;
    	}
    	spin_unlock_irq(&pidmap_lock);
    
    	return pid;
    
    out_unlock:
    	spin_unlock_irq(&pidmap_lock);
    	put_pid_ns(ns);
    
    out_free:
    	while (++i <= ns->level)
    		free_pidmap(pid->numbers + i);
    
    	kmem_cache_free(ns->pid_cachep, pid);
    	return ERR_PTR(retval);
    }
    ```

2. 从指定命名空间中分配唯一PID

    [kernel/pid.c](http://elixir.free-electrons.com/linux/v4.11.6/source/kernel/pid.c#L153)
    
    ```
    static int alloc_pidmap(struct pid_namespace *pid_ns)
    {
    	int i, offset, max_scan, pid, last = pid_ns->last_pid;
    	struct pidmap *map;
    
    	pid = last + 1;
    	// 默认最大值在 include/linux/threads.h 中定义为 (CONFIG_BASE_SMALL ? 0x1000 : 0x8000)
    	if (pid >= pid_max)  
    		pid = RESERVED_PIDS;  // RESERVED_PIDS = 300
    	offset = pid & BITS_PER_PAGE_MASK;
    	map = &pid_ns->pidmap[pid/BITS_PER_PAGE];
    	/*
    	 * If last_pid points into the middle of the map->page we
    	 * want to scan this bitmap block twice, the second time
    	 * we start with offset == 0 (or RESERVED_PIDS).
    	 */
    	max_scan = DIV_ROUND_UP(pid_max, BITS_PER_PAGE) - !offset;
    	for (i = 0; i <= max_scan; ++i) {
    		if (unlikely(!map->page)) {
    			void *page = kzalloc(PAGE_SIZE, GFP_KERNEL);
    			/*
    			 * Free the page if someone raced with us
    			 * installing it:
    			 */
    			spin_lock_irq(&pidmap_lock);
    			if (!map->page) {
    				map->page = page;
    				page = NULL;
    			}
    			spin_unlock_irq(&pidmap_lock);
    			kfree(page);
    			if (unlikely(!map->page))
    				return -ENOMEM;
    		}
    		if (likely(atomic_read(&map->nr_free))) {
    			for ( ; ; ) {
    				if (!test_and_set_bit(offset, map->page)) {
    					atomic_dec(&map->nr_free);
    					set_last_pid(pid_ns, last, pid);
    					return pid;
    				}
    				offset = find_next_offset(map, offset);
    				if (offset >= BITS_PER_PAGE)
    					break;
    				pid = mk_pid(pid_ns, map, offset);
    				if (pid >= pid_max)
    					break;
    			}
    		}
    		if (map < &pid_ns->pidmap[(pid_max-1)/BITS_PER_PAGE]) {
    			++map;
    			offset = 0;
    		} else {
    			map = &pid_ns->pidmap[0];
    			offset = RESERVED_PIDS;
    			if (unlikely(last == offset))
    				break;
    		}
    		pid = mk_pid(pid_ns, map, offset);
    	}
    	return -EAGAIN;
    }
    ```

3. 回收PID

    [kernel/pid.c](http://elixir.free-electrons.com/linux/v4.11.6/source/kernel/pid.c#L104)
    
    ```
    static void free_pidmap(struct upid *upid)
    {
    	int nr = upid->nr;
    	struct pidmap *map = upid->ns->pidmap + nr / BITS_PER_PAGE;
    	int offset = nr & BITS_PER_PAGE_MASK;
    
    	clear_bit(offset, map->page);
    	atomic_inc(&map->nr_free);
    }
    ```

## 4. 参考

1. [Ian Zhang的专栏《linux内核PID管理》](http://blog.csdn.net/zhanglei4214/article/details/6765913)
2. [郭海林的博客《Linux 内核进程管理之进程ID》](http://www.cnblogs.com/hazir/p/linux_kernel_pid.html)