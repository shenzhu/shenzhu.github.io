---
title: "当我们谈论一致性时，我们在谈论什么"
tags:
  - 分布式系统
---

平时阅读以及在一些技术讨论中，我们经常会用到"一致性"这个词，它被使用地非常宽泛以至于大家在用到这个名词的时候会理所当然地以为它表达的意思很准确，而不去细想这到底意味着什么，甚至两个对话者对这一概念的理解都不甚相同。为了明确什么是"一致性"，这里我们要分清两个概念"一致性(consistency)"和"共识(consensus)"。

## 共识(Consensus)

在计算机里，我们经常做的一件事情就是抽象(abstraction)，加层(layer)从而separate concern以及hide complexity，比如操作系统作为软硬件接口，对上层应用程序屏蔽硬件细节。这是一种非常好的做法，但同时我们也要认识到，可抽象的东西不仅仅局限于实际的事物，对于概念也可以进行抽象。

一种我们经常用到的抽象就是事务，如果没有事务的话，应用程序需要考虑硬件是否出故障，程序跑了一半失败怎么办，以及怎样处理来自多个程序的并发，而事务的出现，屏蔽了单机多并发情况下的各种细节，借助于事务我们可以很方便的构建复杂的业务逻辑。

同样地，在多个机器一起完成某些任务或者处理某些问题的时候，我们也有一个很重要的抽象，这就是共识(consensus)。顾名思义，共识指的就是集群中的机器就某件事情达成一致，在这基础上，我们可以让这些机器对谁是leader达成一致(leader election)，对数据的划分达成一致(sharding)，而且如果我们能让集群中的机器对一件事情达成一致，也能让它们对多个事情达成一致，于是这些机器可以有完全相同的WAL，这也就是basic Paxos和multi Paxos的基本思想。

共识的实现需要特定的共识算法，最常用的是Paxos和Raft

## 一致性(Consistency)

说了一大圈共识，那一致性又是什么呢？在平时的讨论中，我们大概会在以下几种情况下听到"一致性":

- ACID: 事务的性质中，第二条C指的是consistency
- Consistent Hashing: 一种特殊的哈希算法
- Replication: Leader和follower要保持一致
- BASE: Basically available, soft state, eventual consistency
- CAP Therom: 在实际系统设计中，由于network partition不可避免，我们只能在availability与consistency中选择一个

这些情况里提到的一致性含义各不相同，我们分别来讨论一下

- ACID: 事务的"一致性"指的是在事务执行的过程中，业务逻辑层面要保证的某些不变量，比如转账的时候要保证账户数额总数不变。但需要注意的是，这里的"一致性"并不是数据库层面的性质，而是应用程序层面，在最坏的情况下，如果应用程序代码有误，强行操作数据库打破不变量，数据库也没办法阻止
- Consistent Hashing: Consistent hashing算法的大概意思是将整个空间当成一个环，server node属于环上的某些节点，record被hash到环的某个位置，之后顺时针遇到的第一个server node就是它应该在的server node。说实话我也没太明白它为什么叫"consistent" hashing...可能是指如果server node有变化，大部分record的mapping保持一致？关键是理解这里的"一致性"与其他地方不同
- Replication/BASE: 这两处一致性的概念都一样，指的是follower和leader之间没有replication lag，数据完全同步
- CAP Therom: CAP里consistency指的其实是linearizability(线性一致性)，它是一种recency guarantee，大致思想是多个client在与多个server node交互读写某个value的时候，整体看起来的样子似乎是这个value只有一个副本，也就是说它是站在观察者的角度而言。举个例子，假设我们有3个事务A，B，C，B在A结束之前开始，C在A，B都结束之后开始，那么下面这两种情况都符合linearizability，A写了x，B读到了C也读到了，以及A写了x，B没读到C读到了，但是A写了x，B读到了C没读到的情况不满足linearizability

