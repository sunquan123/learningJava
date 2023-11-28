# Redis学习手册

## 如何集成Redis到Spring Boot中？

### 添加redis所需依赖：

```xml
<!-- redis 缓存操作 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.5.15</version>
</dependency>
<!-- 阿里JSON解析器 -->
<dependency>
    <groupId>com.alibaba.fastjson2</groupId>
    <artifactId>fastjson2</artifactId>
    <version>2.0.25</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.9.0</version>
</dependency>
```

### 添加配置

常规配置如下： 在application.yml配置文件中配置 redis的连接信息

```yaml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    password:
    database: 0
    lettuce:
      pool:
        max-idle: 16
        max-active: 32
        min-idle: 8
```

### 配置类

```java
@Configuration
public class RedisConfig {
  @Bean
  public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>();
    redisTemplate.setConnectionFactory(factory);
    StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

    FastJson2JsonRedisSerializer fastJson2JsonRedisSerializer =
        new FastJson2JsonRedisSerializer(Object.class);

    // 设置key和value的序列化规则
    redisTemplate.setKeySerializer(stringRedisSerializer); // key的序列化类型
    redisTemplate.setValueSerializer(fastJson2JsonRedisSerializer); // value的序列化类型
    redisTemplate.setHashKeySerializer(stringRedisSerializer);
    redisTemplate.setHashValueSerializer(fastJson2JsonRedisSerializer);
    redisTemplate.afterPropertiesSet();

    return redisTemplate;
  }
}
```

```java
public class FastJson2JsonRedisSerializer<T> implements RedisSerializer<T>
{
    public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

    private Class<T> clazz;

    public FastJson2JsonRedisSerializer(Class<T> clazz)
    {
        super();
        this.clazz = clazz;
    }

    @Override
    public byte[] serialize(T t) throws SerializationException
    {
        if (t == null)
        {
            return new byte[0];
        }
        return JSON.toJSONString(t, JSONWriter.Feature.WriteClassName).getBytes(DEFAULT_CHARSET);
    }

    @Override
    public T deserialize(byte[] bytes) throws SerializationException
    {
        if (bytes == null || bytes.length <= 0)
        {
            return null;
        }
        String str = new String(bytes, DEFAULT_CHARSET);

        return JSON.parseObject(str, clazz, JSONReader.Feature.SupportAutoType);
    }
}
```

### 项目中使用

```java
@Component
public class CacheUtils {
  private static Logger logger = LoggerFactory.getLogger(CacheUtils.class);
  @Autowired public RedisTemplate redisTemplate;
  private static final String SYS_CACHE = "sys-cache";


  /**
   * 获取缓存
   *
   * @param cacheName
   * @param key
   * @return
   */
  public Object get(String cacheName, String key) {
    //    return getCache(cacheName).get(getKey(key));
    return redisTemplate.opsForHash().get(cacheName, getKey(key));
  }


  /**
   * 写入缓存
   *
   * @param cacheName
   * @param key
   * @param value
   */
  public void put(String cacheName, String key, Object value) {
    redisTemplate.opsForHash().put(cacheName, getKey(key), value);
  }

  /**
   * 从缓存中移除
   *
   * @param cacheName
   * @param key
   */
  public void remove(String cacheName, String key) {
    redisTemplate.opsForHash().delete(cacheName, getKey(key));
  }

  /**
   * 从缓存中移除所有
   *
   * @param cacheName
   */
  public void removeAll(String cacheName) {
    Set<String> keys = redisTemplate.opsForHash().keys(cacheName);
    for (String key : keys) {
      redisTemplate.opsForHash().delete(cacheName, key);
    }
    logger.info("清理缓存： {} => {}", cacheName, keys);
  }
}
```

### 如何在单元测试中使用Redis？

重点解决的问题是无法自动注入RedisTemplate，所以手动初始化RedisTemplate，中间需要初始化LettuceConnectionFactory工厂类，设置好redis服务器的地址和密码等参数。重点需要执行connectionFactory.afterPropertiesSet()方法，保证工厂能正常初始化成功。

