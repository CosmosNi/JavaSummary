# Zookeeper

###### 1.1  Zookeeper简介

<!--大部分内容来自（从Paxos到Zookeeper分布式一致性原理与实践）此书-->

Zookeeper是一个开源的分布式协调服务，是一个典型的分布式数据一致性的解决方案，分布式应用程序可以基于此实现诸如数据发布/订阅，负载均衡，命名服务，分布式协调/通知，集群管理，Matser选举，分布式锁和分布式队列等功能。



###### 1.2 Zk特性

- 顺序一致性：从同一个客户端发起的事务请求，最终将会严格地按照其发起顺序被应用到Zk中去。
- 原子性：所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，也就是说，要么整个集群的所有机器都成功应用了某个事务，要么都没有应用。
- 单一视图：无论客户端连接是哪个zk服务器，其看到的服务端数据模型都是一致的。
- 可靠性：一旦服务端成功应用了一个事务，并完成对客户端的响应，那么该事务所引起的服务端状态变更将会被一直保留下来，除非有另外一个事务对其进行了变更。
- 实时性：在一定的时间段内，客户端最终一定能够从服务端上读取到最新的数据状态。



###### 1.3 zk集群

zk集群中有三种角色，Leader，Follower，Observer。zk集群中的所有机器通过一个Leader选举过程来选定一台名为”Leader“的机器，Leader服务器为客户端提供读和写服务。Follower，Observer都能提供读服务，区别在于Observer不参加写操作的”过半写成功“策略。因此Observer可以在不影响写性能的情况下提升集群的读性能。



###### 1.4 会话

客户端和服务器之间建立一个TCP长连接，通过这个连接，客户端可以通过心跳检测与服务器保持有效的会话，也能够向服务器端发送请求并接受响应，同时接收来自服务器的Watch事件通知。Session的sessionTimeout值用来设置一个客户端会话的超时时间。在服务器发生故障时，只要在sessionTimeout规定的时间内能够重新连接到集群中的任意一台服务器，那么之前创建的会话仍然有效。



###### 1.5 数据节点（Znode）

zk将所有数据都存在内存中，数据模型是一颗树，有斜杠（/）进行分割路径。

在zk中，ZNode分为持久节点和临时节点。所谓持久节点是指一旦这个ZNode被创建了，除非主动进行ZNode的移除操作，否则这个ZNode将一直保存在zk上。

临时节点，生命周期与客户端会话绑定，一旦客户端会话失效，那么这个客户端创建的所有临时节点都会被移除。zk还允许用户为每个节点添加一个特殊的属性：SEQUENTIAL。一旦节点被标记这个属性，那么这个节点在创建的时候，zk会自动在其节点后面追加一个整形数字，这个整形数字是一个由父节点维护的自增数字。



###### 1.6 Watch

zk允许用户在指定节点上注册一些watch，并且在一些特定事件触发的时候，zk会将事件通知到感兴趣的客户端上去，该机制是zk实现分布式协调服务的重要特性。





###### 1.7 权限

- CREATE：创建子节点的权限。
- READ：获取节点数据好子节点列表的权限。
- WRITE：更新节点数据的权限。
- DELETE：删除子节点的权限。
- ADMIN：设置节点的ACL权限
- 

###### 1.8 ZAB协议

定义：所有事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器称为Leader服务器，而余下的其他服务器都是Follower服务器。LEADER服务器负责将一个客户端请求转换成一个事务Proposal（提案），并将该Proposal分发个集群中所有Follower服务器。之后Leader服务器需要等待所有Follower的反馈，一旦超过半数的Follower服务器进行正确的反馈后，那么Leader就会再次向所有的Follower服务器分发Commit消息，要求将Proposal进行提交。

