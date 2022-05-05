---
title: "分布式系统reading list(长期更新)"
tags:
  - 分布式系统
---

### 概念
* 时间
  * [Lamport Clock](https://martinfowler.com/articles/patterns-of-distributed-systems/lamport-clock.html): 由于NTP不完全准确，因此不同server甚至是同一server的时间都是无法直接比较的，因此我们可以使用logical clock(Lamport clock)。基本思路是每个server维护Lamport clock，server在写的时候increment Lamport clock，并且将增加后的clock返回给client，这样client在多次写的时候这些行为都有order。两个限制 => 1. partial order; 2. 无法使用真实时间推断
  * [Hybrid Logical Clock](https://martinfowler.com/articles/patterns-of-distributed-systems/hybrid-clock.html): 同时使用真实时间和逻辑时间，Hybrid Logical Clock也可以通过48位真实时间+16位逻辑时间的方法转化为真实时间，解决了Lamport Clock的部分问题
  * [分布式系统的时间](https://www.jianshu.com/p/8500882ab38c): 介绍分布式系统中时间的概念以及常用的几种方法，Logical Clock, TrueTime API, Hybrid Logic Clock和Timestamp Oracle

### 分布式数据库
* Spanner
  * [Martin Kleppmann on Spanner](https://www.youtube.com/watch?v=oeycOVX70aE)：DDIA的作者高屋建瓴介绍spanner，侧重于spanner中事务的实现以及TrueTime API带来的causality consistency