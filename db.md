tinyint、smallint、mediumint、int、bigint、float、double、decimal、char、varchar、text、binary、varbinary、blob、datetime、timestamp、enum、set

primary key、unique、use index(a,b)、ignore index(a)

union查询、子查询（from、where）、inner join=join、outer join、left join、right join

DB、DBMS、DBA、DBS

元组、码、候选码、主码、外码、主属性、非主属性

第一范式、第二范式、第三范式

ER图、实体、属性、关系

主码和外码

drop、delete、truncate区别

ascii、gb2312、gbk、gb18030、big5、unicode、utf-8

mysql配置字符集层次：实例、数据库、表、列、连接

mysql utf-8和utf-8mb4

jdbc连接mysql显示异常

char和varchar区别

varchar(10)和varchar(100)区别

decimal和float、double区别

text、blob使用

null和''区别

mysql：连接器、分析器、优化器、执行器、存储引擎、日志模块

mysql执行sql语句的过程

隐式类型转换索引失效

```sql
select 12 = '012avbgbg'
```

自增主键不连续的四个场景

读未提交、读已提交、可重复读、串行化

一致性非锁定读、快照读、MVCC、锁定读、当前读、Next-key Lock

MVCC的实现：隐藏字段、read view、undo-log

MVCC视角下的读已提交、可重复读

RR隔离级别下解决幻读方式：MVCC、Next-key Lock

redolog作用、内容物理形式、redolog buffer、redolog刷盘策略（0、1、2）、后台线程刷盘策略（1s、1/2）、redolog文件日志组

binlog作用、内容逻辑形式、记录的三种格式、binlog cache、binlog写入时机（0、1、N）

redolog和binlog两阶段提交

explain、select_type、type、key、possible_keys、extra、rows

undolog作用

redolog、undolog、binlog满足了事务的几大特性

timestamp和datetime区别，存储大小、记录时间范围、时区显示

锁表lock tables、触发器、存储过程、分析表

哈希表、二叉树、AVL树、红黑树、B树、B+树、哈希索引、全文索引

非聚簇索引、聚簇索引、二级索引（辅助索引）

主键索引、普通索引、唯一索引、联合索引、全文索引、前缀索引（字符串类型）

覆盖索引、最左前缀匹配原则：范围查询之后的不在索引里查询（>、<）；>=、<=、between and、like前模糊不停止，可以继续匹配，mysql 8.0.13后sql可以不满足最左前缀匹配，mysql优化器使用跳跃扫描保证查询用到索引，但是限制较多

acid是什么

脏读、丢失修改、不可重复读、幻读

行锁、表锁、共享锁、独占锁、意向锁、自增锁的三种模式、对于插入数据的影响

可重复读下的MVCC快照读、可重复读下的加锁读、加锁更新

可重复读级别下，出现幻读：插入数据、写事务提交、读事务加锁读

读已提交+binlog statement状态+更新数据，出现从库数据不一致问题

mysql5.7下的next-key lock是什么：唯一索引的范围查询、索引上的等值查询

innodb加索引，是否会锁表

索引条件下推（ICP）？

mysql如何保证唯一索引的唯一性的

count(*)、count(1)、count(列)区别、count(**)的优化

order by可能返回顺序不固定，可优化

深度分页limit优化

mysql实现insertOrUpdate功能

走不走索引：计算、函数、or、like、隐式类型转换、in

区分度不高的字段也可以用索引？

优化器选择驱动表对join的影响

hash join的优势：基于内存、基于磁盘

buffer pool是什么

怎么做热点数据行的更新？

## 深入阅读：

- 《MySQL 技术内幕：InnoDB 存储引擎》
- https://dev.MySQL.com/doc/refman/5.7/en/
- [一篇文章看懂mysql中varchar能存多少汉字、数字，以及varchar(100)和varchar(10)的区别 - 那些年的代码 - 博客园](https://www.cnblogs.com/zhuyeshen/p/11642211.html)
- [联合索引的最左匹配原则全网都在说的一个错误结论](https://mp.weixin.qq.com/s/8qemhRg5MgXs1So5YCv0fQ)
- [为什么 MySQL 的自增主键不单调也不连续 - 面向信仰编程](https://draveness.me/whys-the-design-mysql-auto-increment/)
- [技术分享 | 隔离级别：正确理解幻读](https://opensource.actionsky.com/20210818-mysql/)
- [详解 MySql InnoDB 中意向锁的作用 - 掘金](https://juejin.cn/post/6844903666332368909)
- [Mysql 锁：灵魂七拷问](https://tech.youzan.com/seven-questions-about-the-lock-of-MySQL/)
- [MySQL next-key lock 加锁范围](https://segmentfault.com/a/1190000040129107)
- [Innodb 中的事务隔离级别和锁的关系](https://tech.meituan.com/2014/08/20/innodb-lock.html)
- [深入剖析 MySQL 自增锁 - 掘金](https://juejin.cn/post/6968420054287253540)
- [Spring Boot 整合 MinIO 实现分布式文件服务](https://www.51cto.com/article/716978.html)
- 
- [《周志明的软件架构课》](https://time.geekbang.org/opencourse/intro/100064201)
- [一树一溪的 MySQL 系列教程](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg3NTc3NjM4Nw==&action=getalbum&album_id=2372043523518300162&scene=173&from_msgid=2247484308&from_itemidx=1&count=3&nolastread=1#wechat_redirect)
- [Yes 的 MySQL 系列教程](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzkxNTE3NjQ3MA==&action=getalbum&album_id=1903249596194095112&scene=173&from_msgid=2247490365&from_itemidx=1&count=3&nolastread=1#wechat_redirect)
- [写完这篇 我的 SQL 优化能力直接进入新层次 - 变成派大星 - 2022](https://juejin.cn/post/7161964571853815822)
- [两万字详解！InnoDB 锁专题！ - 捡田螺的小男孩 - 2022](https://juejin.cn/post/7094049650428084232)
- [深入理解 MySQL 索引底层原理 - 腾讯技术工程 - 2020](https://zhuanlan.zhihu.com/p/113917726)
- 
- 
- 
- 《高性能 MySQL》第 7 章 MySQL 高级特性
- 《MySQL 技术内幕 InnoDB 存储引擎》第 6 章 锁
- Relational Database：https://www.omnisci.com/technical-glossary/relational-database
- 
- 
- 
- 