- 崩溃恢复：当Leader服务器出现故障的时候，ZAB协议就会进入恢复模式并选举新的Leader服务器。当选举产生新的Leader服务器后，同时集群中已经有过半机器与该Leader服务器完成了状态同步后，ZAB协议就会退出恢复模式。当遵守ZAB协议的服务器新加入时，会自动进入恢复模式，与Leader服务器进行数据同步。

  ZAB协议需要确保那些已经在Leader服务器上提交的事务最终被所有服务器都提交。

  ZAB协议需要确保丢弃那些只在Leader服务器上被提出的事务。

  为了满足以上两种情况，ZAB协议设计了如下的Leader选举算法：保证新选举出来的Leader服务器拥有集群中所有机器最高编号（即ZXID最大）的事务Proposal，那么就可以保证新选举出来的Leader一定具有所有已经提交的提案。

  

- 消息广播：Leader服务器在接收到客户端的事务请求后，会生成对应事务提案并发起一轮广播协议；而如果集群中其他机器接收到客户端的事务请求，那么这些非Leader服务器会首先将这个事务请求转发给Leader服务器。在整个消息广播过程中，Leader服务器会为每个事务请求生成对应的Proposal来进行广播,并且在广播Proposal之前，Leader会为这个事务Proposal分配一个全局单调递增的唯一ID，我们称为事务ID（即ZXID）。由于ZAB协议需要保证每一个消息严格的因果关系，因此必须将每一个事务Proposal按照其ZXID的先后顺序进行排序与处理。

  在广播过程中，Leader服务器会为每一个Follower服务器分配一个单独的队列，然后将需要广播的事务Proposal依次放入到这个队列中去，并且根据FIFO策略进行消息发送。每一个Follower服务器接收到Proposal后，都会首先将其以事务日志的形式写入到本地磁盘中去，并且在成功写入后反馈给Leader服务器一个Ack响应。当Leader服务器收到超过半数Follower的Ack响应后，就会广播一个Commit消息给所有的Follower服务器以通知其进行事务提交，同时Leader也会完成对事务的提交。

  ZXID是一个64位的数字，其中低32位可以看做一个简单的单调递增的计数器，Leader在产生一个新的事务Proposal的时候，都会对计数器加1。而高32位代表Leader周期的epoch的编号。每当选举产生一个新的Leader服务器，都会从这个Leader服务器上取出其本地日志最大事务的Proposal的ZXID，并从该ZXID解析出对应的epoch值，然后对其进行加1操作，并以此作为新的epoch，并将低32位位置0来开始生成新的ZXID。



###### 1.8  ZAB与Paxos算法的区别联系

相同点：

1. 两个都存在一个类似Leader进程的角色，由其负责协调多个follower进程的运行。
2. Leader进程都会等待超过半数的Follower做出正确的反馈后，才会将一个提案进行提交
3. 在ZAB协议中，每个Proposal中都包含一个epoch值，用来代表当前的Leader周期，在Proxs算法中，同样存在这样一个标识，Ballot。



不同点：

在Paxos算法中，一个新选举的主进程会进行两个阶段的工作。第一阶段被称为读阶段，在这个阶段中，这个新的主进程会通过和其它进程进行通信的方式来收集上一个主进程提出的方案，并将它们提交。第二阶段被称为写阶段，在这个阶段中，当前主进程开始提出它自己的提案。

ZAB协议在paxos的基础上，增加了一个同步的阶段。再同步阶段之前，ZAB协议也存在一个和Paxos算法中的读阶段类似的过程，称为发现阶段。在同步阶段，新的Leader会确保存在过半的Follower已经提交了之前Leader周期中的所有事务Proposal。这一同步阶段的引入，能够有效地保证Leader在新的周期提出事务Proposal之前，所有的进程都已经完成了对之前所有事务Proposal的提交。一旦完成同步阶段，那么ZAB就会执行和Paxos算法类似的些阶段。

ZAB主要用于构建一个高可用的分布式数据主备系统。

Paxos主要用于构建一个分布式的一致性状态机系统。





###### 1.9 zk的典型应用场景

1. 发布/订阅
2. 负载均衡
3. 命名服务
4. 分布式协调/通知
5. 集群管理
6. Master选举
7. 分布式锁
8. 分布式队列



###### 2.0 Znode类型

持久节点：数据节点从创建后一直存在于zk服务器上，直到有删除操作。

