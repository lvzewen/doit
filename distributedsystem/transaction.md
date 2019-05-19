# 事务

## 事务的具体定义

事务提供一种机制将一个活动涉及的所有操作纳入到一个不可分割的执行单元，组成事务的所有操作只有在所有操作均能正常执行的情况下方能提交，只要其中任一操作执行失败，都将导致整个事务的回滚。简单地说，事务提供一种“要么什么都不做，要么做全套（All or Nothing）”机制。

## 数据库本地事务

### ACID

1. A:原子性(Atomicity)
一个事务(transaction)中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
2. C:一致性(Consistency)
事务的一致性指的是在一个事务执行之前和执行之后数据库都必须处于一致性状态。如果事务成功地完成，那么系统中所有变化将正确地应用，系统处于有效状态。如果在事务中出现错误，那么系统中的所有变化将自动地回滚，系统返回到原始状态。
3. I:隔离性(Isolation)
指的是在并发环境中，当不同的事务同时操纵相同的数据时，每个事务都有各自的完整数据空间。由并发事务所做的修改必须与任何其他并发事务所做的修改隔离。事务查看数据更新时，数据所处的状态要么是另一事务修改它之前的状态，要么是另一事务修改它之后的状态，事务不会查看到中间状态的数据。
4. D:持久性(Durability)
指的是只要事务成功结束，它对数据库所做的更新就必须永久保存下来。即使发生系统崩溃，重新启动数据库系统后，数据库还能恢复到事务成功结束时的状态。

### InnoDB实现原理

nnoDB是mysql的一个存储引擎，大部分人对mysql都比较熟悉，这里简单介绍一下数据库事务实现的一些基本原理，在本地事务中，服务和资源在事务的包裹下可以看做是一体的:

而事务的ACID是通过InnoDB日志和锁来保证。事务的隔离性是通过数据库锁的机制实现的，持久性通过redo log（重做日志）来实现，原子性和一致性通过Undo log来实现。UndoLog的原理很简单，为了满足事务的原子性，在操作任何数据之前，首先将数据备份到一个地方（这个存储数据备份的地方称为UndoLog）。然后进行数据的修改。如果出现了错误或者用户执行了ROLLBACK语句，系统可以利用Undo Log中的备份将数据恢复到事务开始之前的状态。
和Undo Log相反，RedoLog记录的是新数据的备份。在事务提交前，只要将RedoLog持久化即可，不需要将数据持久化。当系统崩溃时，虽然数据没有持久化，但是RedoLog已经持久化。系统可以根据RedoLog的内容，将所有数据恢复到最新的状态。
对具体实现过程有兴趣的同学可以去自行搜索扩展。

## 分布式事务

### 什么是分布式事务

分布式事务就是指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于不同的分布式系统的不同节点之上。简单的说，就是一次大的操作由不同的小操作组成，这些小的操作分布在不同的服务器上，且属于不同的应用，分布式事务需要保证这些小操作要么全部成功，要么全部失败。本质上来说，分布式事务就是为了保证不同数据库的数据一致性。

### 分布式事务产生的原因

从上面本地事务来看，我们可以看为两块，一个是service产生多个节点，另一个是resource产生多个节点。

1. service多个节点
2. resource多个节点

### 分布式事务的基础

从上面来看分布式事务是随着互联网高速发展应运而生的，这是一个必然的我们之前说过数据库的ACID四大特性，已经无法满足我们分布式事务，这个时候又有一些新的大佬提出一些新的理论:

#### CAP

CAP定理，又被叫作布鲁尔定理。对于设计分布式系统来说(不仅仅是分布式事务)的架构师来说，CAP就是你的入门理论。

1. C (一致性):对某个指定的客户端来说，读操作能返回最新的写操作。对于数据分布在不同节点上的数据上来说，如果在某个节点更新了数据，那么在其他节点如果都能读取到这个最新的数据，那么就称为强一致，如果有某个节点没有读取到，那就是分布式不一致。
2. A (可用性)：非故障的节点在合理的时间内返回合理的响应(不是错误和超时的响应)。可用性的两个关键一个是合理的时间，一个是合理的响应。合理的时间指的是请求不能无限被阻塞，应该在合理的时间给出返回。合理的响应指的是系统应该明确返回结果并且结果是正确的，这里的正确指的是比如应该返回50，而不是返回40。
3. P (分区容错性):当出现网络分区后，系统能够继续工作。打个比方，这里个集群有多台机器，有台机器网络出现了问题，但是这个集群仍然可以正常工作。

#### BASE

BASE 是 Basically Available(基本可用)、Soft state(软状态)和 Eventually consistent (最终一致性)三个短语的缩写。是对CAP中AP的一个扩展

