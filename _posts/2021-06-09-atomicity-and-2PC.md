---
layout: post
title:  "原子性和2PC"
date:   2021-06-09 15:10:29 +0800
comments: true
categories: 数据库 
---

## 什么是事务的原子性
事务的原子性指all-or-nothing，即或者全做all，或者全不做nothing。做或者不做的标准在于持久化，即如果一个事务完成了持久化，则认为其做了；如果没有完全持久化，则认为没有做。

也即是说，**原子性以持久性为前提**。

## 事务原子性实现
2PC（2-Phrase Commit）协议是实现事务原子性的基本方法。

事务操作往往涉及一个复杂的计算过程，横跨一个时间段，而原子性则体现在操作的即时性instancy。为了实现事务操作的原子性，一个基本策略是将事务分为两部分：一个是实施/准备阶段，占据事务的绝大部分时间；一个是提交，极度简化。然后将事务操作的即时性绑定在提交动作上：如果已提交则认定事务全做all，否则认定事务全不做nothing。

单节点服务下，比如传统的数据库管理系统RDBMS的事务原子性提交体现为在log中写上commit record。

分布式场景下的2PC是单节点场景下的自然延伸，事务是否commit取决于多个分布式节点，为了支持2PC，有一个节点被作为协调者，是否commit的判断也被绑定在协调者是否记录commit record的行为上，当然，多节点达成一致（commit or abort）的过程更复杂。典型的应用比如X/Open XA(The goal of XA is to guarantee atomicity in "global transactions")。分布式场景下的2PC本质上也是一种consensus算法，只不过它存在一个根本缺陷[<sup>1</sup>](#refs)，it is a blocking protocol：After a participant has sent an agreement message to the coordinator, it will block until a commit or rollback is received.

 因为这个原因,If the coordinator fails permanently, some participants will never resolve their transactions，所以it is not resilient to all possible failure configurations, and in rare cases, manual intervention is needed to remedy an outcome. 即它的容错性不高，特殊情况下的故障恢复需要人工介入（比如协调者故障，或者在commit时有参与者发生故障）

3PC是基于2PC的改良，它将2PC下的commit分解为prepare-commit和do-commit两个阶段，从而解决了异常场景（在commit时，participant fail 或者coordinator和participant同时fail）下的自动recover问题，避免了人工介入，具体可参考 [<sup>2</sup>](#refs)，但它仍存在2PC的阻塞缺陷。

Lamport 后来提出了著名的Paxos协议，其基础方法为 state machine replication approach，**Paxos does not block as long as a majority of processes (managers) are correct**. Paxos actually solves the more general problem of consensus, hence, it can be used to implement transaction commit as well. 多年后，Lamport又写了一篇论文[<sup>3</sup>](#refs)，从事务和事务提交的角度来专门比较2PC和Paxos，几个关键论点摘录如下：
1. Two-Phase Commit is the classical transaction commit protocol. Indeed, it is sometimes thought to be synonymous with transaction commit
2. Two-Phase Commit is not fault-tolerant because it uses a single coordinator whose failure can cause the protocol to block. Paxos Commit, a new transaction commit protocol that uses multiple coordinators
and makes progress if a majority of them are working.
3. **Two-Phase Commit is a degenerate case of the Paxos Commit algorithm with a single coordinator**2PC提交是Paxos提交算法在只使用一个协调者下的蜕化案例.


<div id="refs"></div>

参考：
- [^1] [Paxos vs two phase commit](https://stackoverflow.com/questions/27304887/paxos-vs-two-phase-commit)
- [2] [Consensus, Two Phase and Three Phase Commits](https://medium.com/@balrajasubbiah/consensus-two-phase-and-three-phase-commits-4e35c1a435ac)
- [3] GRAY, J., LAMPORT, L.  2004.  Consensus on Transaction Commit. 