持久顺序节点：每一个父节点都会为它的第一级节点维护一份顺序，用于记录每个

子节点的先后顺序。

临时节点：生命周期和客户端的会话绑在一起。客户端会话失效，节点自动清理。

临时顺序节点：



###### 2.1 状态STAT

数据节点的状态信息

| 状态属性       | 说明                                                 |
| -------------- | ---------------------------------------------------- |
| czxid          | 数据节点创建时的事务id                               |
| mzxid          | 节点最后一次被更新时的事务id                         |
| ctime          | 创建时间                                             |
| mtime          | 更新时间                                             |
| version        | 数据节点的版本号                                     |
| cversion       | 子节点的版本号                                       |
| aversion       | 节点的ACL版本号                                      |
| ephemeralowner | 创建该临时节点的会话的sessionId。如果是持久节点，则0 |
| dataLength     | 数据内容的长度                                       |
| numChildren    | 当前节点的子节点个数                                 |
| pzxid          | 该节点的子节点列表最后一次被修改时的事务id           |



###### 2.2 watcher通知状态

| KeeperState        | EventType           | 触发条件                                                     | 说明                        |
| :----------------- | ------------------- | ------------------------------------------------------------ | --------------------------- |
| SyncConnected（3） | None（-1）          | 客户端与服务器成功建立会话                                   | 与服务器处于连接状态        |
| SyncConnected（3） | NodeCreate          | Watcher监听的对应数据节点被创建                              |                             |
| SyncConnected（3） | NodeDeleted         | watcher监听的对应数据节点被删除                              |                             |
| SyncConnected（3） | NodeDataChanged     | Watcher监听的对应数据节点的数据内容发生变更                  |                             |
| SyncConnected（3） | NodeChildrenChanged | Watcher监听的对应数据节点的数据内容发生变更                  |                             |
| Disconnected（0）  | None（-1）          | 客户端和服务器断开连接                                       |                             |
| Expired（-112）    | None（-1）          | 会话超时                                                     | SessionExpiredException异常 |
| AuthFailed（4）    | None（-1）          | 通常两种情况 ：使用错误的scheme进行权限检查，SASL权限检查失败 | AuthFailedException         |



watcher的特性

-  一次性：无论客户端还是服务端，一旦一个Watcher被触发，Zookeeper都会将其从对应的存储中移除。所以在使用watcher的时候需要反复注册。这样的设计有效减轻了服务器的压力。

- 客户端串行执行：客户端watcher回调的过程是一个串行同步的过程。

- 轻量：watchedEvent是zk整个watcher通知机制的最小通知单元，这个数据结构只包含三部分：通知状态，事件类型和节点路径。也就是说，watcher的通知非常简单，只会告诉客户端发生了事件，而不会说明事件的具体内容。

  

###### 2.3 ACL--保障数据的安全

ACL机制：权限模式（scheme），授权对象（ID）和权限（Permission），通常使用“schema：id:permission”来标识一个有效的ACL信息。



###### 2.3.1 权限模式：scheme

- IP：IP模式通过IP地址粒度来进行权限控制。
- Digest：“username：password”形式的权限标识来进行权限配置，便于区分不同应用来进行权限控制
- world：最开放的权限控制模式。数据节点的访问权限对所有用户开放，所有用户都可以在不进行任何权限校验的情况下操作zk上的数据。特殊的Digest模式，权限标识：“world：anyone”
- Super：超级用户，也是一种特殊的Digest模式。在super模式下，超级用户可以对任意zk上的数据节点进行任何操作。

###### 2.3.2授权对象：ID

授权对象指的是权限赋予的用户或一个制定实体，例如IP地址或是机器。



###### 2.3.3 权限：Permission

权限是指那些通过权限检查后可以被允许执行的操作。

- CREATE（C）：数据节点的创建权限，允许授权对象在该数据节点下创建子节点。
- DELETE（D）：子节点的删除权限。
- READ（R）：数据节点的读取权限
- WRITE（W）：数据节点的读取权限
- ADMIN（A）：数据节点的管理权限，允许授权对象对该数据节点进行ACL相关的设置操作。






