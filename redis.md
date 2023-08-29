redis五种数据结构

redis用途：缓存、限流、分布式锁、消息队列、session、排行榜、布隆过滤器

redis3种过期数据删除策略、6种内存数据淘汰策略

redis内存空间释放时机

redis事务、不支持原子性、支持持久性、三种持久化方式、aof三种策略

redis批量操作：原生批量操作、pipeline操作、pipeline和原生批量操作的区别、pipeline和事务操作的区别、lua脚本、lua脚本的优点与缺陷

两种解决大量key同时过期对redis造成影响的方式

```
1、设置不同的过期时间
2、lazy-free
```

发现big key的两种方式

```
1、bigkey参数
2、开源工具分析rdb文件
```

解决big key的四种方式

```
1、拆分
2、设置过期时间
3、有选择的删除
4、集群分片到不同节点
5、读写分离
6、二级缓存
```

发现hot key的四种方式

```
1、hotkey参数
2、jd项目监测热键
3、monitor
4、业务上预测出来
```

解决hot key的三种方式

```
1、读写分离
2、多级缓存
3、热键拆分
4、集群备份、同步、分片到多个节点上
```

慢查询命令、配置项

缓存穿透两种解决方式

```
1、规则校验参数
2、数据库读不出来后，写到缓存中一个null，最好有过期时间
3、布隆过滤器
```

缓存击穿三种解决方式

```
1、部分key定期异步续期
2、数据库读数据加互斥锁，将数据加载到缓存中，再释放锁
```

缓存雪崩三种解决方式

```
1、设置不同的过期时间
2、集群部署
3、部分key定期异步续期
```

旁路缓存读写、为什么要这么设计、两个缺陷与解决方式

```
写逻辑：写db，删缓存
读逻辑：读缓存/读db，更缓存
缺陷：1、频繁写缓存命中率较低2、先读再写，缓存可能是错的3、首次一定读db
解决：1、缓存db强一致场景：每次更新db也更新缓存，用分布式锁确保线程安全
2、允许db和缓存暂时不一致的业务：每次更新db也更新缓存，缓存设置过期时间短
```

读写穿透读写、和旁路缓存区别

```
读穿透：读缓存，缓存中无数据，由缓存读db数据到缓存中，再返回
写穿透：写数据到缓存，由缓存写数据到db中
区别：通过缓存去做这些操作
```

异步缓存写入读写、和读写穿透区别

```
只更缓存，使用异步方式更新db
适合更新频繁，但是一致性要求不要的场景：点赞数、访问量
```

三种读写方式：数据一致性、并发性能

redis阻塞的情况：O(n)的命令、AOF文件刷盘、save命令、AOF重写阻塞、big key传输阻塞、查找big key阻塞、清空数据库、集群扩容阻塞、使用了swap、cpu竞争、定期删除过期key阻塞主线程（可以设置lazy-free）、达到最大内存上限（内存淘汰）

字符串：命令、sds（三个相对于c原生字符串优势）、场景（session、token、图片地址、序列化对象、用户限流、页面限流、分布式锁）

list：命令、双向链表、场景（队列、栈、简陋消息队列）

hash：命令、数组+链表、场景（对象、用户信息、购物车信息、商品信息、文章信息）

set：命令、hashset、场景（点赞数、交并差操作、共同好友、共同关注、好友推荐、公众号推荐、随机抽取用户中奖、随机点名）

zset：命令、有打分的set（ziplist+skiplist，7.0后使用listpack取代ziplist，什么时候切换ziplist到skiplist、跳表是什么（跳表+hash，读score是O(1)，范围查询是O(logn)））、场景（交并差操作、排行榜、送礼、微信步数、段位排行、热度排行、优先级任务队列）

bitmap：命令（操作多个bitmap？）、存储二进制数字的数组、场景（用户签到、活跃用户、用户行为统计（是否点赞过某个视频））

hyperloglog：其他的不太了解，场景（数据量巨大的技术场景，对ip的访问统计、对网站的UV统计）

geospatial index：常用命令、sorted set、场景（地图两点距离、得到某点附近的点、附近的人（jedis实现））

redis产生内存碎片的2个场景、如何查看内存碎片率、2种清理内存碎片方式

RDB作用、两个命令、是否会阻塞主线程

AOF是什么、AOF持久化流程、3种AOF写入策略、AOF重写是什么、AOF重写流程、AOF配置项、AOF手动触发、AOF文件校验机制、rdb+aof是什么

AOF和RDB各自的优势、业务选择什么持久化

分布式锁的用途、性质、主要实现方式

redis实现分布式锁：自己写脚本、redisson、锁的优雅续期、可重入锁、分布式锁的集群可靠性方案

增加分布式锁性能：分段锁（100个商品库存分成10key，一个key保存10个库存，分开让客户端请求，并发性能提升十倍）

zookeper实现分布式锁：zk节点、监听器、curator框架、可重入锁

redis集群是AP

redis主从复制、哨兵模式、cluster（哈希槽、数据分片、2的14次方个槽位、为什么用这个数字）

redis协议：RESP

redis和memcached的区别

redis为什么设计成单线程的，为什么不用多线程

redis瓶颈在哪

IO多路复用技术

redis6.0后多线程用在哪里、为了什么

setnx命令、lua脚本为什么保证原子性、lua脚本的超时处理