```java
@SpringBootTest(classes = CacheUtilsTest.class)
class CacheUtilsTest {
  private static Logger logger = LoggerFactory.getLogger(CacheUtilsTest.class);
  private static RedisTemplate<String, Object> redisTemplate;

  @BeforeAll
  static void startRedis() {
    redisTemplate = new RedisTemplate<String, Object>();
    RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration("192.168.56.100", 6379);
    redisStandaloneConfiguration.setDatabase(0);
    redisStandaloneConfiguration.setPassword("123456");
    LettuceConnectionFactory connectionFactory = new LettuceConnectionFactory(redisStandaloneConfiguration);
    connectionFactory.afterPropertiesSet();
    redisTemplate.setConnectionFactory(connectionFactory);
    StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

    FastJson2JsonRedisSerializer fastJson2JsonRedisSerializer =
            new FastJson2JsonRedisSerializer(Object.class);

    // 设置key和value的序列化规则
    redisTemplate.setKeySerializer(stringRedisSerializer); // key的序列化类型
    redisTemplate.setValueSerializer(fastJson2JsonRedisSerializer); // value的序列化类型
    redisTemplate.setHashKeySerializer(stringRedisSerializer);
    redisTemplate.setHashValueSerializer(fastJson2JsonRedisSerializer);
    redisTemplate.afterPropertiesSet();
  }

  @Test
  void testGetCacheNames() {
    String pattern = "zhglxt";
    redisTemplate.opsForHash().put("zhglxt-sys-config","sys_config:sys.index.skinName","skin-purple");
    Set<String> keys = new HashSet<>();
    // 获取redis全部key
    Set<String> hashKetSet = redisTemplate.keys("*");
    logger.info("getCacheNames:{}", hashKetSet);
    for (Object s : hashKetSet) {
      String ss = (String) s;
      if (ss.startsWith(pattern)) {
        keys.add(ss);
      }
    }
    logger.info("getCacheNames end:{}", keys);
  }
}
```

TODO：可以使用embeded-redis进行单元测试环境下模拟redis的测试方式

## redis常用命令

```shell
# 登录时指定ip、端口、密码
redis-cli -h 127.0.0.1 -p 6379 -a 123456
# 登录后也可以指定密码
auth 123456
# 查询全部key
keys *
# 查看key是什么类型的数据
type key
# 查看key的剩余生存时间
ttl key
```

### Redis五种数据结构

#### string

##### 常用命令

```shell
# O(1)复杂度 有值则覆盖，无值则新建
set key val
# O(1)复杂度 删除某个key
del key
# O(1) 查看某key的value
get key
# O(1)复杂度 有值则不变，无值才新建
setnx key val
# O(1)复杂度 将val关联到key，并设置过期时间seconds 
setex key seconds val
# O(1)复杂度 有值则覆盖，无值则新建，并返回旧值
getset key val
# O(n)复杂度 同时设置多个key和val
mset key1 val1 key2 val2
# O(n)复杂度 有值则不变，无值才新建
msetnx key1 val1 key2 val2
# O(1)复杂度 将val追加到key对应的旧值中，无值则新建
append key value
# O(n)复杂度 同时获得多个key的val
mget key1 key2
# O(n)复杂度 返回key对应val的指定范围内容，范围由start和end指定
getrange key start end
# O(1)复杂度 返回指定key的val的字符串长度
strlen key
# O(1)复杂度 将key对应val减1
decr key
decrby key decrement
# O(1)复杂度 将key对应val加1
incr key
incrby key increment
```

##### setnx业务场景

###### 加锁

SETNX 可以用作加锁原语(locking primitive)。比如说，要对关键字(key) foo 加锁，客户端可以尝试以下方式：

```shell
SETNX lock.foo <current Unix time + lock timeout + 1>
```

如果 SETNX 返回 1 ，说明客户端已经获得了锁， key 设置的 unix 时间则指定了锁失效的时间。之后客户端可以通过 DEL lock.foo 来释放锁。

如果 SETNX 返回 0 ，说明 key 已经被其他客户端上锁了。如果锁是非阻塞(non-blocking lock)的，我们可以选择返回调用，或者进入一个重试循环，直到成功获得锁或重试超时(timeout)。

###### 处理死锁(deadlock)

如果因为客户端失败、崩溃或其他原因导致没有办法释放锁的话，怎么办？

这种状况可以通过检测发现——因为上锁的 key 保存的是 unix 时间戳，假如 key 值的时间戳小于当前的时间戳，表示锁已经不再有效。

但是，当有多个客户端同时检测一个锁是否过期并尝试释放它的时候，我们不能简单粗暴地删除死锁的 key ，再用 SETNX 上锁，因为这时竞争条件(race condition)已经形成了：

