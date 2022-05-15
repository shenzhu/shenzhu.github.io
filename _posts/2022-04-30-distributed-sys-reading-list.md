---
title: "分布式系统reading list(长期更新)"
tags:
  - 分布式系统
---

记录一下看过的分布式系统资料

### 概念
* 时间
  * [Lamport Clock](https://martinfowler.com/articles/patterns-of-distributed-systems/lamport-clock.html): 由于NTP不完全准确，因此不同server甚至是同一server的时间都是无法直接比较的，因此我们可以使用logical clock(Lamport clock)。基本思路是每个server维护Lamport clock，server在写的时候increment Lamport clock，并且将增加后的clock返回给client，这样client在多次写的时候这些行为都有order。两个限制 => 1. partial order; 2. 无法使用真实时间推断
  * [Hybrid Logical Clock](https://martinfowler.com/articles/patterns-of-distributed-systems/hybrid-clock.html): 同时使用真实时间和逻辑时间，Hybrid Logical Clock也可以通过48位真实时间+16位逻辑时间的方法转化为真实时间，解决了Lamport Clock的部分问题
  * [分布式系统的时间](https://www.jianshu.com/p/8500882ab38c): 介绍分布式系统中时间的概念以及常用的几种方法，Logical Clock, TrueTime API, Hybrid Logic Clock和Timestamp Oracle

### 分布式事务
* Percolator
  * [Percolator论文](https://research.google/pubs/pub36726/)
  * [Percolator论文阅读笔记](http://loopjump.com/percolator_paper_note/): 比较详细地描述了Percolator中对2PC的实现以及oberserver机制
  * [Google Percolator 的事务模型](http://andremouche.github.io/transaction/percolator.html)
  * [Percolator in TiKV](https://tikv.org/deep-dive/distributed-transaction/percolator/): 介绍Percolator模型以及TiKV的实现
  * [Optimized Percolator in TiKV](https://tikv.org/deep-dive/distributed-transaction/optimized-percolator/): 提到了对Percolator的几种优化，parallel prewrite，short value in write column和point read without timestamp
* Two-phase Commit
  * [分布式事务：两阶段提交与三阶段提交](https://segmentfault.com/a/1190000012534071)

### 分布式数据库
* Spanner
  * [Martin Kleppmann on Spanner](https://www.youtube.com/watch?v=oeycOVX70aE)：DDIA的作者高屋建瓴介绍spanner，侧重于spanner中事务的实现以及TrueTime API带来的causality consistency
* TiDB
  * [三篇文章了解 TiDB 技术内幕 - 说存储](https://pingcap.com/zh/blog/tidb-internal-1): TiKV怎样以key-value的形式高效稳定存储数据
  * [三篇文章了解 TiDB 技术内幕 - 说计算](https://pingcap.com/zh/blog/tidb-internal-2/): TiDB怎样实现关系模型与key-value模型的映射以及TiDB架构
  * [三篇文章了解 TiDB 技术内幕 - 谈调度](https://pingcap.com/zh/blog/tidb-internal-3): PD的作用以及原理