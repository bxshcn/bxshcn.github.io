---
layout: post
title:  "分布式一致性"
comments: true
categories: 分布式
---

## concurrent programming
并发可以有效提高系统效率和容量。但实现一个并发系统需要考虑很多问题，其中最基本的是：
1. 系统中的每个对象在任意时刻应该是correct的，这就是系统的safety属性
2. 系统尽可能的保持活动以提高效率，这就是系统的liveness属性

我们需要同时考虑这两个方面，但它们却是互斥矛盾的：因为共享内存和存储的存在，我们不得不在多个进程读写关键区域时，使用某种机制（比如锁）来避免不确定的行为，因此而导致的阻塞和等待，虽然保证了系统的safety属性，但会降低系统的liveness；而一个总是为了liveness而随意操作的系统会失去safety属性，从而最终失去liveness。

所以人们实际只能妥协，在这两者中寻找平衡。

## consistency model
consistency本质上是correctness condition。一个系统的正确性判定条件会因场景不同而不同，有的要求高，有点要求低。因此我们可以将consistency作为对系统safety满足程度的度量标准。

最安全的程度是serialization，就是没有并发，所有活动行为全部串行。但为了提高性能，又要允许并发。为了在并发情况下，保证系统在某种程度上的正确性，人们提出了很多consistency modal。

