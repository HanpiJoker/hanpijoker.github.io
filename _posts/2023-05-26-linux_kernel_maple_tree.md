---
layout: post
author: HanpiJoker
title: Linux Kernel MapleTree
---

## Linux Kernel MapleTree

> 资料引用说明：
>
> 1. [The Maple Tree, A Modern Data Structure for a Complex Problem](https://blogs.oracle.com/linux/post/the-maple-tree-a-modern-data-structure-for-a-complex-problem)
> 2. [Introducing maple trees](https://lwn.net/Articles/845507/)
> 3. Linux Kernel Source Code
> 4. [Maple Tree](https://www.kernel.org/doc/html/next/core-api/maple_tree.html)
> 5. [Youtube The Linux Maple Tree - Matthew Wilcox, Oracle](https://www.youtube.com/watch?v=XwukyRAL7WQ)
> 6. [Youtube PPT](https://static.sched.com/hosted_files/ossna19/79/20190822_LinuxCon_Maple_Tree.pdf)
> 7. [Maple Tree v2 Patches For The Linux Kernel - 13~840% Faster For Malloc Threads Test Case](https://www.phoronix.com/news/Maple-Tree-v2)
> 8. [[PATCH 00/94\] Introducing the Maple Tree - Liam Howlett (kernel.org)](https://lore.kernel.org/linux-mm/20210428153542.2814175-1-Liam.Howlett@Oracle.com/)
> 9. [rcuvm:asplos12.pdf (mit.edu)](https://pdos.csail.mit.edu/papers/rcuvm:asplos12.pdf)

## MapleTree 介绍

### 1. 为什么引入 MapleTree

当前内核中是通过 vma_area_struct 来管理虚拟内存块的。每个 vma_area_strcut 包含了与内存映射相关的信息，内核对于 vma_area_struct 的访问是十分频繁的。所以为了保证性能。对于 vma_area_struct 的管理需要尽可能的确保性能和并法；目前内核优化 vma_area_struct 采取了以下的方案：

1. 使用的是红黑树和双向链表结合的方式，来实现节点的快速插入和删除。
2. 通过额外管理 vma 之前的间隙来确保可以更快的获取到所需的虚拟内存间隙；
3. 使用读写信号量管理结构体的并发访问，在保证互斥的基础上尽可能提高并发度。

即使做了上述的优化，在 Oracle 的大佬们看来，针对 vma_area_struct 的管理依旧是存在问题亟待提升的：

- vma_area_struct 与红黑树以及链表是深度绑定的，所以在涉及通过红黑树以及链表进行相关操作的时候，需要将整个 vma_area_struct 都放入 cache 中，目前 vma_area_struct 已经庞大到需要三个 cacheline 来存储。这无法充分利用硬件的预取机制进行性能优化；
- vma_area_struct 通过读写信号量的方式来管理并发访问，虽然能一定程度上提高并发访问效率。但是依旧不是最佳解（RCU）。不过由于红黑树需要旋转的特点，导致实现一个无锁的 RCU Rbtree 变得十分困难。

因此 Oracle 的大佬们梳理需求得到需要一种可以跟踪管理间隙、存储数据范围以及可以通过 RCU 实现无锁化的新的数据结构来替代现有的 RBtree + LinkedList。

### 2. MapleTree 是什么

MapleTree 是一种多叉树的数据结构，针对虚拟内存管理场景额外支持了范围查找，RCU 支持，以及针对缓存机制进行了优化可以快速访问前继和后继节点。

> 这里提及的针对缓存机制进行的优化指的是：MapleTree 类似与 RadixTree，管理节点与被管理的结构体是分离的存在，在管理节点中存储指向结构体的指针。相比于 RBtree 的嵌入式结构的优势在于：遍历和查找前后节点的过程中只需要将管理节点占用的内存加载到缓存中，一方面是遍历过程中不需要将部分遍历不关心的数据加载到缓存中，另一方面是可以在一条 cacheLine 中缓存多个节点提高了遍历过程中的访存效率。
>
> 这里后续看代码发现，MapleTree 设计节点的时候，保证了节点的结构体大小是固定的 256Byte，这是常用的 cacheline size 的整数倍。

下图是 RBtree，RadixTree，MapleTree 的具体区别：

![image-20230528235709683](https://github.com/HanpiJoker/hanpijoker.github.io/raw/master/Pictures/image_20230526_LinuxMapleTree_001.png)

### 3. MapleTree 带来的具体提升

目前上述的邮件连接中给出了一部分的性能测试数据，不过由于在测试的过程中还是使用 mmap_sem 保证代码的原子性的，所以在并发的优化上还没有真正体现。

> **While still using the mmap_sem, the performance seems fairly similar on real-world workloads, while there are variations in micro-benchmarks.**
>
> **Increase in performance in the following micro-benchmarks in Hmean:**
> **- wis malloc1-threads: Increase of 13% to 840%**
> **- wis page_fault1-threads: Increase of 1% to 14%**
> **- wis brk1-threads: Disregard, this test is invalid.**
>
> 
> **Decrease in performance in the following micro-benchmarks in Hmean:**
> **- wis brk1-processes: Decrease of 45% due to RCU required**
>
> **Mixed:**
> **- wis pthread_mutex1-threads: +11% to -3%**
> **- wis signal1-threads: +6% to -12%**
> **- wis malloc1-processes: +9% to -18% (-18 at 2 processes, increases after)**
> **- wis page_fault3-threads: +8% to -22%**

这里的性能测试使用的是 micro-benchmark 进行测试的，针对不同的测试项有不同的表现。不过具体每个测试项的侧重点是什么，这是暂时不太清楚的。目前由于 VMA 只是简单将 RBtree + LinkedList 替换为 MapleTree，还没有对锁的使用进行优化。所以这里看到的提升主要针对与 cache miss 的优化。还看不到使用 RCU 之后针对并发的优化。

## MapleTree 的原理

> MapleTree 的代码解析和原理实现比较复杂，这里还需要时间去进一步学习。

## MapleTree 的使用

MapleTree 暴露的接口分为两部分：基础接口(mtree_xxx 为前缀)和高级接口(mas_xxx 为前缀)

### 1. 基础接口的使用

```c
// 根节点结构体
struct maple_tree {
	union {
        spinlock_t ma_lock;
        lockdep_map_p ma_external_lock;
    };
    void __rcu *ma_root;
    unsigned int ma_flags;
};

// 子结点结构体
struct maple_node {
	union {
		struct {
			struct maple_pnode *parent;
			void __rcu *slot[MAPLE_NODE_SLOTS];
		};
		struct {
			void *pad;
			struct rcu_head rcu;
			struct maple_enode *piv_parent;
			unsigned char parent_slot;
			enum maple_type type;
			unsigned char slot_len;
			unsigned int ma_flags;
		};
		struct maple_range_64 mr64;  // 普通的节点，支持范围查找的结构体句柄；
		struct maple_arange_64 ma64; // 会保留空闲地址段的(gap) 的结构体句柄；
 		struct maple_alloc alloc;  // 用于节点申请和释放的时候的相关信息
	};
};

// 1. 初始化 mapletree
MTREE_INIT(name, __flags); // 静态初始化一个 maple tree
MTREE_INIT_EXT(name, __flags, __lock); // 静态初始化一个使用外部锁的 maple tree
mt_init_flags(struct maple_tree *mt, unsigned int flags); // 函数调用初始化一个 maple tree

// 2. 节点插入
// mtree_insert_xx 如果传入范围或是 index 是空闲的就将 entry 插入；
mtree_insert(struct maple_tree *mt, unsigned long index, void *entry, gfp_t gfp);
mtree_insert_range(struct maple_tree *mt, unsigned long first, unsigned long last, void *entry, gfp_t gfp);
// mtree_store_xx 对传入范围或是 index 进行覆盖；
mtree_store(struct maple_tree *mt, unsigned long index, void *entry, gfp_t gfp);
mtree_store_range(struct maple_tree *mt, unsigned long first, unsigned long last, void *entry, gfp_t gfp);
// mtree_alloc_xx 从指定区间找到一段空闲的空间插入；
mtree_alloc_range(struct maple_tree *mt, unsigned long *strartp, void *entry, unsigned long size, unsigned long min, unsigned long max, gfp_t gfp);
mtree_alloc_rrange(struct maple_tree *mt, unsigned long *strartp, void *entry, unsigned long size, unsigned long min, unsigned long max, gfp_t gfp);

// 3. 节点查找
mtree_load(struct maple_tree *mt, unsigned long index); // 根据传入的 index 单点查找
mt_find(struct maple_tree *mt, unsigned long *index, unsigned long max); // 根据范围查找节点
mt_for_each(__tree, __entry, __index, __max)；// 查找在区间内的多个节点

// 4. 节点删除
mtree_erase(struct maple_tree *mt, unsigned long index);

// 5. maple tree 销毁
mtree_destroy(struct maple_tree *mt);
```

### 3. 高级接口的使用

高级接口相比基础接口提供了更高的灵活性，在使用上也可以与基础接口混用。以下针对高级接口的使用介绍源自于内核的 Documents

```c
// 高级接口的核心是 struct ma_state 结构体, 主要用于跟踪后续操作过程的相关信息
struct ma_state {
	struct maple_tree *tree;	/* The tree we're operating in */
	unsigned long index;		/* The index we're operating on - range start */
	unsigned long last;		/* The last index we're operating on - range end */
	struct maple_enode *node;	/* The node containing this entry */
	unsigned long min;		/* The minimum index of this node - implied pivot min */
	unsigned long max;		/* The maximum index of this node - implied pivot max */
	struct maple_alloc *alloc;	/* Allocated nodes for this operation */
	unsigned char depth;		/* depth of tree descent during write */
	unsigned char offset;
	unsigned char mas_flags;
};

// 使用高级接口前使用下面的宏定义声明和定义一个 struct ma_state
#define MA_STATE(name, mt, first, end)					\
	struct ma_state name = {					\
		.tree = mt,						\
		.index = first,						\
		.last = end,						\
		.node = MAS_START,					\
		.min = 0,						\
		.max = ULONG_MAX,					\
		.alloc = NULL,						\
		.mas_flags = 0,						\
	}
```