- C1 和 C2 读取 lock.foo 并检查时间戳，SETNX 都返回 0 ，因为它已经被 C3 锁上了，但C3 在上锁之后就崩溃(crashed)了。

- C1 向 lock.foo 发送 DEL 命令。

- C1 向 lock.foo 发送 SETNX 并成功。

- C2 向 lock.foo 发送 DEL 命令。

- C2 向 lock.foo 发送 SETNX 并成功。

- 出错：因为竞争条件的关系，C1 和 C2 两个都获得了锁。

怎么解决有待查询资料

##### incr业务场景

incr可以实现一个限流器，比如一个应用限制用户访问1秒钟最多10次请求。伪代码如下：

```java
FUNCTION LIMIT_API_CALL(ip)
ts = CURRENT_UNIX_TIME()
keyname = ip+":"+ts
current = GET(keyname)
IF current != NULL AND current > 10 THEN
 ERROR "too many requests per second"
END
IF current == NULL THEN
 MULTI
   INCR(keyname, 1)
   EXPIRE(keyname, 1)
 EXEC
ELSE
 INCR(keyname, 1)
END
PERFORM_API_CALL()
```

这里在增加次数和设置过期时间时使用事务，确保不会出现单个请求增加次数后未执行过期时间设置导致一直存在该缓存，导致一个用户永远只能用10次。

或者可以将incr和expire用一个lua脚本实现，也可以避免竞态问题出现：

```lua
local current
current = redis.call("incr",KEYS[1])
if tonumber(current) == 1 then
 redis.call("expire",KEYS[1],1)
end
```

#### hash

##### 常用命令

```shell
# O(1)复杂度
hget key field
# O(n)复杂度 返回key对应的hash中，所有的域和值。
hgetall key
# O(1)复杂度 有值则覆盖，无值则新建
hset key field value
# O(1)复杂度 有值则不变，无值才新建
hsetnx key field value
# O(n)复杂度 同时设置多个field和val
hmset key field1 value1 field2 value2
# O(n)复杂度 同时获得多个field的val
hmget key field1 field2
# (n)复杂度 同时删除多个field和val
hdel key field1 field2
# O(1)复杂度 获得指定key的hash中的field数量
hlen key
# O(1)复杂度 判断指定key的hash中field是否存在
hexists key field
# O(1)复杂度 为指定key的hash中的field的值加上增量 increment
hincrby key field increment
# O(n)复杂度 返回指定key的hash中的所有域
hkeys key
# O(n)复杂度 返回指定key的hash中的所有值
hvals key
```

list的常用命令

set的常用命令

#### bitmap

##### 常用命令

```shell
# O(1)复杂度 设置指定key下的bit数组指定index下的二进制数字是0还是1
setbit key index 0/1
# O(n)复杂度 获得指定key的bit数组中1的个数
bitcount key
```

##### bitmap的业务场景

实现用户上线次数统计：

举个例子，如果今天是网站上线的第 100 天，而用户 peter 在今天阅览过网站，那么

执行命令 SETBIT peter 100 1 ；如果明天 peter 也继续阅览网站，那么执行命令 SETBIT

peter 101 1 ，以此类推。

当要计算 peter 总共以来的上线次数时，就使用 BITCOUNT 命令：执行 BITCOUNT

peter ，得出的结果就是 peter 上线的总天数。

想要知道用户第几天上线也可以知道，整个存储成本相当的低。即使运行 10 年，占用的空间也只是每个用户 10*365 比特位(bit)，也即是每个用户 456 字节。但是对于这个数量级执行bitcount和get几乎一样快。

##### 如果一个bitmap特别大怎么计算bitcount比较好？

1. 分区域执行bitcount，再在内存中进行累加。

2. 将大的bitmap分散到不同的key中。

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

### ZSet

ZSet（也称为Sorted Set）是Redis中的一种特殊的数据结构，它内部维护了一个有序的字典，这个字典的元素中既包括了一个成员（member），也包括了一个double类型的分值(score)。这个结构可以帮助用户实现记分类型的排行榜数据，比如游戏分数排行榜，网站流行度排行等。

Redis中的ZSet在具体实现上，有多种结构，大类的话有两种，分别是ziplist(压缩列表)和skiplist(跳跃表)，但是这只是以前，在Redis 5.0中新增了一个listpack（紧凑列表）的数据结构，这种数据结构就是为了替代ziplist的，而在之后Redis 7.0的发布中，在ZSet的实现中，已经彻底不在使用zipList了。