1. 基本可用:分布式系统在出现故障时，允许损失部分可用功能，保证核心功能可用。
2. 软状态:允许系统中存在中间状态，这个状态不影响系统可用性，这里指的是CAP中的不一致。
3. 最终一致:最终一致是指经过一段时间后，所有节点数据都将会达到一致。

BASE解决了CAP中理论没有网络延迟，在BASE中用软状态和最终一致，保证了延迟后的一致性。BASE和 ACID 是相反的，它完全不同于ACID的强一致性模型，而是通过牺牲强一致性来获得可用性，并允许数据在一段时间内是不一致的，但最终达到一致状态。

### 事务的隔离级别

在事务的四大特性ACID中，要求的隔离性是一种严格意义上的隔离，也就是多个事务是串行执行的，彼此之间不会受到任何干扰。这确实能够完全保证数据的安全性，但在实际业务系统中，这种方式性能不高。因此，数据库定义了四种隔离级别，隔离级别和数据库的性能是呈反比的，隔离级别越低，数据库性能越高，而隔离级别越高，数据库性能越差。

#### 事务并发执行会出现的问题

我们先来看一下在不同的隔离级别下，数据库可能会出现的问题：
更新丢失 当有两个并发执行的事务，更新同一行数据，那么有可能一个事务会把另一个事务的更新覆盖掉。当数据库没有加任何锁操作的情况下会发生。
脏读 一个事务读到另一个尚未提交的事务中的数据。该数据可能会被回滚从而失效。如果第一个事务拿着失效的数据去处理那就发生错误了。
不可重复读 不可重复度的含义：一个事务对同一行数据读了两次，却得到了不同的结果。它具体分为如下两种情况：

1. 虚读：在事务1两次读取同一记录的过程中，事务2对该记录进行了修改，从而事务1第二次读到了不一样的记录。
2. 幻读：事务1在两次查询的过程中，事务2对该表进行了插入、删除操作，从而事务1第二次查询的结果发生了变化。

#### 数据库的四种隔离级别

Read uncommitted 读未提交
在该级别下，一个事务对一行数据修改的过程中，不允许另一个事务对该行数据进行修改，但允许另一个事务对该行数据读。
因此本级别下，不会出现更新丢失，但会出现脏读、不可重复读。
Read committed 读提交
在该级别下，未提交的写事务不允许其他事务访问该行，因此不会出现脏读；但是读取数据的事务允许其他事务的访问该行数据，因此会出现不可重复读的情况。
Repeatable read 重复读
在该级别下，读事务禁止写事务，但允许读事务，因此不会出现同一事务两次读到不同的数据的情况（不可重复读），且写事务禁止其他一切事务。
Serializable 序列化
该级别要求所有事务都必须串行执行，因此能避免一切因并发引起的问题，但效率很低。

### 分布式事务解决方案

#### 是否真的要分布式事务

上面说过出现分布式事务的两个原因，其中有个原因是因为微服务过多。我见过太多团队一个人维护几个微服务，太多团队过度设计，搞得所有人疲劳不堪，而微服务过多就会引出分布式事务，这个时候我不会建议你去采用下面任何一种方案，而是请把需要事务的微服务聚合成一个单机服务，使用数据库的本地事务。因为不论任何一种方案都会增加你系统的复杂度，这样的成本实在是太高了，千万不要因为追求某些设计，而引入不必要的成本和复杂度。

#### 2PC

说到2PC就不得不聊数据库分布式事务中的 XA Transactions。
在XA协议中分为两阶段:
第一阶段：事务管理器要求每个涉及到事务的数据库预提交(precommit)此操作，并反映是否可以提交.

第二阶段：事务协调器要求每个数据库提交数据，或者回滚数据。

缺点:

单点问题:事务管理器在整个流程中扮演的角色很关键，如果其宕机，比如在第一阶段已经完成，在第二阶段正准备提交的时候事务管理器宕机，资源管理器就会一直阻塞，导致数据库无法使用。
同步阻塞:在准备就绪之后，资源管理器中的资源一直处于阻塞，直到提交完成，释放资源。
数据不一致:两阶段提交协议虽然为分布式数据强一致性所设计，但仍然存在数据不一致性的可能，比如在第二阶段中，假设协调者发出了事务commit的通知，但是因为网络问题该通知仅被一部分参与者所收到并执行了commit操作，其余的参与者则因为没有收到通知一直处于阻塞状态，这时候就产生了数据的不一致性。

总的来说，XA协议比较简单，成本较低，但是其单点问题，以及不能支持高并发(由于同步阻塞)依然是其最大的弱点。