consistency modal实际上是人和系统之间的规则[约定](https://en.wikipedia.org/wiki/Consistency_model)，如果系统遵循这些约定来实施行为，那么就能保证数据状态的safety。具体实践上，有两种基本的策略：
1. 由程序员来保证一致性模型的实现，比如使用lock，semaphore，tunnel等机制来实现并发编程
2. 在系统中内化一致性模型，比如提供atomic register，程序语言提供Linearizable data structure，或者类似future，promise，defer等特性

> 注意
> 
> 一致性模型只是说明了什么样的行为序列是correctness的，但并没有明确唯一的行为序列，因为可能有多个行为序列都满足该一致性模型的要求。
> 
> 但一致性模型的**具体实现**都会将一个concurrent operations转换成一个**唯一确定**的sequential operations。
>

### linearizablity
herlihy在1990年给出了linearizablity，它是利用并发对象的抽象数据类型语义，来实现的一种并发正确性条件。即只要共享数据的类型是linearable，那么基于此而构造的系统就是linearizablity。herlihy证明了linearizablity具有local和non-blocking两种属性，这使得构造和验证一个满足linearizablity的系统变得简单易行。

所谓local property，是指a system is linearizable if each individual object is linearizable. 即linearizability的正确性**直接构筑于object之上**。这与sequential consistency不同，sequential consistency的行为约束是施加在单个进程上。比如我们说一个[atomic register](https://www.amazon.com/dp/0123705916)是linearable的，因此基于该数据对象进行的所有操作天然是linearizability的。另外一个例子就是大名鼎鼎的CAS，基于CAS操作的寄存器也是linearable的。

所谓non-blocking property，是指processes invoking totally-defined operations are never forced to wait.因为任何操作的同步都建立在real-time的操作上，因此每个操作总有一个即时生效的点，从而我们可以将任意操作行为视为原子的，因而无需等待：

![linearizability](/assets/2021-07-29-consistency/linearizability.webp)

写操作虽然w(b)开始，到w'(b)才结束，但实际上在b这个点生效。第一次读操作也有一定的时间跨越，但因为读的那一刻b还未真正写入生效，因此仍然读到的是a，第二次才读到b。

显然，linearizablity将正确性直接构建在object上，并引入real-time （从而支持将任意操作行为原子化），linearizability不仅保留了单进程上操作的先后序列，还保留了多进程之间在共享数据操作上的先后序列。

linearizability = sequential consistency + (atomic operation/real-time order)，原子操作是指所有操作即便有调用和结束时点，但更关键的是他们都有唯一的一个生效时点，就像这个操作在这个生效时点瞬时发生一样。real-time order是说任意两个操作的顺序是依据real-time clock来区分的。

### sequential consistency
Sequential consistency是Lamport在1979年提出的一种模型，它要求满足该一致性模型的系统，在转化后的活动序列能保证系统事件的”happen before“关系，假设记为$\rightarrow$其定义如下：
1. 如果a和b隶属于同一个process，并且a先于b发生，那么有$a \rightarrow b$
2. 相关依赖性：如果a表示一个进程发送消息这一event，b表示另外一个进程接收这个消息这一event，那么有$a \rightarrow b$
3. 传递性: 如果有$a \rightarrow b$，且$b \rightarrow c$，那么必有$a \rightarrow c$

实际应用中，我们往往并不考虑其中第2点，因为**两个进程发送消息是实现同步，约束操作序列的具体方法和手段，是任何时候都需要满足的正确性，非独sequential consistency所独有**，因此sequential consistency的核心就是保证转换后的序列在单进程上的投影保留原来的顺序，这也是很多文章在提到sequential consistency并不提及上述2的原因.

这意味着相对linearizabiliy而言，sequential consistency可以**在不考虑real-time**的情况下，任意错位两个进程的操作序列：
![sequential](/assets/2021-07-29-consistency/sequential-history.webp)

很多情况下，sequential consistency本身就可以保证足够的正确性，许多缓存系统都有类似顺序一致性的行为。比如我在 Twitter上发推（我相当于是一个进程），每篇推文渗透到缓存系统的各个层需要耗费一定的时间，不同的用户将会在不同的时间看到这些推文，有些用户可能立刻就看到了我的推文，但有些用户需要很久才能看到，但不管早晚，但是每个用户（即多个其他进程）最终看到的推文的顺序都与我发推的顺序一致。

有的时候，我们会特意设计这样的业务场景，通过合同约定将原本可能要求linearizability的业务，降格为sequential consistency即可满足的业务，比如约定当一个用户上传视频到YouTube后会立刻得到访问该视频的web页面地址，但并不保证其他用户可以立刻播放该视频。

也就是说，如果我们不在意两个并发进程操作过程中的real-time特性，那么sequential consistency就是一个很好的正确性方案，这可以让我们得到更高的并发量和系统吞吐量，也即更好的liveness。

当然，**如果系统软件提供的sequential consistency不能保证我们业务系统具体场景的正确性，就需要程序员设计特别的同步逻辑，来实现更高的正确性要求。**

### causal consistency
causal consistency比sequential consistency约束更宽松，它只要求转换后的操作序列保留原始的因果关系操作顺序。

### serializability
serializability是一个比较特殊的正确性条件，它主要应用在数据库领域，与事务transaction和隔离性isolation相关。或者说，serializability是transaction consistency，而其他都是non-transaction consistency。我们通常所说的都是strict serializability。

strict serializability指两个事务相继执行，without overlap in time, and thus completely isolated from each other: No concurrent access by any two transactions to the same data is possible，如果完全按照这个正确性要求来重排事务操作序列，将导致系统的效率显著降低。

> 补充
> 
> 人们将strict serializability分为两类，view-serializability和conflict-serializability。前者对应上面的一般性定义，但conflict-serializability相对特殊，即只要在linearizability的基础上，保证两个overlapping事务中的冲突操作（包括读-写，即一个事务中，写-读，或者写-写，但某些具体操作并不会产生冲突，如果这两个操作是可交换的commutative）的时序(time order)。
> 
> **Only precedence (time order) in pairs of conflicting (non-commutative) operations is important**
> 
>> Two operations are conflicting if they are of different transactions, upon the same datum (data item), and at least one of them is write. 
>> 
>> The transaction of the second operation in the pair is said to be in conflict with the transaction of the first operation. 
>> 
>> 为了施加conflict-serializability，可以采用一些具体的技术，比如：
>> 1. Precedence graph(or Serializability graph, Conflict graph) cycle elimination
>> 2. Two-phase locking (2PL)
>> 3. Timestamp ordering (TO)
>> 4. Serializable snapshot isolation (SI)
>> 
>> 根据施加上述冲突解决策略的时机的不同，可以分为Optimistic和pessimistic两类，前者假设没有冲突，直到最后事务结束时才检测是否存在冲突，如果有则回滚；后者一旦发现冲突就采取阻塞同步机制来保证正确性。

但既然我们是针对的并发事务，并希望能获取更好的性能，我们选择relax serializability的要求。比如ACID数据库的serializable级别实际上是Snaphshot Isolation，或者BASE数据库的eventual consistency，但无论如何，只有我们能保证最终系统的正确性，这种relax放宽要求才有意义。

### eventual consistency
最终一致性is also called optimistic replication(aslo called lazy replication) is widely deployed in distributed systems. 其本质就是多份拷贝的问题，其基本思想是在分布式环境中取得更大的系统活性和效率，临时性的舍弃全局safety保证。

最终一致性实质上由如下局部一致性来保证，换句话说，针对分布式系统中的任意单个节点，我们必须保证下面的正确性要求：
1. causal consistency（保证因果关系决定的操作顺序）
2. Read-your-writes consistency
3. session consistency（即在单个session中保证read-your-writes）,是2在session上下文中的表现。
4. Monotonic read consistency（单调读，后续不可能读到比当前已经读到的值更旧的值）If a process has seen a particular value for the object, any subsequent accesses will never return any previous values.
5. Monotonic write consistency（单调写）In this case the system guarantees to serialize the writes by the same process. 

除上述局部一致性保证外，系统还必须统一所有节点的数据状态，采用的机制就是复制。In order to ensure replica convergence, a system must reconcile differences between multiple copies of distributed data. 这种reconcililation实际上就是get consensus的过程，包括两个核心要素：
1. exchanging versions or updates of data
2. choosing an appropriate final state

简言之，在分布式场景下，通过同步来实现正确性语义往往代价甚高，一种变通措施是临时舍弃实时的全局正确性，通过复制这种批量的交换状态信息来达成最终的全局正确性。it is in fact implemented in situations in which having a synchronization protocol would be too costly, and a fortuitous exchange of information between replicas can be good enough. 

## 一致性的实现
除了系统软件内置的一致性实现，我们还需要在业务代码级别来实现进程的通讯，以取得正确的结果。

首先是语言级别提供的机制，对程序员透明。比如defer, promise，future等。

其次需要程序员显式的实现，具体可分为两种模式：
1. Shared memory communication，即基于共享内存或存储数据实现并发组件的通讯，this style of concurrent programming usually needs the use of some form of locking (e.g., mutexes, semaphores, or monitors) to coordinate between threads。主要的语言代表是java和C#
2. Message passing communication,基于交换消息来实现并发组件的通讯，Message-passing concurrency tends to be far easier to reason about than shared-memory concurrency, and is typically considered a more robust form of concurrent programming.主要语言代表比如go，scala，Erlang

**一般情况下，一个系统都有多种场景，每种场景的正确性要求不同，因此我们实际上会实现多种一致性模型，比如有时候我们在编程时遵循sequential consistency即可，但对于共享数据的并发操作的正确性，我们往往依赖于CAS以及相关的锁机制，即要遵循于linearizability。**

## 一致性的代价

任何情况下，要获得系统（状态）的正确性都需要耗费一定的代价，在并发场景下，这个代价尤其大，而且更为困难和复杂。为了降低编程的复杂度，提高并发的效率（liveness活性），人们想了很多办法，其基本思路仍然是分治，即在硬件和系统软件层面支持一定的一致性规则，在应用软件层面补足实现系统正确性的剩余要求。

从另外一个角度看。实现一致性模型的系统（硬件或者系统软件）就是将随机并发的操作序列转换成一个符合一定规则的操作序列。如果我们**将这个操作序列称为一个history**，则符合要求的histories往往有很多个。一致性模型实现的目的，就是通过排除不符合要求的histories，让所得histories更少，从而方便编程选出一个正确的history，当然，正确的histories也许仍不唯一，但编程选出的这个history因为满足一致性模型，与人的认知一致，因而是一个**可行解**。

strong一致性模型比weaker一致性模型会得到更少的可选histories，且每个hisotory会**更有序**。显然，施加有序性需要协调coordinating，越多的有序性，就需要越多的协调和同步。需要排除的histories越多，结果集越有序，系统（硬件或系统软件）中的各种组件就要进行更多的沟通协作。所以如果场景允许，我们都会考虑降低一致性要求以取得更高的系统效率。


参考：
- [1] aphyr. [Strong consistency models](https://aphyr.com/posts/313-strong-consistency-models)
- [2] [Serializability](https://en.wikipedia.org/wiki/Serializability)
- [3] [Concurrency_control](https://en.wikipedia.org/wiki/Concurrency_control)
- [4] [consistency](http://jepsen.io/consistency)
- [5] [safety-and-liveness](https://blog.alexcu.me/safety-and-liveness/index.html)
- [6] [eventually_consistent](https://www.allthingsdistributed.com/2007/12/eventually_consistent.html)
- [7] LAMPORT, L. 1978. Time, Clocks, and the Ordering of Events in a Distributed System
- [8] M. Herlihy, J. Wing, 1990. Linearizability: A Correctness Condition for Concurrent Objects