![](./pic/redis/zset.png)

当ZSet的元素数量比较少时，Redis会采用ZipList（ListPack）来存储ZSet的数据。ZipList（ListPack）是一种紧凑的列表结构，它通过连续存储元素来节约内存空间。当ZSet的元素数量增多时，Redis会自动将ZipList（ListPack）转换为SkipList，以保持元素的有序性和支持范围查询操作。

在这个过程中，Redis会遍历ZipList（ListPack）中的所有元素，按照元素的分数值依次将它们插入到SkipList中，这样就可以保持元素的有序性。

在Redis的ZSET具体实现中，SkipList的这种实现，不仅用到了跳表，还会用到dict（字典）。

![](./pic/redis/zset-1.png)

其中，SkipList用来实现有序集合，其中每个元素按照其分值大小在跳表中进行排序。跳表的插入、删除和查找操作的时间复杂度都是 O(log n)，可以保证较好的性能。

dict用来实现元素到分值的映射关系，其中元素作为键，分值作为值。哈希表的插入、删除和查找操作的时间复杂度都是 O(1)，具有非常高的性能。

#### 何时转换

ZipList（ListPack）和SkipList之间是什么时候进行转换的呢？

当我们想zset中ADD第一个元素的时候，Redis会进行判断，如果符合以下条件，则使用zipList来实现zset，否则将使用skipList实现zset：

- ZSet集合中的元素数量小于zset_max_ziplist_entries（zset-max-listpack-entries） 的值（默认为 128 ）

- ZSet集合中所有元素的长度小于zset_max_ziplist_value（zset-max-listpack-value） 的值（默认为 64字节 ）

这时候，就会使用zipList来实现，否则会使用skipList来实现，但是，如果使用了zipList并不表示就不会变成skipList，当以上条件任意一个不被满足时，还是会转成skipList的。

总的来说就是，当元素数量少于128，每个元素的长度都小于64字节的时候，使用ZipList（ListPack），否则，使用SkipList！

#### 跳表

跳表也是一个有序链表，如下面这个数据结构：

![](./pic/redis/链表.png)

在这个链表中，我们想要查找一个数，需要从头结点开始向后依次遍历和匹配，直到查到为止，这个过程是比较耗费时间的，他的时间复杂度是0（n）。

当我们想要向这个链表中插入一个数的时候，过程和查找类似，先需要从头开始遍历找到合适的为止，然后再插入，他的时间复杂度也是 O(n)。

那么，怎么能提升遍历速度呢，有一个办法，那就是我们对链表进行改造，先对链表中每两个节点建立第一级索引，如下图所示：

![](./pic/redis/链表-1.png)

有了我们创建的这个索引之后，我们查询元素20，我们先从一级索引5->15 ->25 ->35中查找，发现20介于15和25之间，然后，转移到下一层进行搜索，即15->20->25，即可找到25这个节点了。

可以看到，同样是查找25，原来的链表需要遍历5个元素(1、5、10、15、20、25)，建立了一层索引之后，只需要遍历2个元素即可（15、20）。

有了上面的经验，我们可以继续创建二级索引、三级索引....

![](./pic/redis/链表-2.png)

在这样一个链表中查找25这个元素，只需要遍历2个节点就可以了（5、25）。

因为我们的链表不够大，查找的元素也比较靠前，所以速度上的感知可能没那么大，但是如果是在成千上万个节点、甚至数十万、百万个节点中遍历呢？这样的数据结构就能大大提高效率。

像上面这种带多级索引的链表，就是跳表。

## redis什么时候删除过期key？

我们都知道，Redis的Key是可以设置过期时间的，那么，过期了一定会立即删除吗？

回答这个问题之前，我们先看下Redis是如何实现的Key的过期。

以下是官网中关于过期的实现的描述（https://redis.io/commands/expire/ ）：

![](./pic/redis/过期删除.png)

也就是说Redis的键有两种过期方式：一种是惰性删除，另一种是定时删除。

惰性删除指的是当某个客户端尝试访问一个键，发现该键已经超时，那么它会被从Redis中删除。

当然，仅仅依靠惰性删除还不够，因为有些过期的键可能永远不会再被访问。这些键应该被及时删除，因此Redis会定期随机检查一些带有过期时间的键。所有已经过期的键都会从键空间中删除。