#### TCC

TCC事务机制相比于上面介绍的XA，解决了其几个缺点:
1.解决了协调者单点，由主业务方发起并完成这个业务活动。业务活动管理器也变成多点，引入集群。
2.同步阻塞:引入超时，超时后进行补偿，并且不会锁定整个资源，将资源转换为业务逻辑形式，粒度变小。
3.数据一致性，有了补偿机制之后，由业务活动管理器控制一致性

对于TCC的解释:
Try阶段：尝试执行,完成所有业务检查（一致性）,预留必须业务资源（准隔离性）
Confirm阶段：确认执行真正执行业务，不作任何业务检查，只使用Try阶段预留的业务资源，Confirm操作满足幂等性。要求具备幂等设计，Confirm失败后需要进行重试。
Cancel阶段：取消执行，释放Try阶段预留的业务资源
Cancel操作满足幂等性Cancel阶段的异常和Confirm阶段异常处理方案基本上一致。

#### 本地消息表

此方案的核心是将需要分布式处理的任务通过消息日志的方式来异步执行。消息日志可以存储到本地文本、数据库或消息队列，再通过业务规则自动或人工发起重试。人工重试更多的是应用于支付场景，通过对账系统对事后问题的处理。

#### MQ事务

基本流程如下:
第一阶段Prepared消息，会拿到消息的地址。
第二阶段执行本地事务。
第三阶段通过第一阶段拿到的地址去访问消息，并修改状态。消息接受者就能使用这个消息。
如果确认消息失败，在RocketMq Broker中提供了定时扫描没有更新状态的消息，如果有消息没有得到确认，会向消息发送者发送消息，来判断是否提交，在rocketmq中是以listener的形式给发送者，用来处理。

#### Saga事务

其核心思想是将长事务拆分为多个本地短事务，由Saga事务协调器协调，如果正常结束那就正常完成，如果某个步骤失败，则根据相反顺序一次调用补偿操作。
Saga的组成：
每个Saga由一系列sub-transaction Ti 组成
每个Ti 都有对应的补偿动作Ci，补偿动作用于撤销Ti造成的结果,这里的每个T，都是一个本地事务。
可以看到，和TCC相比，Saga没有“预留 try”动作，它的Ti就是直接提交到库。
Saga的执行顺序有两种：
T1, T2, T3, ..., Tn
T1, T2, ..., Tj, Cj,..., C2, C1，其中0 < j < n
Saga定义了两种恢复策略：
向后恢复，即上面提到的第二种执行顺序，其中j是发生错误的sub-transaction，这种做法的效果是撤销掉之前所有成功的sub-transation，使得整个Saga的执行结果撤销。
向前恢复，适用于必须要成功的场景，执行顺序是类似于这样的：T1, T2, ..., Tj(失败), Tj(重试),..., Tn，其中j是发生错误的sub-transaction。该情况下不需要Ci。
这里要注意的是，在saga模式中不能保证隔离性，因为没有锁住资源，其他事务依然可以覆盖或者影响当前事务。

### 分布式事务协议

#### 两阶段提交协议 2PC

分布式系统的一个难点是如何保证架构下多个节点在进行事务性操作的时候保持一致性。为实现这个目的，二阶段提交算法的成立基于以下假设：

1. 该分布式系统中，存在一个节点作为协调者(Coordinator)，其他节点作为参与者(Cohorts)。且节点之间可以进行网络通信。
2. 所有节点都采用预写式日志，且日志被写入后即被保持在可靠的存储设备上，即使节点损坏不会导致日志数据的消失。
3. 所有节点不会永久性损坏，即使损坏后仍然可以恢复

##### 第一阶段（投票阶段）：

协调者节点向所有参与者节点询问是否可以执行提交操作(vote)，并开始等待各参与者节点的响应。
参与者节点执行询问发起为止的所有事务操作，并将Undo信息和Redo信息写入日志。（注意：若成功这里其实每个参与者已经执行了事务操作）
各参与者节点响应协调者节点发起的询问。如果参与者节点的事务操作实际执行成功，则它返回一个"同意"消息；如果参与者节点的事务操作实际执行失败，则它返回一个"中止"消息。

##### 第二阶段（提交执行阶段）：

当协调者节点从所有参与者节点获得的相应消息都为"同意"时：

协调者节点向所有参与者节点发出"正式提交(commit)"的请求。
参与者节点正式完成操作，并释放在整个事务期间内占用的资源。
参与者节点向协调者节点发送"完成"消息。
协调者节点受到所有参与者节点反馈的"完成"消息后，完成事务。

