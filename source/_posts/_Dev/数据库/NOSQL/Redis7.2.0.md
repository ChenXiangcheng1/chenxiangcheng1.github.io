---
title: Redis
date: 2023-05-02
update: 2023-07-24

categories:
  - 数据库
keywords: Redis, 缓存
description: 关于Redis的笔记,涉及Redis简介、Redis数据类型、Redis命令、缓存、异步消息队列、分布式锁、rdb-aof持久化、主从同步、Redis集群
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_10_11_29.png" alt="image-20230510112901318
---



# Redis7.2.0

**Redis 是一个内存数据库，能够持久化到磁盘上。**数据模型是**键值对**，但支持许多不同类型的值：**Strings字符串、Lists列表、Sets集合、Sorted Sets排序集合、Hashes哈希、Streams流、HyperLogLogs、位图**
应用场景：分布式缓存(Mybatis也有缓存，但Redis是**分布式**的)

另一个很好的例子是将 Redis 视为 memcached 的一个更复杂的版本，其中的操作不仅仅是 SET 和 GET，而是使用**复杂数据类型**的操作，如Lists列表、Sets集合、有序数据结构等等。



可以在 Linux/WSL 中进行部署
docker 中安装 Redis Stack (没人用)



TODO：有空看跳表、看[文档](https://redis.io/docs/data-types/streams/)



## 文档

[官方文档](https://redis.io/docs/)	|	[github源码](https://github.com/redis/redis)	|	[github redis-windows](https://github.com/redis-windows/redis-windows/blob/main/README.zh_CN.md)	|	[docker镜像 redis](https://hub.docker.com/_/redis)

[资源#实现了Redis协议的客户端](https://redis.io/resources/clients/#java)

[资源#使用了Redis的库](https://redis.io/resources/clients/#java)

[资源#管理和部署Redis的工具](https://redis.io/resources/tools/)

[资源#扩展Redis服务功能的模块](https://redis.io/resources/modules/)



需要使用 CLI Redis 命令时，查询 [命令文档](https://redis.io/commands/)，命令名都是有语义



## 1简介

**缓存中间件 Memcache 和 Redis 的区别**

| Memcache：键值对     | Redis                  |
| -------------------- | ---------------------- |
| 支持简单数据类型     | 数据类型丰富           |
| 不支持数据持久化存储 | 支持数据磁盘持久化存储 |
| 不支持主从同步       | 支持主从同步           |
| 不支持数据分片       | 支持数据分片(集群)     |



**为什么 Redis 能这么快**

10w + QPS (query per second)

* 完全基于内存，绝大部分请求是**纯内存操作**，不需要像磁盘I/O那样阻塞操作，执行效率高

* 数据结构简单 没有 RDB 一样的强制关联，对数据操作也简单 hashmap O(1)

* 采用**单线程**处理网络请求，单线程也能处理高并发请求，若想使用多核也可启动多实例
  对于客户端的读写请求由一个主线程串行处理，写操作不会有并发问题，避免了频繁的上下文切换和锁竞争。所有操作都是**原子性**的

* 使用**AIO模型**，事件驱动的异步非阻塞IO+回调机制，监听多个 FD 实行多客户端高并发

[AIO笔记](../../Java/类库/Java的IO机制.md#AIO 异步IO)



**主流应用架构**

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_10_11_29.png" alt="image-20230510112901318" style="zoom:80%;" />



## 缓存的雪崩、穿透、击穿

**缓存雪崩** Cache Avalanche：在某一时间段，缓存中的大量键同时失效或过期，此时有大量请求访问，可能导致数据库压力过大宕机

解决方案1：给旧数据过期加个几分钟随机时间
缺点：可能过几分钟还是有大量请求

解决方案2：加互斥锁控制去读数据库的线程数量
缺点：该时间段吞吐量下降

解决方案3：做二级缓存，A1为原始缓存短期的，A2为拷贝缓存长期的，A1失效了，A2还可以查询



**缓存穿透** Cache Penetration：大量恶意请求或无效请求访问，可能导致数据库压力过大宕机

解决方案：
1使用布隆过滤器进行预检查，将所有可能存在的 key 存在一个足够大的 bitmap 中，那么一个不存在的数据就会被这个bigmap 拦截，从而减轻对数据库的访问压力
2缓存空值，对数据库查询为空的请求，将其缓存为空值并给予一个较短的过期时间
3网关层限制恶意 IP 访问
4做好参数校验，对不合理的参数进行拦截不走数据库



**缓存击穿** Cache Miss：大量请求访问某一热点 key，此时热点 key 过期，，可能导致数据库压力过大宕机

解决方案：热点数据不过期，活动结束后再删除



## 2value的数据类型及其命令

[文档#数据类型](https://redis.io/docs/data-types/)

key 的命名：项目名/应用名:表名/业务模块名user|product|cache|counter|lock:字段名xxx_name:xxx_id:状态:value类型str|lst|map|set|zst
全小写

| value的数据类型 | 数据结构                                  | 存储             | 应用场景                                                     |
| --------------- | ----------------------------------------- | ---------------- | ------------------------------------------------------------ |
| String          | 二进制安全动态字符串                      | 文本、结构体信息 | 缓存 HTML 片段或页面、Counter计数器、分布式锁setnx           |
| List            | 字符串链表                                | 序列信息         | 实现堆栈和异步消息队列(存储最新N个内容、消息订阅/发布)       |
| Set             | 唯一字符串的无序集合                      | 对象唯一ID信息   | 集合运算(百万用户共同关注共同喜好)、表示关系(配合结构体信息使用) |
| Hash            | 键值对集合                                | 结构体信息       | 缓存                                                         |
| Zset            | 按关联的 score 排序的唯一字符串的有序集合 |                  | 排行榜                                                       |



### 通用命令

不同数据类型的通用命令，list前缀l，set前缀s，hash前缀h，zset前缀z，多键操作前缀m

| 命令           | 示例          | 返回                                       | 释义                                                         |
| -------------- | ------------- | ------------------------------------------ | ------------------------------------------------------------ |
| select <index> | select 0      |                                            | 更改所选数据库<br /><br />**Redis多数据库**的说明：<br />Redis不支持自定义数据库名称。<br/>Redis不支持为每个数据库设置访问密码<br/>Redis的多个数据库之间不是完全隔离的，FLUSHALL命令会清空所有数据库的数据。<br/>多数据库不适用存储不同应用的数据(一个Redis服务对应一个应用) |
| **set**        | set name lily |                                            | 设置key=name, value=lily，若键已存在则会修改该键值对的数据类型与数据 |
| **get**        | get hello     |                                            | 获得key=hello结果<br />当键不存在时，返回nil空               |
| del            | del a         | 返回值是删除的键的个数                     | 删除一个或多个键值对                                         |
| type           |               | 返回值可能是 string、hash、list、set、zset | 获得键值的数据类型。                                         |
| dbsize         | dbsize        |                                            | 返回 key 的总数                                              |
| exists         | exists a      | 如果存在返回整数类型1，否则则返回0         | 检查 key=a 是否存在                                          |
|                |               |                                            |                                                              |



#### 自动创建和移除 key

聚合数据类型：list、set、zset、hash、stream

* 当向聚合数据类型中添加元素时，如果目标 key 不存在，则会在添加元素之前创建一个空的聚合数据类型。

* 当从聚合数据类型中删除元素后，如果 value 为空，则 key 会被自动销毁。 Stream 数据类型是此规则的唯一例外。

* 调用只读命令(如 LLEN)，或者删除空 key，总是产生相同的结果，好像键持有一个空的期望的聚合类型，但其实它不存在



#### Keys 与 scan 对比

场景：从海量Key里查询出某一固定前缀的Key

| 命令 | 示例                                                         | 返回 | 释义                                                         |
| ---- | ------------------------------------------------------------ | ---- | ------------------------------------------------------------ |
| keys | keys pattern<br />**Pattern表达式**的说明：<br /> ?匹配一个字符<br />*匹配任意个(包括0个)字符<br />[]匹配括号间的任一字符，可以使用“-”表示范围，如a[a-d]可以匹配“ab”，“ac”，“ad”<br />\x匹配字符x，用于转义符号，如果要匹配“？”就需要使用\? |      | 根据 Pattern 表达式查询符合条件的 Key<br />KEYS 命令一次性返回所有匹配的 key<br />键的数量过大会使服务卡顿，消耗内存，O(n) <br />(不要用于生产环境，千万级别keys容易爆内存) |
| scan | SCAN cursor [MATCH pattern] [COUNT count]                    |      | 基于游标的迭代器，需要基于上一次的游标延续之前的迭代过程<br />以0作为游标开始一次新的迭代，直到命令返回游标0完成一次遍历<br />不保证每次执行都返回某个给定数量的元素，支持模糊查询<br />一次返回的数量不可控，只能是大概率符合count参数 <br />(适合用于生产环境) |



#### 设置生存时间

| 命令    | 示例              | 返回                           | 释义                                   |
| ------- | ----------------- | ------------------------------ | -------------------------------------- |
| expire  | expire hello 20   |                                | 设置生存时间，设置key=hello 20秒后过期 |
| pexpire | expire hello 2000 |                                | 设置生存时间，单位毫秒                 |
| PERSIST | PERSIST hello     |                                |                                        |
| ttl     | ttl hello         | 返回剩余生存时间，-1表示已结束 | 查看key=a的过期剩余时间                |



### String

简单动态字符串

* 最基本的数据类型，**二进制安全动态字符串**
  C语言中，字符串是char数组以\0结尾，会因为数据中包含特殊字符而造成解析错误，二进制不安全，取长度O(n)
  Redis实现了二进制安全的简单动态字符串sds，即可以存储任何二进制数据如 JSON 化对象、序列化的多媒体数据，取长度O(1)

* 一个字符串类型键允许存储的数据最大容量是512MB，建议单个 kv 不超过100 kb。

可用于实现缓存

```c++
/*
 * 保存字符串对象的结构：简单动态字符串sds
 */
struct sdshdr
    // buf中已占用空间的长度
    int len;
    // buf中剩余可用空间的长度
    int free;
    //数据空间
	char buf[];
};
```

| String类型命令 | 示例                                 | 返回                                       | 释义                                       |
| -------------- | ------------------------------------ | ------------------------------------------ | ------------------------------------------ |
| mset           | mset hello world java best           | OK、error                                  | 同时设置多个键值，key=hello,value=hello    |
| mget           | mget hello java                      | 返回数组<br />1) value1<br />2) vlaue2 ... | 同时获取多个键值                           |
| incr/decr      | incr count<br />decr count           | 返回修改后数值                             | key值自增/自减1                            |
| incrby/decrby  | incrby count 99<br />decrby count 99 | 返回修改后数值                             | 自增自减指定步长                           |
| incrbyfloat    |                                      | 返回修改后数值                             |                                            |
| strlen         |                                      | 返回 value 的长度                          | 返回 value 的长度，如果 key 不存在则返回 0 |



### List

* **是 String 元素组成的列表**，一系列字符串的“数组”，按插入顺序排序
* List列表的最大长度为 2^32- 1，可以包含 40 亿个元素

L表示列表最左边的操作，R表示列表最右边的操作

可用于实现堆栈或队列

| list类型命令l开头                                            | 返回                                                         | 释义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| lpush key [element...]                                       |                                                              | 将新元素添加到列表的左侧（头部）                             |
| rpush key [element...]                                       |                                                              | 将新元素添加到列表的右侧（尾部）                             |
| lpop listkey                                                 |                                                              | 从列表头部删除并返回一个元素，左侧弹出，如果弹出最后一个元素，则删除列表。 |
| rpop m                                                       |                                                              | 从列表尾部删除并返回一个元素，左侧弹出，如果弹出最后一个元素，则删除列表。 |
| LTRIM key <start> <end>                                      |                                                              | 将列表缩小到指定的元素范围                                   |
| lrange list 0 -1                                             | 返回数据<br />1) value1<br />2) value2                       | 从列表中提取范围内的元素，索引-1是最后一个元素，-2是倒数第二个元素 |
| llen                                                         |                                                              | 返回列表的长度                                               |
| lmove                                                        |                                                              | 以原子方式将元素从源列表移动到目标列表                       |
| BLPOP key [key ...] timeout<br />BRPOP key [key ...] timeout | 返回两个元素的数组<br />1) key<br />2) value<br />超时返回 NULL | Block lpop 阻塞命令，从列表头部删除并返回一个元素。如果列表为空，则该命令将阻塞，直到有元素可用或达到指定的超时为止。timeout=0，永久等待。<br />客户端有序，第一个阻塞等待列表的客户端先得到服务 |
| blmove                                                       |                                                              | Block lmove 阻塞命令，以原子方式将元素从源列表移动到目标列表。如果源列表为空，该命令将阻塞，直到有新元素可用。 |



### Set

* **是 String 元素组成的无序集合**，通过哈希表实现，不允许重复
* Set 集合的最大大小为 2^32- 1，可以包含 40 亿个成员

| set类型命令s开头             | 返回                                                         | 释义                                                         |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| sadd key member [member ...] |                                                              | 将新成员添加到集合中                                         |
| srem                         |                                                              | 从集合中删除指定的成员                                       |
| spop                         |                                                              | 从集合中删除随机的一个成员                                   |
| smembers key                 |                                                              | 以任意顺序返回对应集合的所有成员，O(n)，不适用于生产环境     |
| SSCAN                        |                                                              | 迭代地检索集合的所有成员，适用于生产环境                     |
| srandmember                  |                                                              | 从集合中返回一个随机的成员                                   |
| scard                        |                                                              | 返回集合的大小（也称为基数）                                 |
| sinter setkey1 setkey2       |                                                              | 返回两个或多个集合共有的成员集（即交集）                     |
| sunion setkey1 setkey2       |                                                              | 返回两个或多个集合的并集                                     |
| sdiff setkey1 setkey2        | 返回集合setkey1减集合setkey2后<br />差异不存在时 (empty array) | 返回两个或多个集合的差集                                     |
| SISMEMBER key member         | 1属于 0不属于                                                | 测试一个成员是否属于目标集合。对于大数据集或流数据会使用大量内存，可以使用 [Bloom 布隆过滤器或 Cuckoo 布谷鸟过滤器](https://redis.io/docs/data-types/probabilistic/bloom-filter/)作为低精度替代方案 |
| SMISMEMBER                   | 1属于 0不属于                                                | 测试一个成员是否属于目标集合。对于大数据集或流数据会使用大量内存，可以使用 [Bloom 或 Cuckoo 过滤器](https://redis.io/docs/data-types/probabilistic/bloom-filter/)作为低精度替代方案 |



### Hash

* **是 String 元素组成的键值对集合**，适合用于存储结构化数据，例如对象、JSON
* 每个 hash 能存储 2^32- 1(40 亿)个键值对

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_10_19_45.png" alt="image-20230510194540167" style="zoom:67%;" />

| hash类型命令h开头              | 返回                       | 说明                  |
| ---------------------------- | --------------------| ---------------------------- |
| hset key field value [field value...] |  | 设置 hash 的一个或多个字段的值。 |
| hget key field      |       | 返回给定字段的值 |
| hmset emp:1 age 30 name kaka |  | 设置 hash 的多个字段   |
| hmget emp:1 age name         | 返回一个 values 数组 | 返回一个或多个给定字段的值 |
| hincrby key field increment |  | 将给定字段的值增加所提供的整数 |
| hgetall emp:1                |                 | 获取hash所有值，O(n)   |
| hkeys |  | 返回存储在 key 的 hash 中的所有字段名称，O(n) |
| hvals |  | 返回存储在 key 的 hash 中的所有值，O(n) |
| hdel emp:1 age               |                | 删除emp:1的age       |
| hexists emp:1 name           |            | 检查是否存在          |
| hlen emp:1                   |                    | 获取指定长度          |



### Sorted Set

* **是 String 元素组成的有序集合**，通过跳表和哈希表组成的一个双端口数据结构实现，不允许重复

* **Zset 添加了 score分数 **，来为集合中的成员进行从小到大的排序，若 score 一致则按字典序比较键名

| `sortedset`类型命令z开头                 | 返回                                                         | 释义                                                         |
| ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| zadd key score member [score member ...] |                                                              | 将新成员和关联的分数添加到排序集中。如果该成员已存在，则更新分数，O(logN) |
| ZRANGE key start stop [WITHSCORES]       | 返回排序集的成员<br />使用 WITHSCORES 参数同时返回分数<br />1)value1<br />2)score1... | 返回排序集的成员，在给定范围内从最小到最大排序，O(log(n)+m)，n是成员的数量，m是返回结果的数量 |
| ZREVRANGE key start stop [WITHSCORES]    | 返回排序集的成员<br />使用 WITHSCORES 参数同时返回分数<br />1)value1<br />2)score1... | 返回排序集的成员，在给定范围内从最大到最小排序，O(log(n)+m)，n是成员的数量，m是返回结果的数量 |
| zrangebyscore key minscore maxscore      |                                                              | 按 [minscore, maxscore] 范围，查 value，-inf 表示负无穷      |
| zremrangebyscore                         | 返回删除元素的数量                                           | 按 [minscore, maxscore] 范围，删除成员                       |
| zrank hackers "Anita Borg"               |                                                              | 返回该成员的排名，假设按升序排序                             |
| ZREVRANK hackers "Anita Borg"            |                                                              | 返回该成员的排名，假设按降序排序                             |
| zrangebylex hackers [B [P                |                                                              | lexicographically字典地，按 [minlex, maxlex] 范围，查 value，假设按字典升序排名 |
| ZREVRANGEBYLEX                           |                                                              | lexicographically字典地，按 [minlex, maxlex] 范围，查 value，假设按字典降序排名 |
| ZREMRANGEBYLEX                           |                                                              | 按 [minlex, maxlex] 范围，删除成员                           |
| ZLEXCOUNT                                |                                                              | 返回该成员的排名，假设按字典升序排序                         |



**以下数据类型不常用**

Stream：用于轻量级消息队列

Geospatial indices [dʒioʊ;] 地理空间索引：用于存储地理位置信息

Bitmap 位图

Bitfield 位域

概率论的数据结构：

HyperLogLog：用于估计集合的基数(去重计数统计)

Bloom filter 布隆过滤器

Cuckoo filter 布谷鸟过滤器





## 存储结构体信息

**在 string 中存储整个序列化对象**

```Redis CLI
INCR id:users  # 生成唯一的用户ID。
SET user:{id} '{"name":"Fred","age":25}'
SADD users {id}
```

优势：适合查询整个对象时使用

劣势：不适合频繁地访问或更新单个字段，需要解析 JSON 字符串



**在 hash 中存储每个对象的属性**

```Redis CLI
INCR id:users
HMSET user:{id} name "Fred" age 25
SADD users {id}
```

优势：适合查询某一字段时使用支持频繁更新，不需要解析 JSON 字符串

劣势：不适合嵌套对象的查询



## 在Redis中与数据交互

### Pub/Sub消息的发布与订阅

| 命令                   | 返回 | 说明                                                         |
| ---------------------- | ---- | ------------------------------------------------------------ |
| subscribe channel      |      | 订阅者可以订阅任意数量的频道(subscribe channel)来阻塞式接收消息 |
| public channel message |      | 发送者发送消息                                               |



## Redis异步消息队列

异步指：消息的生产者和消费者解耦，生产者可以异步地推送消息到队列，消费者可以异步地获取并处理消息

方案一：
使用List作为队列，RPUSH生产消息，LPOP消费消息
缺点：没有等待队列里有值就直接消费
弥补：可以通过在应用层引入Sleep机制去调用LPOP重试，有处理延迟



方案二：
使用Redis list 和 blpop 实现异步队列
缺点：一个消息只能供一个消费者消费



方案三：
Pub/Sub主题订阅者模式：发送者发送消息(public channel message)，订阅者可以订阅任意数量的频道(subscribe channel)来阻塞式接收消息
缺点：消息的发布是**无状态**的，**无法保证可达**
解决：使用专业的消息队列 Kafka



## 3Redis分布式锁

Java 的 synchronized 关键字只能用于同一 JVM 中线程的同步，而分布式锁可以跨多个 JVM

推荐使用 Redisson 中的分布式锁



### 实现分布式锁需要解决的问题

* 互斥性：任意时刻只能有一个客户端获取锁
  解决方式：redis 操作原子性 + NX

* 安全性：锁只能被持有该锁的客户端删除

* 死锁：需要有机制避免某一客户端宕机后死锁
  解决方式：EX

* 容错：服务端部分宕机客户端依然可以获取释放锁



### SET 命令及其参数

命令：`SET key value [NX|XX] [EX seconds] [PX milliseconds] `

* 参数

  * EX second：设置键的过期时间为 second 秒

  * PX millisecond：设置键的过期时间为 millisecond 毫秒

  * NX：not exist 只在键不存在时，才对键进行设置操作

  * XX：只在键已经存在时，才对键进行设置操作

* 返回值：设置成功，返回OK；设置失败，返回 nil。

* 时间复杂度：O(1)

例子：`set locktarget 12345 nx ex 10 `



### Jedis实现

```java
// 有 Bug，setnx 和 expire 的组合，原子性得不到满足
RedisService redisService = SpringUtils.getBean(RedisService.class);
Long status = redisService.setnx(key,"1");
if(status == 1){
	redisService.expire(key,expire);
	//执行独占资源逻辑
	doOcuppiedWork()
}

// 正确方式
RedisService redisService = SpringUtils.getBean(RedisService.class);
String result = redisService.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);
if ("OK".equals(result)) {
	redisService.expire(key,expire);
	//执行独占资源逻辑
	doOcuppiedWork()
}
```



由于清除大量的ky很耗时，在大量的key集中同时过期的时候会出现短暂的卡顿现象
**集中过期解决方案**：在设置每个key的过期时间的时候加上随机值



## 4数据持久化

[文档](https://redis.io/topics/persistence)	|	[别人博客笔记](http://oldblog.antirez.com/post/redis-persistence-demystified.html)

重启Redis服务时，为了恢复数据，避免从后端数据库中恢复数据

当文件损坏可以使用 redis-check-aof 或 redis-check-dump 修复文件

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_12_16_19.png" alt="image-20230512161932659" style="zoom:67%;" />

<center>重启Redis 数据恢复流程图</center>



### RDB快照

RDB(Redis DataBase)：备份数据库状态，保存某一时刻的 base 全量数据，创建 rdb 文件。在 Redis 服务启动时会检测并载入 rdb 文件内数据

base 全量数据：表示某一时刻的全部数据
inc 增量数据：相对另一时刻的变化数据



#### 手动触发

* save 命令：阻塞当前Redis服务器，直到 rdb 文件创建完成。
* bgsave 命令：fork() 系统调用创建子进程作为后台进程执行 rdb 持久化操作，fork() 时阻塞Redis服务器，rdb持久化操作时不阻塞

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_12_15_13.png" alt="image-20230512151350789" style="zoom:85%;" />

<center>bgsave 流程图</center>

检查子进程：防止子进程竞争

[fork()笔记](../../Linux/Linux.md#Fork())

Linux fork() 系统调用的**写时复制策略**与 Redis 的 bgsave 命令：fork() 系统调用创建子进程执行 rdb 持久化操作到临时 rdb 文件，当父进程需要处理写请求时会开辟新的临时内存区域存储这段时间发生的数据变化，子进程使用 fork() 时刻的内存区域(不含最新变化数据)，当子进程 rdb 持久化操作完成，将新临时内存区域同步到原来的内存区域，用临时 rdb 文件替换之前的 rdb 文件。



#### 自动触发

* 根据 redis.conf 配置里的 `save <seconds>  <changes>`：若seconds秒内有changes次修改时，定时触发 bgsave 生成 rdb 文件

* 主从复制时(从节点从主节点进行全量复制)：主节点触发 bgsave 操作生成 rdb 快照发送给从节点

* 执行 Debug Reload 命令时
* 执行 shutdown 命令且没有开启 AOF 持久化时



### AOF日志

AOF(append only file)：备份数据库修改操作的 redo log，本质是写后日志(先写内存，后写日志append only log)



#### 手动触发

* bgrewriteaof 命令：触发AOF重写操作，重写AOF日志文件

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_12_16_30.png" alt="image-20230512163007762" style="zoom:80%;" />

<center>bgrewriteaof  流程图</center>

重写(rewrite)AOF文件：当启动AOF持久化，随数据增加子进程增量地保存数据到磁盘AOF文件，不断地追加写命令到AOF文件末尾。Redis服务器执行完一个写命令后，会以协议格式将写命令附加到 aof_buf 缓冲区，再根据不同策略always/everysec/no 同步`fsync()`到 aof 文件中
自动重写，满足条件 BGREWRITEAOF 自动重写AOF文件

[write() 缓冲区笔记](../../Linux/Linux.md#write() 的缓冲区)

AOF 重写操作原理

1. 调用 fork()，创建一个子进程
2. 子进程把新的 AOF 写到一个临时文件里，不依赖原来的 AOF 文件
3. 主进程持续将新的变动同时写到内存和原来的 AOF 里
4. 主进程获取子进程重写 AOF 的完成信号，往新 AOF 同步增量变动
5. 使用新的 AOF 文件替换掉旧的 AOF 文件



#### 自动触发

* 根据 redis.conf 配置里的 `appendonly yes`



### RDB和AOF的优缺点

RDB优点

* 默认使用LZF压缩算法，适合备份全量数据
* Redis加载 rdb 文件比 aof 文件快，恢复速度快

RDB缺点

* 实时性不够，无法秒级持久化。若Redis挂掉恢复至上次快照，丢失的数据多
* rdb 文件是二进制文件，没有可读性



AOF优点

* 适合保存增量数据，数据不易丢失，**秒级持久化**
* 可读性高

AOF缺点

* 文件体积大，恢复时间长



### RDB-AOF 混合模式

根据 redis.conf 配置里的 `aof-use-rdb-preamble yes`

RDB-AOF 混合模式：先以 RDB 做全量持久化，再以 AOF 做增量持久化



## 5Redis的主从同步机制

一般所有写操作在主节点 master 上进行，读操作在从节点 slave 上进行

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_12_16_59.png" alt="image-20230512165921001" style="zoom:67%;" />

维护集群数据的最终一致性，不要求完全一致，要趋于同步

**全量同步**过程

1. Salve 从节点发送 sync 命令到 Master 主节点
2. Master 执行 BGSAVE 命令，从当前数据中生成一份 RDB 快照文件
3. Master 主节点将保存数据快照期间接收到的写命令记录在缓冲区
4. Master 的 BGSAVE 命令完成后，将 RDB 文件发送给 Salve 从节点
5. Slave 载入接收到的 RDB 文件，即将新的 RDB 文件替换掉旧的 RDB 文件
6. Master 主节点将缓冲区中的所有写命令发送给 Slave 从节点，Slave 从节点执行这些写命令，完成全量同步。



**增量同步**过程

1. Master主节点接收到用户的操作指令，判断是否需要传播到Slave从节点
2. 将操作记录追加到 AOF 文件
3. 将操作传播到其他Slave从节点
   3.1 对齐主从库 (确保从数据库是否是该操作的对应数据库)
   3.2 按Redis协议格式，往响应缓存中写入指令和参数
4. 将缓存中的数据发送给Slave从节点



主从模式的弊端：不具有高可用性，当 master 主节点挂掉将不能写入，所以引入Redis Sentinels



### Sentinel 哨兵模式

是集群管理工具，解决主从同步 Master 宕机后自动进行故障转移的主从切换问题

* 监控：检查主从服务器是否运行正常
* 提醒：通过 API 向管理员或者其他应用程序发送故障通知
* **自动故障转移**：主从切换

集群中的 Sentinels，使用 Gossip 协议通信来接收主服务器是否下线的信息，使用投票协议决定是否执行自动故障迁移以及选择哪个 Slave 从服务器作为 Master 主服务器



#### Gossip流言协议

同步元数据

* 每个节点都随机地与对方通信，最终所有节点的状态达成一致
* 种子节点定期随机向其他节点发送节点列表以及需要传播的消息(去中心化的)
* 不保证信息一定会传递给所有节点，但是最终会趋于一致



## 6Redis集群模式

[文档#创建和使用Redis集群](https://redis.io/docs/management/scaling/#create-and-use-a-redis-cluster)	|	[Reference#Redis集群规范](https://redis.io/docs/reference/cluster-spec/)



**Redis集群TCP端口**

需要打开两个 TCP 端口，如客户端通信端口 6379、集群总线端口16379(一般为port+10000)



**Redis集群数据分片sharding**

分片：在 Redis Cluster 中，整个数据集被分为 16384 个哈希槽(slots)，每个 Redis 节点负责处理一部分槽。再按照某种规则划分数据，将数据分散存储在多个节点上，降低单节点的压力。即按照哈希算法去划分数据，再将不同 key 分散存储在多个节点上。



**一致性哈希算法**

常规的哈希算法划分无法实现节点的动态增减，可以使用一致性哈希算法
一致性哈希算法对于节点的增减，只需要重新定位，换空间中的一小部分数据，具有较好的容错性和扩展性

1 将计算出来的哈希值对 2∧32(节点数)取模，将哈希值空间组织成虚拟的圆环

2 将各服务器的 IP 或主机名作为关键字计算哈希除摩，这样每个服务器就能确定在哈希环上的位置
	节点分布是基于圆环的，无法很好的控制数据分布，Redis Cluster 的槽位空间是自定义分配的，可以自定义大小与位置

3 重定向：将数据的 key 使用相同的函数 Hash 计算出哈希值除摩，顺时针方向的最近节点作为目标存储服务器，实现最小化有损的服务



**Hash 环的数据倾斜问题**

在服务节点少时容易发生，数据分布不均匀

引入虚拟节点解决数据倾斜的问题，具体就是，对服务器节点 key 加编号来计算多个哈希值，通常设置为 32 个或更多

<center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_12_22_01.png" alt="image-20230512220153608" style="zoom:80%;" /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_12_22_06.png" alt="image-20230512220608701" style="zoom:80%;" /></center>



**搭建Redis集群**

```bash
$ redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:7004 to 127.0.0.1:7000
Adding replica 127.0.0.1:7005 to 127.0.0.1:7001
Adding replica 127.0.0.1:7003 to 127.0.0.1:7002
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 54f31c5750e13a8d7aa9839b661013d6edbe014f 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
M: b7539324d270d2c1602c1f1fc421ec0832b2becd 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
M: f33dea68bf27f3755a93b9265e5e5fe08e688355 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
S: a93d4d08447dc9b35a2ddca0b42b74149f5ce998 127.0.0.1:7003
   replicates b7539324d270d2c1602c1f1fc421ec0832b2becd
S: 409643a40dfacef4716acd28d8e922b5c458bf74 127.0.0.1:7004
   replicates f33dea68bf27f3755a93b9265e5e5fe08e688355
S: 79fc470cd296765b9b2350974fdaeb6d1d98774d 127.0.0.1:7005
   replicates 54f31c5750e13a8d7aa9839b661013d6edbe014f
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
..
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: 54f31c5750e13a8d7aa9839b661013d6edbe014f 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 409643a40dfacef4716acd28d8e922b5c458bf74 127.0.0.1:7004
   slots: (0 slots) slave
   replicates f33dea68bf27f3755a93b9265e5e5fe08e688355
M: b7539324d270d2c1602c1f1fc421ec0832b2becd 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: f33dea68bf27f3755a93b9265e5e5fe08e688355 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: a93d4d08447dc9b35a2ddca0b42b74149f5ce998 127.0.0.1:7003
   slots: (0 slots) slave
   replicates b7539324d270d2c1602c1f1fc421ec0832b2becd
S: 79fc470cd296765b9b2350974fdaeb6d1d98774d 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 54f31c5750e13a8d7aa9839b661013d6edbe014f
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

​	

# 我的

## 工具

| Redis 目录下的工具               | **说明**                      |
| -------------------------------- | ----------------------------- |
| redis-server  --help             | 查看启动 redis 服务的帮助     |
| redis-server /path/to/redis.conf | 运行                          |
| redis-cli --help                 | 查看 redis 命令行客户端的帮助 |
| redis-benchmark                  | Redis 性能测试工具            |
| redis-check-aof                  | AOF 文件修复工具              |
| redis-check-dump                 | RDB 文件检查工具              |



## 配置 (重要)

配置文件在 redis 安装目录下(已中文翻译)：`E:\Applications\Scoop\apps\redis\current\redis.conf`

| 常用配置项                    | 示例                                               | 说明                                                         | 参数                                                         |
| ----------------------------- | -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **NETWORK **                  | **网络**                                           |                                                              |                                                              |
| port                          | port 6379                                          | 设置端口号，默认6379                                         |                                                              |
| **GENERAL **                  | **通用**                                           |                                                              |                                                              |
| daemonize                     | daemonize yes                                      | 是否启用后台运行，默认no<br />daemonize：在后台用守护进程的方式启动（作为服务在系统后台运行）<br />前台运行：ctrl + c 直接退出了，服务也关闭了 |                                                              |
| loglevel                      | loglevel notice                                    | 日志等级                                                     |                                                              |
| logfile                       | logfile 日志文件                                   | 设置日志文件                                                 |                                                              |
| databases                     | databases 255                                      | 设置redis数据库总量                                          |                                                              |
| **SNAPSHOTTING **             | **RDB持久化配置**                                  |                                                              |                                                              |
| save \<seconds>  \<changes>   | save 3600 1 300 100 60 10000<br />save "" 禁用快照 | seconds秒后若发生changes次写入则使用bgsave保存数据到磁盘     |                                                              |
| stop-writes-on-bgsave-error   | stop-writes-on-bgsave-error yes                    | 当bgsave出错时，主进程停止写入操作。若有完善的Redis监控系统可以关闭 |                                                              |
| rdbcompression                | rdbcompression yes                                 | 压缩保存，服务器上建议为no。因为redis为CPU密集型服务器，硬盘空间不重要 |                                                              |
| rdbchecksum                   | rdbchecksum yes                                    | 是否开启校验和，有10%的性能影响                              |                                                              |
| dbfilename                    | dbfilename dump.rdb                                | rdb 文件名称                                                 |                                                              |
| dir                           | dir 目录名                                         | 设置rdb和aof存储的磁盘目录                                   |                                                              |
| **SECURITY **                 | **安全**                                           |                                                              |                                                              |
| requirepass                   | requirepass Redis1615                              | 设置默认用户的密码                                           |                                                              |
| **APPEND ONLY MODE**          | **仅追加模式(AOF持久化配置)**                      |                                                              |                                                              |
| appendonly                    |                                                    | 开启 aof 持久化                                              |                                                              |
| appendfilename                | appendfilename "appendonly.aof"                    | AOF的文件名前缀                                              |                                                              |
| appendfsync                   | appendfsync everysec                               | AOF文件同步策略，在可靠性和短阻塞之间取舍                    | always：每个写命令执行完立刻将 aof_buf 缓存写到 AOF 文件。最慢最安全<br />everysec：每秒将 aof_buf 缓存写到 AOF 文件。折中<br />no：由操作系统决定(一般是缓存区满了才写入)。最快不安全 |
| no-appendfsync-on-rewrite     | no-appendfsync-on-rewrite no                       | aof重写时取消aof文件同步(appendfsync no)                     |                                                              |
| auto-aof-rewrite-percentage   | auto-aof-rewrite-percentage 100                    | 自动aof重写操作，如果当前AOF文件的大小超过了上次重写后AOF文件的百分之多少后，就触发重写AOF文件。 |                                                              |
| auto-aof-rewrite-min-size     | auto-aof-rewrite-min-size 64mb                     | 自动aof重写操作，表示执行AOF文件重写操作的AOF文件最小大小。如果AOF文件大小低于这个值，则不会触发重写操作。 |                                                              |
| aof-rewrite-incremental-fsync | aof-rewrite-incremental-fsync yes                  | 每生成4MB数据就同步一次AOF文件                               |                                                              |
| aof-use-rdb-preamble          | aof-use-rdb-preamble yes                           | 开启rdb-aof混合持久化                                        |                                                              |
|                               |                                                    |                                                              |                                                              |



## 客户端|集成库|框架

### Redis CLI及其命令

有不懂的命令查命令手册就好

| redis-cli [OPTIONS] [CMD [arg [arg ...]]] | 释义                 |
| ----------------------------------------- | -------------------- |
| -c                                        | 启动集群模式         |
| -p <port>                                 | 服务器端口(默认6379) |
|                                           |                      |



| redis-cli --cluster <command> [args...] [opts...] 集群管理命令和参数 | 参数                      | 释义                 |
| ------------------------------------------------------------ | ------------------------- | -------------------- |
| create <host1:port1> [host2:port2]                           | --cluster-replicas <args> | 创建集群             |
| reshard                                                      |                           | 重新分配             |
| check                                                        |                           | 检测集群的健康状况   |
| --cluster-yes                                                |                           | 集群以非交互模式运行 |

| 常用命令             | 说明                                                      |
| -------------------- | --------------------------------------------------------- |
| **服务端操作**       |                                                           |
| shutdown             | 同步备份数据集到磁盘，然后关闭Redis服务                   |
| **客户端偏好**       |                                                           |
| :set hints           | 开启在线提示                                              |
| :set nohints         | 关闭在线提示                                              |
| **非数据操作**       |                                                           |
| help                 | 查看 redis cli 命令帮助                                   |
| auth <password>      | 配置文件 requirepass 项，ACL 功能(访问控制列表)的密码验证 |
| ping                 | 测试客户端与服务器的连接是否正常                          |
| clear                | 清理终端                                                  |
| config set key value | 设置行配置                                                |
| info                 | 获取信息或服务器统计数据                                  |
| **数据持久化**       |                                                           |
| save                 | 同步备份数据集到磁盘，建议使用 bgsave                     |
| bgsave               | 异步备份数据集到磁盘                                      |
| lastsave             | 显示上次持久化的时间戳                                    |
| bgrewriteaof         | 异步重写AOF文件到磁盘                                     |

| **快捷键** | 释义 |
| ---------- | ---- |
| tab按键    | 补全 |



wait

同步写入







#### Pipeline

Redis 基于请求/响应模型，对于每个请求处理需要一一应答。Pipeline 批量执行指令，将多个 Redis 命令一次性发送到 Redis 服务器执行，将多次 IO 往返时间节省为一次。

```commands.txt
SET key1 value1
SET key2 value2
GET key1
GET key2
```

```cmd
redis-cli --pipe < commands.txt
```



### Jedis客户端

star11068

Redis简单客户端，采用**BIO**的方式与Redis进行通信，**适合单线程**



### Lettuce客户端

star4993

基于**Netty**的Redis客户端，采用**异步IO**的方式与Redis进行通信

支持**连接池**，多个线程可共享一个连接

支持Redis高级特性，如**Redis哨兵**、**Redis集群**



### Redisson框架

star21113

基于Redis的分布式对象和服务框架，提供了分布式锁、分布式对象、分布式集合等**分布式功能**。



### Spring Data Redis库

star1584

Redis 集成库，提供模板类，底层的 Redis 客户端默认为 Lettuce





# 未整理

# 一、Redis

1. 集群搭建：主从、哨兵、集群
2. 如何维护和mysql的数据一致性



难点在更新维护Redis（实现线程安全）和数据一致性问题，异步机制很难保证100%成功(能通过主从提高成功率)（会导致数据不一致）



过期状态的三种解决方案

1. create_time + exipredTime = expiredTimePoint 可靠
2. 插入时记录expiredTimePoint 字段，和第一钟差不多一样
3. Redis / RocketMQ 更改状态
   
   +容错机制：长时间间隔一个月，离线脚本（令一个程序）低峰时扫描全表，手动补全修改错误状态



一般把更新频率低的数据，放到 Redis 中比如，种类
而且数据量要小，缓存小



## Jedis (todo: 待整理)

文档：https://github.com/redis/jedis

### 依赖

```xml
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
```

### 基本用法

```java
package com.zjbti;

import redis.clients.jedis.Jedis;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @date 2021/1/8 @Created by cxc
 */
public class JedisTestor {

    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6380);
        try {
            jedis.select(2);
            System.out.println("succeed");

            // 字符串
            jedis.set("sn", "771");
            String sn = jedis.get("sn");
            System.out.println(sn);
            jedis.mset(new String[]{"title", "t", "num", "10"});
            List<String> goods = jedis.mget(new String[]{"sn", "title", "num"});
            System.out.println(goods);
            Long num = jedis.incr("num");
            System.out.println(num);

            // Hash键值
            jedis.hset("stu:1", "name", "cxc");
            String name = jedis.hget("stu1", "name");
            System.out.println(name);
            Map studentMap = new HashMap();
            studentMap.put("name", "陈");
            studentMap.put("age", "18");
            studentMap.put("id", "8");
            jedis.hmset("stu:2", studentMap);
            Map<String, String> smap = jedis.hgetAll("stu:2");
            System.out.println(smap);

            // List
            jedis.del("letter");
            jedis.rpush("letter", new String[]{"d", "e", "f"});
            jedis.lpush("letter", new String[]{"c", "b", "a"});
            List<String> letter = jedis.lrange("letter", 0, -1);
            System.out.println(letter);
            jedis.lpop("letter");
            jedis.rpop("letter");
            letter = jedis.lrange("letter", 0, -1);
            System.out.println(letter);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            jedis.close();
        }
    }
}

```

### 业务使用

```java
package com.zjbti;

import com.alibaba.fastjson.JSON;
import redis.clients.jedis.Jedis;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

/**
 * @date 2021/1/8 @Created by cxc
 */
public class CacheSample {

    public CacheSample() {
        Jedis jedis = new Jedis("127.0.0.1", 6380);
        try {
            List<Goods> goodsList = new ArrayList<Goods>();
            goodsList.add(new Goods(8818, "中", "de1", 12f));
            goodsList.add(new Goods(8819, "大", "de2", 13f));
            goodsList.add(new Goods(8820, "小", "de3", 14f));
            jedis.select(3);
            for (Goods goods : goodsList) {
                String json = JSON.toJSONString(goods);
                System.out.println(json);
                String key = "Goods:" + goods.getGoodsId();
                jedis.set(key, json);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            jedis.close();
        }
    }

    public static void main(String[] args) {
        new CacheSample();
        System.out.print("请输入要查询的编号：");
        String goodsId = new Scanner(System.in).next();
        Jedis jedis = new Jedis("127.0.0.1", 6380);
        try {
            jedis.select(3);
            String key = "Goods:" + goodsId;
            if (jedis.exists(key)) {
                String json = jedis.get(key);
                System.out.println(json);
                Goods goods = JSON.parseObject(json, Goods.class);
                System.out.println(goods.getGoodsName());
                System.out.println(goods.getPrice());
            } else {
                System.out.println("不存在，重新输入。");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            jedis.close();
        }
    }
}

```

## Springboot

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

```java
//    键空间通知，keyevent会返回key
    private void sendToRedis(Long oid, Long uid, Long couponId) {
        String key = uid.toString() + "," + oid.toString() + "," + couponId.toString();
        try {
            stringRedisTemplate.opsForValue().set(key, "1", this.payTimeLimit, TimeUnit.SECONDS);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

## DataTables

## Redis键空间通知（keyspace notification）

文档：https://redis.io/docs/manual/keyspace-notifications/
打开Redis配置文件，找到 `notify-keyspace-events` 配置项

---

select 选择数据库



用来监听某些事件，pub/sub 发布/订阅

del 事件
key-space  返回事件类型
key-event  返回名字

expired事件
事件标识：keyevent类型通知E 、Expired过期事件x 、 列表命令l 、集合命令s 
notify-keyspace-events Ex 

配置文件中有命令描述





psubscribe \__keyevent@0__:expired

_表示，订阅 0 号数据库 的\__keyevent@0__:expired名的事件，再kv过期时，key类型通知会打印出key

setex [name second value] 

setex name 10 7yue：表示设置String 类型的kv ：name：7yue过期事件10s

get name

# 二、Redis入门

## 二、Redis 在 Java 客户端的使用

### 1、创建新工程 chinasofti-redis

**在parent上面创建新module**



### 2、导入相关依赖

```xml
<!-- 导入jedis的依赖 -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>

        <!--   日志      -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </dependency>
```



### 3、简单的Demo

```java
package com.chinasofti.jedis;

import redis.clients.jedis.Jedis;

/**
 *  Jedis 连接redis 简单操作
 */
public class JedisDemo {
    public static void main(String[] args) {

        // 1. 创建对象 - jedis  2.获取连接
        Jedis jedis = new Jedis("127.0.0.1",6379);

        // 3.操作
        // 设置键值
        jedis.set("testDemo","123456123456");

        // 获取键值
        String value = jedis.get("testDemo");
        System.out.println(value);

        //4.关闭连接
        jedis.close();


    }

}
```



### 4、创建连接池的demo

```java
package com.chinasofti.jedis;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

/**
 *  jedis 的数据连接池
 */
public class JedisPoolDemo {

    public static void main(String[] args) {
        // 1.创建连接池的配置信息
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        // 设置最大连接数
        jedisPoolConfig.setMaxTotal(100);

        // 2. 获取连接
        // 获取连接池对象
        JedisPool jedisPool = new JedisPool(jedisPoolConfig,"127.0.0.1",6379);
        // 获取连接对象
        Jedis jedis = jedisPool.getResource();

        //3. set / get
        String value = jedis.get("testDemo");
        System.out.println(value);

        // 4. 释放连接
        jedisPool.returnBrokenResource(jedis);
        jedisPool.close();
    }
}
```



### 5、使用集群来实现Jedis

**分片式集群**

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_02_16_17.png" style="zoom:50%;" />





## 三、整合redis实现树菜单的缓存

### 1、为什么要添加缓存？

随着用户的访问越来越多，数据库的读写压力越来越大。添加缓存可以减少数据的压力，从而提高性能

### 2、在什么地方可以使用缓存？

分布式的系统是由很多子系统组成。比如前台系统、后台、用户、订单..........

在后台添加缓存 ： 减少连接数据库，提高后台性能

在前台添加缓存： 减少接口的访问并发调用，减少接口的调用次数，从而提高性能



### 3、整合Redis进系统

3.1 导入依赖

manager-service

## 一、树菜单的Redis缓存

### 1、导入依赖.jar包

```xml
<!--  springboot - redis   -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```



### 2、集成Redis的配置

```yml
# redis 的相关配置
spring:
  redis:
    # 指定是哪个数据库
    database: 0
    # redis 的服务器地址
    host: 127.0.0.1
    # redis 的端口号
    port: 6379
    # redis 密码  注： 当你的redis有密码的时候在去配置
#    password:

    # 配置连接池信息
    jedis:
      pool:
        # 配置最大连接数
        max-active: 200
        # 设置连接池的最大阻塞时间, -1 代表： 一直等，直到连接成功
        max-wait: -1
        # 连接池最大空闲数
        max-idle: 10
        # 连接池最小空闲时间
        min-idle: 0

    # 设置连接超时时间 单位毫秒
    timeout: 1000
```



### 3、设置Redis的序列化方式

```java
package com.chinasofti.manager.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

/**
 *  设置redis的序列化方式
 */
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory factory){
        RedisTemplate<String,Object> template = new RedisTemplate<>();
        // 连接配置工厂
        template.setConnectionFactory(factory);

        // 设置序列化类
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();

        // 设置Key 类型
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);

        // 使用指定的类型
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

        // Key 采用String 类型的序列化
        template.setKeySerializer(stringRedisSerializer);

        // hash 的 Key 采用String类型序列化
        template.setHashKeySerializer(stringRedisSerializer);

        // Value 的序列化方式
        template.setValueSerializer(jackson2JsonRedisSerializer);

        // hash Value 的序列化方式
        template.setHashValueSerializer(jackson2JsonRedisSerializer);

        template.afterPropertiesSet();

        return template;
    }

}
```



### 4、封装Redis 的操作

```java
package com.chinasofti.manager.utils;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.connection.ConnectionUtils;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import java.util.concurrent.TimeUnit;

/**
 *  redis的基本操作封装类
 */
@Component
public class RedisUtils {

    // 获取操作模板
    @Autowired
    private RedisTemplate<String,Object> template;


    /**
     *  指定缓存失效的时间
     */
    public Boolean expire(String key, long time){
        try {
            if (time>0){
                template.expire(key,time, TimeUnit.SECONDS);
            }
            return true;
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 获取 生存/过期 时间
     */
    public long getExpire(String key){
        return template.getExpire(key);
    }

    /**
     *  判断Key是否存在
     */
    public Boolean hasKey(String key){
        try {
            return template.hasKey(key);
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }


    /**
     *  删除缓存
     */
    public void del(String... key){
        if (key !=null && key.length>0){
            template.delete(key[0]);
        }else {
            template.delete(CollectionUtils.arrayToList(key));
        }
    }

    /**
     *  获取Key 的值
     */
    public Object get(String key){
        // 三元表达式
        return key == null?null:template.opsForValue().get(key);
    }

    /**
     *  设置键的值
     */
    public boolean set(String key,Object value){
        try {
            template.opsForValue().set(key,value);
            return true;
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }


    /**
     *  设置键值并且设置生存时间
     */
    public boolean set(String key,Object value,long time){
        try {
            if (time > 0){
                template.opsForValue().set(key,value,time,TimeUnit.SECONDS);
                return true;
            }else {
                template.opsForValue().set(key,value);
                return true;
            }

        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }


    /** 其他还有很多，可以自己尝试做  **/
}
```



### 5、CategoryImpl添加缓存

```java
package com.chinasofti.manager.service.impl;

import com.alibaba.dubbo.config.annotation.Service;
import com.chinasofti.domain.Category;
import com.chinasofti.manager.mapper.CategoryMapper;
import com.chinasofti.manager.utils.RedisUtils;
import com.chinasofti.service.CategoryService;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

@Service
@Component
public class CategoryServiceImpl implements CategoryService {

    @Autowired
    private CategoryMapper categoryMapper;

    @Autowired
    private RedisUtils redisUtils;

    private static final ObjectMapper MAPPER = new ObjectMapper();


    @Override
    public List<Category> selectCategoryList(Category category) {
        /*
        缓存的逻辑：
            第一次进入时，去查询缓存中有没有数据，
            如果有直接提取拿出数据，如果没有就进行数据库查询，
            并且保存到redis中
         */

        List<Category> list = new ArrayList<>();
        try {
            Object tree_menu = this.redisUtils.get("CHINASOFTI_MANAGER_CATEGORY_TREE_MEBU");

            if (tree_menu !=null ){
                // 命中缓存，从缓存中提取数据
                JavaType javaType = MAPPER.getTypeFactory().constructParametricType(ArrayList.class, Category.class);
                list = MAPPER.readValue(tree_menu.toString(),javaType);
            }else {
                // 当前缓存中不存在，从数据库读取数据
                list = this.categoryMapper.selectCategoryList(category);

                // 添加到缓存
                redisUtils.set("CHINASOFTI_MANAGER_CATEGORY_TREE_MEBU",MAPPER.writeValueAsString(list),60*60*24*30);
            }

        }catch (Exception e){
            e.printStackTrace();
        }

        return list;
    }
}
```



### 6、测试效果

![](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_02_16_23.png)