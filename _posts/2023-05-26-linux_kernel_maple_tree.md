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

下图是 RBtree，RadixTree，MapleTree 的具体区别：

![image-20230528235709683](https://github.com/HanpiJoker/hanpijoker.github.io/raw/master/Pictures/image_20230526_LinuxMapleTree_001.png)
