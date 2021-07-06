---
layout: post
title:  "持久化和WAL"
comments: true
categories: 数据库 
tags: WAL,no-force,steal,checkpoint
---
## 持久化Duration的本质
持久化的本质是数据落盘，有两种确认数据落盘的方式
1. 将更新的数据（raw data）直接写到外部非易失性存储上
2. 将更新数据的相关日志写到外部非易失性存储上

## 什么是WAL
WAL(Write-Ahead-Log)是第2种，即只要日志落盘，即认为完成持久化。WAL是一种sequential结构，写WAL的本质是顺序追加写。对于常见的磁盘来说，其效率比将更新的数据随机落盘要高几个数量级，这是WAL作为常见持久化机制的根本原因。

WAL包括两类日志，一种是redo log，另外一种是undo log。WAL中的Ahead的意思，是指无论如何，这些log都是优先于raw data落盘完成的。

## 事务的原子性Atomic和提交commit
两阶段提交2PC是实现事务原子性的常规手段，第一阶段执行事务，第二阶段提交确认事务。所谓的提交，即在WAL中追加一条commit record。一旦WAL中存在commit record，即认为事务已经提交。未提交事务在故障恢复时需要回滚。

> 注意
> commit record并非纯粹的log内容，它是为了支持事务而额外添加的标记。

## no-force/steal
通过WAL和2PC实现持久性和事务原子性的情况下，写raw data的时机是一个非常重要的考量因素，不同的时机选择决定了数据库的关键实现逻辑

任何情况下，总是在WAL之后才会写raw data，但相对于写commit record，有三种写raw data的时机
1. 在写commit record之前就写raw data
2. 在写commit record时
3. 在写commit record之后才写raw data。


如果我们在commit record前就写，相当于我们将raw data的更新前后信息（物理数据，或者逻辑指令）写到undo log后，马上就可以写raw data，这就是steal策略，steal什么呢？steal commit前的空闲I/O。很多情况下，transaction会耗费较长时间，其中涉及大量的raw data更新，如果都等到commit时或者之后才写，那么即便transaction执行期间的I/O很低也无法利用。在commit之前就写raw data，就相当于steal了commit前的空闲I/O。显然，**steal策略会增加系统实现的逻辑复杂性，需要undo log的支持**。

如果我们在写commit record时强制force写transaction中的raw data，可能因为频繁写小的raw data降低磁盘吞吐率，也可能因为写大量的raw data导致存储繁忙而暂时性hang住系统，所以大家一般都不会采用这种force策略，而是直接写commit record，之后才写raw data：反正更新已经记录到了redo log中，完成了持久化，不如等I/O闲的时候慢慢写，这就是no-force。**no-force策略需要redo log的支持**。

## MySQL InnoDB Crash recovery
MySQL的InnoDB引擎实现了steal和no-force策略，因此同时使用了redo和undo log。为更好的理解上述内容，本节简要介绍MySQL InnoDB的Crash Recovery过程。

### Checkpoint
类似于commit record，Checkpoint也是数据库记录在redo log中的一种标识label，其本质是一个LSN（log sequence number）值，表示**已经落盘**的最新的change。

> MySQL写data首先更新内存，此时会同步写buffer pool（an area in main memory where InnoDB caches table and index data as it is accessed）和log buffer（the memory area that holds data to be written to the log files on disk）。而log buffer一般会周期性准实时的flush到log file中（Fuzzy checkpoint，InnoDB flushes modified database pages from the buffer pool in small batches），flush完后更新log buffer的checkpoint label，并最终被写到redo log中。

### recovery过程
1. 前滚：因为no-force的原因，存在已提交但未落盘的内容，因此我们需要checks the redo logs and performs a roll-forward of the database to the present。即在redo log中比较并找到checkpoint到commit record标记之间的日志并应用，将已提交但未落盘的事务重新实施。
2. 后滚：因为steal的原因，存在未提交但已落盘的内容，因此我们需要rolls back uncommitted transactions that were present at the time of the crash，即在undo log中找到最后一个commit record到checkpoint标记之间的日志来实施回滚。
> 前滚完成后，MySQL即可打开链接接受外部新的读写请求，后滚将以后台线程的形式进行。