具体来说，Redis每秒会执行以下操作10次：

- 从带有过期时间的键集合中随机选择20个键。

- 删除这些键中所有已经过期的键。

- 如果已经过期的键占比超过25%，则重新从步骤1开始。

直到过期Key的比例下降到 25% 或者这次任务的执行耗时超过了25毫秒，才会退出循环。

所以，Redis其实是并不保证Key在过期的时候就能被立即删除的。因为一方面惰性删除中需要下次访问才会删除，即使是定时删除，也是通过轮询的方式来实现的。如果要过期的key很多的话，就会带来延迟的情况。

### 定时删除

#### 优点

1. 及时释放内存：主动删除能够及时地释放过期键占用的内存，避免内存空间被长时间占用，从而降低了内存使用率。

2. 很多冷数据可以被主动删除及时清空掉。

3. 避免写操作延迟：由于过期键被定期删除，不会导致过多的过期键在访问时触发删除操作，因此可以减少读写操作的延迟。

#### 缺点

1. 增加系统开销：定期扫描和删除操作会增加系统的开销，特别是在有大量键需要处理时，可能会导致Redis的性能下降。

### 惰性删除

#### 优点

1. 减少系统开销：被动删除不会定期地进行扫描和删除操作，因此可以减少系统的开销，节省计算资源。 

#### 缺点

1. 可能导致内存占用高：被动删除可能导致过期键长时间占用内存，直到被访问时才被删除，这可能会导致内存占用率较高。系统有大量冷数据时会一直得不到释放。
2. 可能导致访问延迟：当大量键同时过期并在访问时触发删除操作时，可能会导致读写操作的延迟。

Redis的惰性删除策略，不需要额外配置。当你设置键的过期时间（TTL）时，Redis会自动处理被动删除。

要使用主动删除策略，需要在Redis配置文件中设置过期键检查的频率。你可以通过设置以下配置参数来调整主动删除的行为：

- hz（每秒执行的定时器频率）：增加该值可以提高主动删除的频率。

- maxmemory（Redis的最大内存限制）：设置合适的最大内存限制，以确保Redis在内存不足时触发主动删除。

例如，在Redis配置文件中可以设置：

```properties
maxmemory 1gb
hz 10
```

### 错误操作

开发者在设置键过期时间时常会出现：

1. 开始先插入一个键并设置了过期时间：set key1 value1 ex 200

2. 正常情况下这个键会在200s后被删除

3. 后续更新这个键时：set key1 value2

4. 这次忘记设置过期时间了，导致redis执行set命令时误以为需要清除过期时间，导致这个键再也不会过期了

5. 这时执行ttl key1，会看到返回-1（永不过期）

但是，仅仅通过给 key 设置过期时间还是有问题的。因为还是可能存在定时删除和惰性删除漏掉了很多过期 key 的情况。这样就导致大量过期 key 堆积在内存里，然后就 Out of memory 了。

怎么解决这个问题呢？答案就是：**Redis 内存淘汰机制。**

## Redis 内存淘汰机制是什么？

> 

Redis 提供 6 种数据淘汰策略：

1. **volatile-lru（least recently used）**：从已设置过期时间的数据集（`server.db[i].expires`）中挑选最近最少使用的数据淘汰。
2. **volatile-ttl**：从已设置过期时间的数据集（`server.db[i].expires`）中挑选将要过期的数据淘汰。
3. **volatile-random**：从已设置过期时间的数据集（`server.db[i].expires`）中任意选择数据淘汰。
4. **allkeys-lru（least recently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）。
5. **allkeys-random**：从数据集（`server.db[i].dict`）中任意选择数据淘汰。
6. **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！

4.0 版本后增加以下两种：

7. **volatile-lfu（least frequently used）**：从已设置过期时间的数据集（`server.db[i].expires`）中挑选最不经常使用的数据淘汰。
8. **allkeys-lfu（least frequently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 key。

### 项目中的使用策略

如果项目中的缓存数据比较重要，则不能配置带有allkeys前缀的淘汰机制，这些机制会在内存不足时有概率的删除掉未配置过期时间的key，导致重要数据被清空。应当使用volatile前缀的淘汰机制。

如果项目中的缓存数据被删除了也不会造成功能上的故障，则可以使用allkeys前缀的淘汰机制，这样也能保证在redis内存不足时及时清除数据，保证缓存可用。

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
