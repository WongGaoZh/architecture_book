# 本地事务和全局事务

## CAP理论(网上的我就不解释了.我用大白话解释一下自己的理解)

``` text
C : 一致性 : 在很短的时间里面,数据也要保证一致性,A改完后,B要在下个逻辑里面看到A改动的数据.
A : 可用性 : 在很短的时间里面,数据保证可用性, A改完后,B在下个逻辑看不到A改动的数据,但是不影响最终的业务
P : 分区容错性 : 有一个节点连接中断, 整个集群能依旧对外提供服务
```

## ACID理论
``` text
原子性（atomicity）：一个事务中的所有操作，不可分割，要么全部成功，要么全部失败；
一致性（consistency）：一个事务执行前与执行后数据的完整性必须保持一致；
隔离性（isolation）：一个事务的执行，不能被其他事务干扰，多并发时事务之间要相互隔离；
持久性（durability）：一个事务一旦被提交，它对数据库中数据的改变是永久性的。
```



## BASE理论

``` text
基本可用（Basically Available）：分布式系统在出现故障时，保证核心可用，允许损失部分可用性。（响应时间上的损失、功能上的损失）
软状态（Soft State）：系统中的数据允许存在中间状态，中间状态不影响系统的整体可用性。（支付中、处理中等）
最终一致性（Eventually Consistent）：系统中的数据不可一直处于软状态，必须在有时间期限，在期限过后应当保证数据的一致性。（支付中变为支付成功）
```



## 本地事务 

适用于 单体服务使用单个数据源的场景, 直接依赖于数据源本身提供的事务能力来工作的
> 理论依据 : 在 20 世纪 90 年代，IBM Almaden 研究院总结了研发原型数据库系统“IBM System R”的经验，发表了 ARIES 理论中最主要的三篇论文，其中《ARIES: A Transaction Recovery Method Supporting Fine-Granularity Locking and Partial Rollbacks Using Write-Ahead Logging》着重解决了 ACID 的其中两个属性：原子性（A）和持久性（D）在算法层面上应当如何实现。而另一篇《ARIES/KVL: A Key-Value Locking Method for Concurrency Control of Multiaction Transactions Operating on B-Tree Indexes》则是现代数据库隔离性（I）奠基式的文章，下面，我们先从原子性和持久性说起。

这里可以访问我之前的文章 :[mysql事务隔离级别](https://wonggaozh.github.io/Blog/#/zh-database/103_mysql%E4%BA%8B%E5%8A%A1%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB)

这里简单的对这个文章进行一个总结 : 
``` text
实现原子性和持久性: 当一个记录用追加文件的形式先记录到磁盘里面,当日志记录安全落盘,才会表示当前事务已经完成持久化.

问题 : 
这样会出现一个问题,就是事务提交之前都不能修改磁盘的数据,这对提升数据库的性能十分不利,为了解决这个问题,ARIES 提出了“Write-Ahead Logging”的日志改进方案

解决办法 : 

使用undo log(解决原子性) 和 redo log (解决持久性)

### 数据库的隔离级别

其实不同隔离级别以及幻读、不可重复读、脏读等问题都只是表面现象，是各种锁在不同加锁时间上组合应用所产生的结果，以锁为手段来实现隔离性才是数据库表现出不同隔离级别的根本原因。

同时辅助于mvcc版本控制机制来解决重复读 
行锁,间隙锁,next-lock锁来解决幻读问题.
```

## 全局事务 : 
全局事务被限定为一种适用于单个服务使用多个数据源场景的事务解决方案。

### XA规范(eXtended Architecture)

X/Open组织（现在的Open Group）定义了一套DTP（Distributed Transaction Processing）分布式事务处理模型，主要包含以下四部分：

其核心内容是定义了全局的事务管理器（Transaction Manager，用于协调全局事务）和局部的资源管理器（Resource Manager，用于驱动本地事务）之间的通信接口。XA 接口是双向的，能在一个事务管理器和多个资源管理器（Resource Manager）之间形成通信桥梁，通过协调多个数据源的一致动作，实现全局事务的统一提交或者统一回滚，

### JTA( Java Transaction API)

不过，XA 并不是 Java 的技术规范（XA 提出那时还没有 Java），而是一套语言无关的通用规范，基于 XA 模式在 Java 语言中的实现了全局事务处理的标准，这也就是我们现在所熟知的 JTA。JTA 最主要的两个接口是：

事务管理器的接口：javax.transaction.TransactionManager。这套接口是给 Java EE 服务器提供容器事务（由容器自动负责事务管理）使用的，还提供了另外一套javax.transaction.UserTransaction接口，用于通过程序代码手动开启、提交和回滚事务。
满足 XA 规范的资源定义接口：javax.transaction.xa.XAResource

### XA的2PC流程
XA 将事务提交拆分成为两阶段过程：

准备阶段：又叫作投票阶段，在这一阶段，协调者询问事务的所有参与者是否准备好提交，参与者如果已经准备好提交则回复 Prepared，否则回复 Non-Prepared

提交阶段：又叫作执行阶段，协调者如果在上一阶段收到所有事务参与者回复的 Prepared 消息，则先自己在本地持久化事务状态为 Commit，在此操作完成后向所有参与者发送 Commit 指令，所有参与者立即执行提交操作,否则，任意一个参与者回复了 Non-Prepared 消息，或任意一个参与者超时未回复，协调者将自己的事务状态持久化为 Abort 之后，向所有参与者发送 Abort 指令，参与者立即执行回滚操作。

> 2PC的问题 
> 1. 单点问题： 如果协调者一直没有恢复，没有正常发送 Commit 或者 Rollback 的指令，那所有参与者都必须一直等待。
> 2. 准备阶段的性能问题: 两段提交过程中，所有参与者相当于被绑定成为一个统一调度的整体，期间要经过两次远程服务调用，三次数据持久化（准备阶段写重做日志，协调者做状态持久化，提交阶段在日志写入 Commit Record），整个过程将持续到参与者集群中最慢的那一个处理操作结束为止，这决定了两段式提交的性能通常都较差。
> 所以提供了3PC的流程

### XA的3PC流程

将准备阶段拆分 :
CanCommit(询问阶段)、PreCommit(准备阶段) 

提交阶段： DoCommit 阶段

将准备阶段一分为二的理由是这个阶段是重负载的操作，一旦协调者发出开始准备的消息，每个参与者都将马上开始写重做日志，它们所涉及的数据资源即被锁住，如果此时某一个参与者宣告无法完成提交，相当于大家都白做了一轮无用功。所以，增加一轮询问阶段，如果都得到了正面的响应，那事务能够成功提交的把握就比较大了，这也意味着因某个参与者提交时发生崩溃而导致大家全部回滚的风险相对变小。
**`因此，在事务需要回滚的场景中，三段式的性能通常是要比两段式好很多的，但在事务能够正常提交的场景中，两者的性能都依然很差，甚至三段式因为多了一次询问，还要稍微更差一些。`**