```
Redis 的指令执行本身是单线程的，这个线程还要执行客户端的 Lua 脚本，如果 Lua脚本执行超时或者陷入了死循环，是不是没有办法为客户端提供服务了呢？

例如：
eval 'while(true) do end' 0
为了防止某个脚本执行时间过长导致 Redis 无法提供服务，Redis 提供了lua-time-limit 参数限制脚本的最长运行时间，默认为 5 秒钟。

lua-time-limit 5000（redis.conf 配置文件中）

当脚本运行时间超过这一限制后，Redis 将开始接受其他命令但不会执行（以确保脚本的原子性，因为此时脚本并没有被终止），而是会返回“BUSY”错误。

Redis 提供了一个 script kill 的命令来中止脚本的执行。新开一个客户端：

script kill

如果当前执行的 Lua 脚本对 Redis 的数据进行了修改（SET、DEL 等），那么通过script kill 命令是不能终止脚本运行的

127.0.0.1:6379> eval "redis.call('set','vincent','666') while true do end" 0

因为要保证脚本运行的原子性，如果脚本执行了一部分终止，那就违背了脚本原子性的要求。最终要保证脚本要么都执行，要么都不执行

127.0.0.1:6379> script kill
(error) UNKILLABLE Sorry the script already executed write commands against the dataset. You can either wait the script
termination or kill the server in a hard way using the SHUTDOWN NOSAVE command.

遇到这种情况，只能通过 shutdown nosave 命令来强行终止 redis。

shutdown nosave 和 shutdown 的区别在于 shutdown nosave 不会进行持久化操作，意味着发生在上一次快照后的数据库修改都会丢失
```

stream常用命令、消费者组命令

虚拟内存机制、配置、场景

事务机制：不支持回滚、往下执行全部命令

缓存与db不一致问题：

双更怎么都会不一致（失败问题、并发问题）、双更缓存利用率太低、性能浪费；

删缓存更db（读写不一致）

更db删缓存（缓存失效时读写不一致，概率较低、失败问题依然导致不一致）

为了解决失败问题，应用中引入消息队列重试方式，由专门消费者处理重试逻辑；订阅数据库binlog变更日志，使用canal投递数据给消息队列，由专门消费者处理重试逻辑

先删后更、数据库主从延迟下先更后删都会出现缓存旧值情况，解决需要延迟双删

电商平台某个时间某个不知名商品突然访问量暴增，怎么防止缓存击穿？

因为事先不知道某个商品会大卖，无法加到缓存里。读逻辑先读缓存，缓存有就返回；缓存没有就先加reddsion锁，第一个读线程先上锁后，读数据库然后写缓存；第二个线程拿不到分布式锁，等待；

redis怎么解决读写不一致场景：延时双删+过期时间；

## 资料：

- [删除数据后，redis为什么内存占用率还是很高？-极客时间](https://time.geekbang.org/column/article/289140)
- [你的 Redis 真的变慢了吗？性能优化如何做 - 阿里开发者](https://mp.weixin.qq.com/s/nNEuYw0NlYGhuKKKKoWfcQ)
- [Redis延迟问题全面排障指南](https://mp.weixin.qq.com/s/mIc6a9mfEGdaNDD3MmfFsg)
- [Redis 开发与运维笔记-Redis 的噩梦-阻塞](https://mp.weixin.qq.com/s/TDbpz9oLH6ifVv6ewqgSgA)
- [一文详解 Redis 中 BigKey、HotKey 的发现与处理](https://mp.weixin.qq.com/s/FPYE1B839_8Yk1-YSiW-1Q)
- [缓存和数据库一致性问题，看这篇就够了 - 水滴与银弹](https://mp.weixin.qq.com/s?__biz=MzIyOTYxNDI5OA==&mid=2247487312&idx=1&sn=fa19566f5729d6598155b5c676eee62d&chksm=e8beb8e5dfc931f3e35655da9da0b61c79f2843101c130cf38996446975014f958a6481aacf1&scene=178&cur_album_id=1699766580538032128#rd)
- [阿里云 Redis 开发规范](https://developer.aliyun.com/article/531067)
- [HyperLogLog 算法的原理讲解以及 Redis 是如何应用它的](https://juejin.cn/post/6844903785744056333)
- [Sketch of the Day: HyperLogLog — Cornerstone of a Big Data Infrastructure](http://content.research.neustar.biz/blog/hll.html)
- 布隆过滤器,位图,HyperLogLog：https://hogwartsrico.github.io/2020/06/08/BloomFilter-HyperLogLog-BitMap/index.html
- [Redis 到底是怎么实现“附近的人”这个功能的呢？](https://juejin.cn/post/6844903966061363207)
- Redis 源码解析——内存分配：[https://shinerio.cc/2020/05/17/redis/Redis源码解析——内存管理](https://shinerio.cc/2020/05/17/redis/Redis%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90%E2%80%94%E2%80%94%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86)
- [Redis 7.0 Multi Part AOF 的设计和实现](https://zhuanlan.zhihu.com/p/467217082)
- 《Redis 开发与运维》
- 《Redis 设计与实现》
- Redis Transactions : [https://redis.io/docs/interact/transactions/](https://redis.io/docs/manual/transactions/)
- Redis Commands：[Commands | Redis](https://redis.io/commands/) 
- Redis Data types tutorial：https://redis.io/docs/manual/data-types/data-types-tutorial/ 
- What is Redis Pipeline：[What is Redis Pipeline](https://buildatscale.tech/what-is-redis-pipeline/)