如果任一参与者节点在第一阶段返回的响应消息为"中止"，或者 协调者节点在第一阶段的询问超时之前无法获取所有参与者节点的响应消息时：

协调者节点向所有参与者节点发出"回滚操作(rollback)"的请求。
参与者节点利用之前写入的Undo信息执行回滚，并释放在整个事务期间内占用的资源。
参与者节点向协调者节点发送"回滚完成"消息。
协调者节点受到所有参与者节点反馈的"回滚完成"消息后，取消事务。

不管最后结果如何，第二阶段都会结束当前事务。
二阶段提交看起来确实能够提供原子性的操作，但是不幸的事，二阶段提交还是有几个缺点的：

执行过程中，所有参与节点都是事务阻塞型的。当参与者占有公共资源时，其他第三方节点访问公共资源不得不处于阻塞状态。
参与者发生故障。协调者需要给每个参与者额外指定超时机制，超时后整个事务失败。（没有多少容错机制）
协调者发生故障。参与者会一直阻塞下去。需要额外的备机进行容错。（这个可以依赖后面要讲的Paxos协议实现HA）
二阶段无法解决的问题：协调者再发出commit消息之后宕机，而唯一接收到这条消息的参与者同时也宕机了。那么即使协调者通过选举协议产生了新的协调者，这条事务的状态也是不确定的，没人知道事务是否被已经提交。

#### 三阶段提交协议 3PC

与两阶段提交不同的是，三阶段提交有两个改动点。
引入超时机制。同时在协调者和参与者中都引入超时机制。
在第一阶段和第二阶段中插入一个准备阶段。保证了在最后提交阶段之前各参与节点的状态是一致的。

也就是说，除了引入超时机制之外，3PC把2PC的准备阶段再次一分为二，这样三阶段提交就有CanCommit、PreCommit、DoCommit三个阶段。

1. CanCommit阶段
3PC的CanCommit阶段其实和2PC的准备阶段很像。协调者向参与者发送commit请求，参与者如果可以提交就返回Yes响应，否则返回No响应。

事务询问
协调者向参与者发送CanCommit请求。询问是否可以执行事务提交操作。然后开始等待参与者的响应。
响应反馈
参与者接到CanCommit请求之后，正常情况下，如果其自身认为可以顺利执行事务，则返回Yes响应，并进入预备状态。否则反馈No
2. PreCommit阶段
协调者根据参与者的反应情况来决定是否可以记性事务的PreCommit操作。根据响应情况，有以下两种可能。
假如协调者从所有的参与者获得的反馈都是Yes响应，那么就会执行事务的预执行。
发送预提交请求 协调者向参与者发送PreCommit请求，并进入Prepared阶段。
事务预提交 参与者接收到PreCommit请求后，会执行事务操作，并将undo和redo信息记录到事务日志中。
响应反馈 如果参与者成功的执行了事务操作，则返回ACK响应，同时开始等待最终指令。
假如有任何一个参与者向协调者发送了No响应，或者等待超时之后，协调者都没有接到参与者的响应，那么就执行事务的中断。
发送中断请求 协调者向所有参与者发送abort请求。
中断事务 参与者收到来自协调者的abort请求之后（或超时之后，仍未收到协调者的请求），执行事务的中断。
3. doCommit阶段
该阶段进行真正的事务提交，也可以分为以下两种情况。
3.1 执行提交
发送提交请求 协调接收到参与者发送的ACK响应，那么他将从预提交状态进入到提交状态。并向所有参与者发送doCommit请求。
事务提交 参与者接收到doCommit请求之后，执行正式的事务提交。并在完成事务提交之后释放所有事务资源。
响应反馈 事务提交完之后，向协调者发送Ack响应。
完成事务 协调者接收到所有参与者的ack响应之后，完成事务。
3.2 中断事务
协调者没有接收到参与者发送的ACK响应（可能是接受者发送的不是ACK响应，也可能响应超时），那么就会执行中断事务。
发送中断请求 协调者向所有参与者发送abort请求
事务回滚 参与者接收到abort请求之后，利用其在阶段二记录的undo信息来执行事务的回滚操作，并在完成回滚之后释放所有的事务资源。
反馈结果 参与者完成事务回滚之后，向协调者发送ACK消息
中断事务 协调者接收到参与者反馈的ACK消息之后，执行事务的中断。

### 分布式事务的解决方案

分布式事务的解决方案有如下几种:

1. 全局消息
2. 基于可靠消息服务的分布式事务
3. TCC
4. 最大努力通知

`https://juejin.im/post/5aa3c7736fb9a028bb189bca`

超时询问机制
Rollback
重试机制+定期校对