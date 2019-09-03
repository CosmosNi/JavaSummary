# redis

###### 1.1 redis的持久化方式

- RDB：在指定的时间间隔对数据进行快照存储。Redis可以通过创建快照来获取存储在内存中的数据在某个时间点上的副本。redis创建快照之后，可以对快照进行备份，可以将快照复制到其他服务器从而创建相同数据的服务器副本。
- AOF：记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据。

###### 1.2 线程模型

redis内部使用文件事件处理器，这个文件处理器是单线程的，所以redis才叫做单线程的模型。他采用IO多路复用机制同时监听多个socket，根据socket上的事件；来选择对应的事件处理器进行处理。

文件处理器包含四个结构：多个socket，IO多路复用程序，文件事件分派器，事件处理器（连接应答处理器，命令请求处理器，命令回复处理器）。



###### 1.3 redis过期两种删除方式

- 定时删除：redis默认每隔100ms就随机抽取一些设置了过期时间的key，检查其是否过期，如果过期就删除
- 惰性删除：定期删除可能会导致很多过期的key到了时间并没有删除掉。加入你的过期key，考定期删除没有被删除掉，还停留在内存中，除非你的系统去查一下那个key，才会被redis给删除。



###### 1.4 redis的内存淘汰机制

1. **volatile-lru**：从以设置过期时间的数据集中挑选最近最少使用的数据淘汰。
2. **volatile-ttl**：从已设置过期时间的数据集中挑选将要过期的数据淘汰
3. **volatile-random**：从已设置过期时间的数据集中任意选择数据淘汰
4. **allkeys-lru**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key
5. **allkeys-random**：从数据集中任意选择数据淘汰
6. **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错
7. **volatile-lfu**：从已设置过期时间的数据集中挑选最不经常使用的数据淘汰
8. **allkeys-lfu**：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的key



###### 1.5 redis产生的相关问题

- 缓存穿透：是指查询一个数据库一定不存在的数据。正常的使用缓存的流程大致时，数据查询先进行缓存查询，如果key不存在或key已经失效，在对数据库进行查询，并把查询的对象，放入缓存。如果数据库查询对象为空，则不放进缓存。解决方案，对空值也做缓存，缓存周期设置较短一些。或此采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中。

- 缓存雪崩：是指在某一个时间段，缓存集中过期失效。解决方案，根据不同情况，缓存不同周期。用锁/分布式锁或者队列串行访问。

  事前:尽量保证整个redis集群的高可用性，发现机器宕机尽快补上。选择合适的内存淘汰策略。

  事中：本地ehcache缓存+hystrix限流&降级，避免Mysql崩掉

  事后：利用redis持久化机制保存的数据尽快回复缓存

- 缓存击穿：是指一个key非常热点，大并发集中对这个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库。解决方案，使用锁，单机用synchronized,lock等，分布式用分布式锁。缓存过期时间不设置，而是设置在key对应的value里。如果检测到存的时间超过过期时间则异步更新缓存。在value设置一个比过期时间t0小的过期时间值t1，当t1过期的时候，延长t1并做更新缓存操作。



###### 1.6 如何保证缓存与数据库双写时的数据一致性

读的时候，先读缓存，缓存没有的话，在读数据库，然后取出数据放入缓存，同时返回响应。

更新的时候，先删除缓存，在更新数据库。



数据库与缓存更新与读取操作进行异步串行化：

①更新数据的时候，根据数据的唯一标识，将操作路由之后，发送到一个jvm内部的队列里面去。

②读取数据的时候，如果发现缓存中没有，那么将从数据库读取数据的操作和更新缓存的操作一起路由到同一个JVM内部的队列中去。

③一个队列对应一个工作线程，然后线程从队列里面去取请求进行操作。
