---
title: 关系型数据库
date: 2023-03-29
update: 2024-03-21

categories:
  - 数据库
keywords: MySQL,关系型数据库架构,InnoDB,MyISAM,索引,SQL调优
description: 主要是总结了MySQL8的索引模块和锁模块
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_28_14_30.png
---

# 数据库

[db-engines rank](https://db-engines.com/en/)



# PostgreSQL16.3

[官方文档](https://www.postgresql.org/docs/)



# MySQL8

目前很多 Linux 发行版用 MariaDB
关系型数据库首推postgresql

[Reference](https://dev.mysql.com/doc/refman/8.0/en/)	|	[源码文档](https://dev.mysql.com/doc/dev/mysql-server/latest/)	|	[github源码](https://github.com/mysql/mysql-server)	|	[官站源码](https://downloads.mysql.com/archives/community/)		
TODO: 有空可以搭建 MySQL Debug 环境，调试源码



## 1 关系型数据库管理系统架构

要设计一个关系型数据库，首先要将其划分成两大部分，一个是**存储部分**，该部分类似一个文件系统来将数据持久化到存储设备当中。那光有存储是不行的，还需要有**程序实例**部分来对存储进行逻辑上的管理，而程序实例部分包含将数据的逻辑关系转换成物理存储关系的**1存储管理模块**、优化执行效率的**2缓存模块**、将SQL语句进行解析的**3SQL解析模块**、记录操作的**4日志管理模块**、进行多用户管理的**5权限划分模块**、**6灾难恢复模块**、优化数据查询效率的**7索引模块**，以及使得数据库支持并发操作的**8锁模块**。

![image-20230328143055209](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_28_14_30.png)

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_08_21_53.png" alt="image-20230508215310783" style="zoom:80%;" />

<center>锁</center>

## 2索引模块

如果WHERE子句中使用了索引，那么ORDER BY子句中不会使用索引



### 2.1索引的作用

因为索引能让我们**避免全表扫描，提升检索效率**。(全表扫描将整张表的数据全部或者分批次加载到内存当中，再搜索不仅浪费内存还花费时间)



需要注意的是，索引并不是建立得越多越好：

数据量小的表不需要建立索引，建立会增加额外的**索引开销** (索引提高了查询速度，但不会提高更新表的速度，反而索引滥用会降低更新表的速度，导致更新、插入和删除等操作的性能下降)
**数据变更**需要维护索引，因此更多的索引意味着更多的**维护成本**
更多的索引意味着也需要更多的**空间**
因此，在创建索引时需要根据具体的业务需求和数据库结构来选择合适的索引类型和索引列，避免过多或不必要的索引。



### 2.2索引类型

将能将记录限定在一定查找范围内的具有区分性的字段设置为索引

**普通索引**：用于任何列，不限制列的唯一性

**唯一索引**：限制列的唯一性

**主键索引**：限制列的唯一性且不能为 NULL

**组合索引**(联合索引)：用于多个列之间的联合查询

**全文索引**：用于在大量的文本数据中进行关键字搜索



### 2.3索引的数据结构

**主流是B+-Tree、还有Hash结构、BitMap等**



#### 平衡二叉树或红黑树

无论是平衡二叉树还是红黑树每个节点包含一个关键字，且最多指向两棵子树，因此树的深度很大。
在检索过程中，检索深度每增加1就会发生一次磁盘IO。



#### B-Tree

m阶B树(m>=2)特征：

1. **根节点至少包括两个孩子节点**
2. **除根节点和叶节点外**，其他每个节点**至少有$\lceil m/2 \rceil$个孩子节点**。		
3. 树中每个节点**最多含有m个孩子**
4. 所有**叶子节点**都位于**同一层**
5. 假设每个非叶子结点中包含有n个关键字信息，其中
   1. $K_i(i=1...n)$ 为关键字，且关键字按顺序**升序**排序$K_{i-1} < K_i$。
   2. 关键字的个数n必须满足：$\lceil{m / 2}\rceil - 1 <= n <= m-1$
   3. 非叶子结点的指针：P[1], P[2]，..P[M]，其中P[1]指向关键字小于K[1]的子树，P[M]指向关键字大于K[M-1]的子树，其它**P[i]指向关键字属于 (K[i-1]，K[i]) 的子树**

![image-20230328152021902](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_28_15_20.png)

是一种更适合于应用在外存场景的数据结构

简单理解：m阶B树，每个节点都是多个关键字的有序表，且B树将更多内容放在同一个节点(每个节点都存储数据发生磁盘IO)，对同一个节点的操作转入内存中完成，减少磁盘操作，减少在外存中节点之间转移的次数，以达到提高效率的目的。



#### B+Tree索引

m阶B树(m>=2)特征：

1. **根节点至少包括两个孩子节点**
2. **除根节点和叶节点外**，其他每个节点**至少有$\lceil{m/2}\rceil$个孩子节点**
3. 树中每个节点**最多含有m个孩子**
4. 所有叶子节点都位于同一层
5. 假设每个非叶子结点中包含有n个关键字信息，其中
   1. $K_i (i=1...n)$为关键字，且关键字按顺序**升序**排序$K_{i-1} < K_i$。
   2. 非叶子节点的**子树指针个数 =关键字的个数 n**
   3. 非叶子节点的子树指针**P[i]指向关键字属于 [K[i，K[i+1]) 的子树**
   4. 非叶子节点仅用来索引，**数据都保存在叶子节点中**
   5. 所有**叶子节点**均有一个**链指针**指向下一个叶子结点

![image-20230328153650850](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_28_15_36.png)

B+Tree更适合用来做存储索引
		B+树的磁盘读写代价更低
		B+树的查询效率更加稳定
		B+树更有利于对数据库的扫描，即能优化范围查询

#### Hash索引

![image-20230328155934144](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_28_15_59.png)

是哈希索引是索引键通过哈希运算，再将运算结果的哈希值和行指针信息存放在Buckets中。

缺点：
		仅仅能满足“=”，"IN”，不能使用范围查询。因为哈希值大小关系和键值的不一致，所以**只能等值过滤**。
		无法被用来避免数据的排序操作
		不能利用部分索引键做查询。而B+-Tree支持利用组合索引中的部分索引。
		不能避免表扫描。由于不同索引键可能存在相同哈希值，无法从哈希索引中直接完成查询，还是要访问Bucket中的数据进行实际的比较。
		遇到大量Hash值相等的情况性能并不一定就会比B-Tree索引高

#### BitMap位图索引

![image-20230328160327739](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_28_16_03.png)

### 2.4密集索引和稀疏索引

![image-20230328162600932](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_28_16_26.png)

* 聚集索引文件：它将**表中的每一行数据都映射到一个索引项上**，索引项的数量与表中的行数相同。
  表的物理排列顺序决定了密集索引，所以**一个表只能有一个密集索引**

* 稀疏索引文件：**只包含表中部分行**的索引项，索引项的数量远远小于表中的行数。
  使用非聚集索引**需要两次IO操作**



**MySQL8索引引擎：**

* InnoDB 支持聚集索引 (主键索引) 索引项存储了行数据，也支持稀疏索引 (辅助键索引) 索引项存储了行指针(**主键值**)
  InnoDB 的数据和索引存储在一块，在ibd文件中
* MyISAM 只支持非聚集索引
  MyISAM 的索引存储在MYI，数据存储在MYD

![image-20230328212827291](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_28_21_28.png)



**不同索引树的查找过程：**

![image-20230328162658827](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_28_16_27.png)



#### InnoDB 的聚集索引选取规则：

​		若一个**主键primary**被定义，该主键则作为密集索引
​		若没有主键被定义，该表的**第一个唯一非空索引unique**则作为密集索引
​		若不满足以上条件，innodb内部会生成一个**隐藏主键**(密集索引)
​		非聚集索引(非主键索引)存储相关键位和其对应的主键值，包含两次查找。即非聚集索引(非主键索引)的叶子节点并不存储行数据的物理地址(行记录)，而是存储该行的主键值(行指针)



### 2.5定位并优化慢查询SQL

**第一步：根据慢日志定位慢查询sql**

```sql
SHOW VARIABLES LIKE '%query%';  -- 查看变量
set GLOBAL slow_query_log = ON;  -- 开启慢查询日志
SET GLOBAL long_query_time = 1;  --需要重新连接MySQL

SHOW STATUS LIKE '%slow_queries%';  -- 查看本次会话的慢SQL的条数
```

```log
# 慢日志信息D:\Develop\mysql\mysql-8.0.32-winx64\data\DESKTOP-2DAQ5TI-slow.log
# Time: 2023-03-29T11:54:01.606847+08:00
# User@Host: root[root] @ localhost [::1]  Id:    19
# Query_time: 1.691872  Lock_time: 0.000002 Rows_sent: 1164167  Rows_examined: 2328334
SET timestamp=1680062039;
SELECT `name` FROM person_info_large ORDER BY `name` DESC;
```



**第二步：使用explain关键字等工具了解查询优化器如何执行查询**

![image-20230329120251064](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_29_12_02.png)

* id字段：值越大执行优先级越高

* type字段：其中all表示全表扫描SQL语句需要优化
  执行效率 system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > all

* extra字段：出现以下2项意味着MYSQL根本不能使用索引,效率会受到重大影响。应尽可能对此进行优化。
  * Using filesort：表示MySQL会对结果使用一个外部索引排序,而不是从表里按索引次序读到相关内容。可能在内存或者磁盘上进行排序。MySQL中无法利用索引完成的排序操作称为"*文件排序*”
  * Using temporary：表示 MySQL在对查询结果排序时*使用临时表*。常见于排序order by和分组查询group by

* key字段：表示使用的索引字段 



**第三步：修改sql或者尽量让sql走索引**

- **修改查询语句**：如尽量减少子查询、避免使用 “SELECT *” 等不必要的操作；
- **优化索引**：如增加或删除索引、优化索引顺序等；
- **优化表结构**：如拆分大表、优化表字段类型等；
- **调整MySQL参数**：如调整缓存大小、优化查询缓存、调整连接数等。

```sql
# 可以修改sql使用索引字段
# 可以添加索引 alter table person_info_large add index idx_name(name);
# 可以用FORCE INDEX测试各种索引
SELECT count(id) FROM person_info_large FORCE INDEX(PRIMARY);
```

**查询优化器**

![image-20230329122101922](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_29_12_21.png)

![image-20230329123038070](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_29_12_30.png)

MySQL 的查询优化器会尽可能使用严格索引 (主键索引 唯一索引) 来消除尽可能多的数据行，以减少查询的范围，提高查询效率。

InnoDB中 `count(id)` 使用 account 非聚集索引索引，而不是 id 主键聚集索引。分析：这是因为聚集索引的叶子节点存储完整的行数据，而非聚集索引只存储主键值，因此使用非聚集索引来计算count效果更好，所以MySQL查询优化器使用account索引。



### 2.6组合索引的最左前缀匹配原则

1. **查询优化器使用组合索引时，会一直沿着组合索引的列索引向右匹配直到遇到范围查询**(>、<、between、like)**就停止匹配**。

```mysql
select * from table1 where a = 3 and b = 4 and c > 5 and d = 6;  - 其中a,b,d的顺序可以任意调整

/*  若建立组合索引(a,b,c,d)，则不能使用d索引的
	若建立组合索引(a,b,d,c)，则都可以用到 */
```

2. **对于SQL中的=和in查询列可以乱序**，查询优化器会把SQL优化成索引可以识别的形式。



**最左匹配原则的成因：**

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_01_22_47.png" alt="image-20230501224707174" style="zoom:80%;" />

<center/>col3索引的B+Tree</center>

因为B-Tree 节点内的关键字是有序的
多字段排序：先按照一个字段排序，然后在该字段相同的情况下按照另一个字段排序。所以第一个字段**绝对有序**，之后的字段**范围有序**。

所以对于组合索引`(col1,col2,col3)`，想走col3索引之前必须先走col1和col2索引



## 3锁模块

[Reference#InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)

### 3.1MyISAM与InnoDB关于锁方面的区别是什么

#### MyISAM

* MyISAM 是 MySQL5.1 之前的默认引擎，支持稀疏索引

  * MyISAM **不支持事务**，一条 SQL 跑完解锁

  * MyISAM 默认用的是**表级锁**，不支持行级锁




##### 表级锁

当对数据 SELECT 会自动加上表级读锁 (**S共享锁**)
当对数据 增删改 会自动加上表级写锁 (**X排他锁**)

上共享锁后支持继续上共享锁，不支持排他锁

| Session1\Session2 | S共享锁 | X排他锁 |
| ----------------- | ------- | ------- |
| **S共享锁**       | 兼容    | 冲突    |
| **X排他锁**       | 冲突    | 冲突    |



##### MyISAM 适合的场景

1. **频繁执行全表 count 语句**。因为 MyISAM 用变量保存行数，InnoDB 不保存行数
2. 对数据进行**增删改的频率不高**，查询非常频繁。因为增删改要锁表
3. **没有事务**



#### InnoDB

* InnoDB 是 MySQL5.1 之后的默认引擎支持事务和外键

  * InnoDB **支持事务**，事务 commit 完解锁


  * InnoDB 默认用的是**行级锁**，也支持表级锁 
    当 SQL 用到**索引**则会根据索引值上锁为**行级锁** (锁的细度越细代价越高)，**没有索引**上锁为**表级锁**
    当对数据 select 默认为非阻塞 select 为**快照读** 不加锁、`SELECT * FROM table1 LOCK IN SHARE MODE;  -- 行级共享读锁`为**当前读**




##### 意向锁

**意向锁**是一种**不与行级锁冲突的表级锁**，表示想上锁，为数据行加共享/排他锁之前，InooDB 会先获取该数据行所在在数据表的对应意向锁。
**作用：上表锁时不必逐个检查行锁**

| 意向锁 Session1\Session2              | IS共享意向锁 | IX排他意向锁 |
| ------------------------------------- | ------------ | ------------ |
| IS                                    | 兼容         | 兼容         |
| IX                                    | 兼容         | 兼容         |
| **行级锁与意向锁 Session1\Session2 ** |              |              |
| S行共享                               | 兼容         | 兼容         |
| X行排他                               | 兼容         | 兼容         |
| **表级锁与意向锁 Session1\Session2 ** |              |              |
| S表共享                               | 兼容         | 冲突         |
| X表排他                               | 冲突         | 冲突         |



##### InnoDB适合的场景

1. 数据**增删改查频繁**。因为增删改只锁行
2. 可靠性要求比较高，要求**支持事务**
3. **并发性能更高**



#### 锁的分类

获取锁 acquire、释放锁 release

##### 按锁的粒度划分

* 行级锁
* 页级锁(BDB引擎)
* 表级锁



##### 按锁的级别划分

共享锁(Shared Lock)：读锁

排它锁(Exclusive Lock)：写锁

互斥锁(Mutex Lock)：独占锁	[Java线程笔记#互斥锁](../Java/类库APIs/Java线程.md#互斥锁)



##### 按锁的获取方式划分

自动锁：通过某种机制在特定范围内自动获取和释放锁

显式锁



##### 按锁的用途划分

DML 锁：数据操作语言

DDL 锁：数据定义语言



##### 按并发控制策略划分(默认)

乐观锁和悲观锁可以结合使用：锁升级策略



###### 乐观锁

假设大多数情况下，事务之间不会发生并发冲突。

乐观锁不会阻塞其他事务的读取或写入操作。只在事务提交时检查是否有其他事务对资源进行了修改，若**检测到冲突**则当前事务回滚并**重试**(自旋)。

应用场景：并发读取频繁，并发写入少的场景

优点：不会阻塞读操作、可以减少锁的开销，并发**性能更好**
缺点：不能确保数据的一致性

实现：版本号(时间戳、哈希) 或 CAS(Compare And Swap) [Java线程笔记#CAS](../Java/类库APIs/Java线程.md#CAS)
版本号：version 字段表示数据是否过期，事务提交之前需要先检查 version 再更新 version++



###### 悲观锁

假设大多数情况下，事务之间会发生并发冲突。

悲观锁在访问资源之前需要**获取锁**，并在事务结束之前一直持有锁，会阻塞其他事务对资源的读取或写入。

应用场景：并发写入频繁的场景

优点：能够确保数据的**一致性**
缺点：存在锁开销，并发性能低



### 3.2事务的四大特性 ACID

**transaction：**事务，指一系列要么完全执行要么完全不执行的操作，维护了 ACID 特性。操作没有事务则不提供原子性和隔离性保证。
事务，指完成某个业务功能所需的一系列操作

**ACID：**

* **原子性** (atomicity [əˈtɑːmɪk])：一个事务是一个不可分割的工作单位。
  * 原子性：结构上，我的事务只有两种状态，**全部执行成功和全部回滚** (核心)
  * Spring 事务原子性：结构上，我的事务只有两种状态，执行和不执行，最主要
  * 实现：undo log：回滚日志，记录修改前的“具体值”，实现事务Rollback
  
* **一致性** (consistency [kənˈsɪstənsi])：一事务必须是使数据库从一个一致性状态变到另一个一致性状态。
  * 一致性：逻辑上，要维护数据**逻辑关系的正确和完整**，没有矛盾。在事务开始和结束时，数据应该处于一致的状态。
  * Spring 事务一致性：业务上，不能有不**符合业务规则**的变化，业务要是合理的

* **隔离性** (isolation [ˌaɪsəˈleɪʃn])：指并发事务的隔离程度。多个**并发事务执行时**，它们之间**是相互隔离的**，一个事务的执行不应该受到其他事务的干扰。
  * 隔离性：并发上，两个事务是不能互相干扰的，每个事务看到的数据应该是一致的，**不能出现并发读取数据时，读取到了其他未提交的事务的数据**的情况。通过加锁解决。MySQL具体分为4种隔离级别
  * Spring 事务隔离性：**并发时**，两个事务**互不干扰**的。每个事务看到的数据应该是一致的，不能出现并发读取数据时，读取到了其他未提交的事务的数据的情况。通过加锁解决。Spring事务具体分为5种隔离级别

* **持久性** (durability [ˌdʊrəˈbɪləti])：一旦事务提交成功，它对数据库中数据的改变就应该是永久性的。
  * 持久性：一旦事务提交成功，其所做的修改将会**永久保存在数据库中**，即使在之后的时间里发生了系统故障或断电等异常情况，数据也不应该丢失。InnoDB 使用 **redo log(重做日志) 写前日志**记录数据库修改操作，在数据库故障时重新执行保证了持久性。
  * Spring 事务持久性，一旦事务**提交成功**，其结果要同步到**持久化**控件中
  * 实现：redo log：重做日志，记录修改后的“具体值”

|            | Redo log 数据库写操作日志                                    | Bin log 逻辑日志                                             |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 应用场景   | 崩溃恢复未落盘的数据，保证已提交事务的持久性                 | 主从复制和数据恢复                                           |
| 实现方式   | redo log是InnoDB引擎实现的，并不是所有引擎都有。             | bin log是Server层(SQL语句分析、优化)实现的，所有引擎可用     |
| 记录的方式 | redo log采用循环写的方式记录数据，日志记录落盘后会被覆盖掉，有写指针和数据落盘指针 | bin log通过追加的方式记录，当文件大小大于max_binlog_size值后，后续的日志会记录到新文件上。 |
| 文件大小   | redo log的大小是固定的                                       | 通过max_binlog_size配置                                      |
| 提交时机   | 逐步写入磁盘                                                 | 一次性                                                       |
|            | innodb_flush_log_at_trx_commit<br />0 - 延迟写，崩溃时丢失1秒数据<br />1 - 实时写，实时刷<br />2 - 实时写，延迟刷，崩溃时丢失1秒数据 |                                                              |

两段提交，写完redo log再写binlog，任何一次提交失败都回滚，保证持久性




#### 并发访问问题

| 并发问题           | 释义                                                         | 如何避免                                                     |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **更新丢失**       | 指多个事务对同一数据进行更新时，其中一个事务的**更新被**另一个事务的更新所**覆盖**，导致数据的更新结果丢失的情况。 | 通过互斥锁解决                                               |
| **读操作并发问题** |                                                              |                                                              |
| **脏读**           | **读未提交**                                                 | 1. RC RR SERIALIZABLE<br />通过只能读持久化数据(快照读)不能读缓存解决 |
| **不可重复读**     | 指在同一事务中，对同一数据进行**多次查询，得到的结果不一致**的情况。侧重于对同一事物的修改，**读取与修改不隔离** | 1. RR SERIALIZABLE (只有第一次快照读会创建一个 Read View 读视图，避免了上锁，推荐)<br />2. 使用行级共享读锁**锁行**(不建议) |
| **幻读**           | 指在同一事务中，对同一数据进行**多次查询，得到的结果不一致**的情况。侧重于新增或者删除，**读取与增删不隔离** | 1. SERIALIZABLE <br />2. RR 使用`SELECT * FROM account_innodb LOCK IN SHARE MODE;`上行级共享读锁**锁表** |
|                    |                                                              |                                                              |

**更新丢失**

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_08_16_43.png" alt="image-20230508164334907" style="zoom: 50%;" />

```mysql
START TRANSACTION;  -- 1
UPDATE `account innodb` SET balance = balance+1 where id = 1;  -- 1
COMMIT;  -- 2
------------------------------------------------------------------------------------
START TRANSACTION;  -- 1
UPDATE `account innodb` SET balance = balance+2 where id = 1;  -- 1
ROLLBACK;  -- 3
```

**脏读**

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_20_10_29.png" alt="1602415650743" style="zoom:50%;" />

```mysql
START TRANSACTION;  -- 1
SELECT * FROM `account innodb` WHERE id = 1;  -- 2
COMMIT;  -- 3
--------------------------------
START TRANSACTION;  -- 1
UPDATE `account innodb` SET balance = balance+1 WHERE id = 1;  -- 1
ROLLBACK;  -- 3
```

**不可重复度**

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_20_10_31.png" alt="1602415667068" style="zoom:50%;" />

```mysql
-- 不可重复读
START TRANSACTION;  -- 1
SELECT * FROM `account innodb` WHERE id = 1;  -- 1  -- 快照读
SELECT * FROM `account innodb` WHERE id = 1;  -- 3
COMMIT;  -- 3
--------------------------------
START TRANSACTION;  -- 1
UPDATE `account innodb` SET balance = balance+1 WHERE id = 1;  -- 2
COMMIT;  -- 2
-- 使用RR隔离级别下，解决不可重复读，此时读快照，改当前读
```

**幻读**

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_20_10_30.png" alt="1602415694379" style="zoom:50%;" />

```mysql
-- RC下发生幻读
START TRANSACTION;  -- 1
SELECT * FROM `account innodb`;  -- 1
UPDATE `account innodb` SET balance = 100;  -- 3  
SELECT * FROM `account innodb`;  -- 1  当前读之后再执行快照读，会发生幻读，多修改了一条数据
COMMIT;  -- 3 
-- reaptable-read 下避免幻读
START TRANSACTION;  -- 1
SELECT * FROM `account innodb` LOCK IN SHARE MODE;  -- 1  -- 行级共享读锁，避免幻读
UPDATE `account innodb` set xx=xx;  -- 3 
SELECT * FROM `account innodb`;
COMMIT;  -- 4
-- Sserializable下解决幻读
START TRANSACTION;  -- 1
SELECT * FROM `account innodb`;  -- 1
COMMIT;  -- 3
--------------------------------
-- 幻读
START TRANSACTION;  -- 1
INSERT INTO `account innodb` VALUES(14, "newman", 500);  -- 2
select * FROM `account innodb`;
COMMIT;  -- 2
```



#### 事务隔离级别(隔离性的具体体现)

| 事务隔离级别                         | 脏读 | 不可重复度 | 幻读                                                         | 实现方式 (伪MVCC 机制)                                       |
| ------------------------------------ | ---- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **RU** read-uncommitted              | 会   | 会         | 会                                                           |                                                              |
| **RC** read-committed (Oracle默认)   |      | 会         | 会                                                           | 在 RC 级别事务的执行过程中，**每次快照读会创建新的 Read View 读视图** (Read View 决定了能读取哪个版本的数据快照)，所以是读已提交且会发生不可重复读。 |
| **RR** repeatable-read (MySQL默认)   |      |            | 会(部分场景可避免)<br />`SELECT * FROM table1 LOCK IN SHARE MODE;  -- 行级共享读锁 锁表`在查询时会对相应的记录加锁，阻止其他事务的增删操作(解决幻读) | 在 RR 级别事务的执行过程中，**只有第一次快照读会创建一个 Read View 读视图** (Read View 决定了能读取哪个版本的数据快照) (所以RR级别下什么时候首次使用快照读很重要)，其他并发事务对于已经存在的数据进行修改时并不会影响到当前事务的快照，所以不会发生不可重复读。但是其他并发事务对于当前事务还不存在的数据进行修改则会影响到当前事务的快照，所以会发生幻读。<br />**Gap 锁**在部分场景避免了幻读<br /> |
| **serializable** [/ˈsɪˌriəˌlaɪzəbl/] |      |            |                                                              |                                                              |

```mysql
-- 事务隔离级别
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT @@GLOBAL.transaction_isolation, @@transaction_isolation, @@session.transaction_isolation;
```



#### 3.3当前读和快照读

* 当前读：读最新版本数据

**加锁的** 阻塞的 

通过 next-key 锁实现，解决幻读问题

```mysql
select * from table1 lock in share mode  -- 行级共享锁
select * from table1 for update  -- 行级排他锁
update、delete、insert
```



* 快照读：读事务快照版本数据

**不加锁的** 非阻塞的 

通过 Read View 实现，解决读操作并发问题。

```mysql
select * from table1  // RU、RC、RR级别下
```



##### 3.4Undo撤销日志 + Read View 实现快照读

next-key 锁

undo log：回滚日志，记录修改前的“具体值”，实现事务Rollback

**快照读的实现：**

| 数据行隐藏字段 | 释义                          |
| -------------- | ----------------------------- |
| DB_TRX ID      | 事务ID，越新开启的事务ID越大  |
| DB_ROLL_PTR    | 回滚指针指向旧版本的 Undo Log |
| DB_ROW_ID      | 行ID，隐藏备用主键字段        |

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_08_22_29.png" alt="image-20230508222937952" style="zoom: 80%;" />

* Undo Log 链：**记录数据修改之前的副本**(因为多版本记录是串行的，不是共存的，所以是伪 MVCC)
  * Insert Undo Log 链，是在**insert**操作中产生的 Undo Log。因为insert操作的记录只对当前事务本身可见，对于其它事务此记录是不可见的，所以insert undo log在事务提交后直接删除。
  * Update Undo Log 链，是在update/delete操作中产生的 Undo Log。事务回滚时需要，快照读需要，在快照不涉及该日志才删除

* Read View 读视图：是一个逻辑数据结构，它包含了已提交事务ID列表和当前活跃事务ID列表，根据已提交事务ID列表来决定读取哪个版本的数据快照(使用可见行算法来判断数据行是否对该事务是可见的)。Read View 在事务开始时创建，快照读取时更新。
  * 可见性算法：若数据行的事务ID大于当前Read View事务ID，则说明数据行是在当前事务开始之后提交的对当前事务不可见，通过DB_ROLL_PTR指针取出Undo Log，直到数据行的事务ID小于等于当前Read View事务ID。



在**伪 MVCC 多版本并发控制机制**中，在事务开始时创建一个初始版本的 Read View 读视图用于记录已提交事务ID列表和当前活跃事务ID列表。当事务修改一条数据时，它会创建该数据的新版本，而不是修改现有的数据，这个新版本会被记录一个新的版本号。当事务读取数据时，会根据已提交事务ID列表和记录的版本号(事务ID)来判断是否可以读取该记录。

在 RC 级别事务的执行过程中，**每次快照读会创建新的 Read View 读视图** (Read View 决定了能读取哪个版本的数据快照)，所以是读已提交且过程中，**只有第一次快照读会创建一个 Read View 读视图** (Read View 决定了能读取哪个版本的数据快照) (**所以RR级别下什么时候首次使用快照读很重要**)，其他并发事务对于已经存在的数据进行修改时并不会影响到当前事务的快照，所以不会发生不可重复读。**但是其他并发事务对于当前事务还不存在的数据进行修改则会影响到当前事务的快照，所以会发生幻读。**

MVCC作用：读操作不加锁，**用于实现读写并发和事务的隔离性**



##### 3.5当前读的next-key锁

表象：基于伪 MVCC 机制实现快照读

内在：RR 级别下实现了**next-key锁** (record记录锁 + Gap锁(用于防止插入))，从而**在RR级别下部分场景避免了幻读**

 

RR级别下的走**主键索引**或**唯一索引**的**当前读**的next-key锁机制：

> InnoDB RR级别主要通过引入 next-key 锁来避免下幻读问题
>
> next-key 锁由 record 锁和 Gap 锁组成
>
> Gap 锁会用在：非唯一索引或者不走索引的当前读，以及[仅命中检索条件的部分结果集]并且[用到主键索引或唯一索引]的当前读中。

* 若where条件**全部命中**，则只会上行级锁 (若是非主键索引，则需要对当前索引和主键索引都上锁)
  因为语义上全命中且唯一，所以不需要 Gap 锁

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_09_17_54.png" alt="image-20230509175429758" style="zoom:67%;" />

* 若where条件**部分命中**或者**全不命中**，则会上行级锁和 (左开右闭区间) Gap锁



RR级别下走**非唯一索引**或**不走索引**的**当前读**的next-key锁机制

* **非唯一索引**：上Gap锁，且使用主键达到更准确的区间

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_09_16_14.png" alt="image-20230509161416233" style="zoom:80%;" />

<center/>Gap 锁区间: [6c,9b) [9b9d) [9d,11f) (使用主键达到更准确的区间)</center>

```mysql
insert into tb1 values('bb', 6);  -- success
insert into tb1 values('cc', 6);  -- block
insert into tb1 values('ee', 11);  -- block
insert into tb1 values('ff', 11);  -- success

utf8mb4_0900_ai_ci的排序规则：1, ab, abc, ac, b, bb, c
```

* **不走索引**：对所有Gap上锁=锁表 (**这种情况需要避免，代价太大**)

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_09_16_12.png" alt="image-20230509161203585" style="zoom:67%;" />



## 4语法SQL

### 系统变量和状态变量

```mysql
show [global|session(default)] {variables|status} [like '%str%']
set [global|session(default)] {variable_key} = {variable_value}
```

https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html

| Variable              | 释义                                                      |
| --------------------- | --------------------------------------------------------- |
| autocommit            |                                                           |
| character_set_client  | 设置mysql返回的字符编码，解决cmd和mysql编码不同而中文乱码 |
| long_query_time       | 需要重新连接客户端                                        |
| slow_query            |                                                           |
| slow_query_log        | MySQL服务开启慢查询日志                                   |
| transaction_isolation | 查询全事务隔离级别                                        |
|                       |                                                           |

https://dev.mysql.com/doc/refman/8.0/en/server-status-variables.html

| Status         | 释义                           |
| -------------- | ------------------------------ |
| engines        | 查看服务器支持的引擎           |
| %query%        | 模糊查询和slow_query有关的变量 |
| %slow_queries% | 查看本次会话的慢SQL的条数      |
|                |                                |



### 4.1DML

#### 注释

```sql
-- 行注释
/* 块注释 */
```



#### SELECT

##### 关联查询

笛卡尔积CROSS JOIN：将每个表的每一行与其他表的每一行组合
连接查询 = 笛卡尔积 + where条件过滤
所以实际项目中建议使用单表查询再用 Java 逻辑封装(将计算放到Service层减轻数据库压力、减少锁的竞争、解耦更容易对数据库扩展)或者采用不超过三次的显式join查询(因为没索引关联结果会爆炸式增长)
需要保证被关联的字段需要有索引

1隐式内连接：`from tb_n1,tb_n2 where tb_n1.id = tb_n2.id`，

2内连接：`tb_n1 join tb_n2 on tb_n1.id = tb_n2.id`

3左外连接：`tb_n1 left join tb_n2 on tb_n1.id = tb_n2.id`：返回左表中的所有行，以及与右表匹配的行，如果在右表中没有匹配的行，则右表的列将为 NULL。

4右外连接：`tb_n1 right join tb_n2 on tb_n1.id = tb_n2.id`：返回右表中的所有行，以及与左表匹配的行，如果在左表中没有匹配的行，则左表的列将为 NULL。

```mysql
select se.*, stu.name sname, cou.name cname from selection se left join student stu on se.student=stu.id left join course cou on se.course=cou.id where course = xxx
```



###### 笔试

[题目1](https://www.nowcoder.com/practice/fc7344ece7294b9e98401826b94c6ea5?tpId=82&tqId=29773&rp=1&ru=/exam/interview&qru=/exam/interview&sourceUrl=%2Fexam%2Finterview%3Forder%3D0&difficulty=5&judgeStatus=undefined&tags=&title=)：查找在职员工自入职以来的薪水涨幅情况

```sql
-- 内连接方式：两种时间空间差不多，但这种更直观
SELECT a.emp_no, (b.salary - c.salary) as growth
FROM employees AS a
INNER JOIN salaries AS b ON a.emp_no = b.emp_no AND b.to_date = '9999-01-01'
INNER JOIN salaries AS c ON a.emp_no = c.emp_no AND a.hire_date = c.from_date
order by growth;
-- 即
SELECT a.emp_no, (b.salary - c.salary) as growth
FROM employees AS a, salaries AS b, salaries AS c where a.emp_no = b.emp_no AND b.to_date = '9999-01-01' and a.emp_no = c.emp_no AND a.hire_date = c.from_date
order by growth;
```

```sql
-- 自查询方式
select now_t.emp_no, now_sal - old_sal as growth
from (
    select emp.emp_no, salary as now_sal
    from employees emp join salaries as sal on emp.emp_no = sal.emp_no
    where to_date = '9999-01-01'
) as now_t join (
    select emp.emp_no, salary as old_sal
    from employees emp join salaries as sal on emp.emp_no = sal.emp_no
    where from_date = hire_date
) as old_t on now_t.emp_no = old_t.emp_no
order by growth;
```

[题目2](https://www.nowcoder.com/practice/f858d74a030e48da8e0f69e21be63bef?tpId=82&tqId=29777&rp=1&ru=/exam/interview&qru=/exam/interview&sourceUrl=%2Fexam%2Finterview%3Forder%3D0&difficulty=5&judgeStatus=undefined&tags=&title=)：获取员工其当前的薪水比其manager当前薪水还高的相关信息

```sql
select emp.emp_no, dept.emp_no, e_sal.salary, m_sal.salary
from dept_emp as emp 
inner join dept_manager as dept on emp.dept_no = dept.dept_no and emp.emp_no != dept.emp_no
inner join salaries as e_sal on emp.emp_no = e_sal.emp_no
inner join salaries as m_sal on dept.emp_no = m_sal.emp_no
where e_sal.salary > m_sal.salary;
-- 即
select emp.emp_no, dept.emp_no, e_sal.salary, m_sal.salary
from dept_emp as emp, dept_manager as dept, salaries as e_sal, salaries as m_sal
where emp.dept_no = dept.dept_no and emp.emp_no = e_sal.emp_no and dept.emp_no = m_sal.emp_no and emp.emp_no != dept.emp_no and e_sal.salary > m_sal.salary;
```

```sql
select emp_no, manager_no, emp_salary, manager_salary
from (
    select emp.emp_no, salary as emp_salary, emp.dept_no
    from dept_emp as emp
    inner join salaries as sal 
    on emp.emp_no = sal.emp_no) as t1
inner join (
    select dept.emp_no as manager_no, salary as manager_salary, dept.dept_no
    from dept_manager as dept
    inner join salaries as sal
    on dept.emp_no = sal.emp_no) as t2
on t1.dept_no = t2.dept_no and emp_no != manager_no
where emp_salary > manager_salary;
```

体会：对于小表**使用关联查询更简洁**，子查询麻烦

[题目3](https://www.nowcoder.com/practice/ea0c56cd700344b590182aad03cc61b8?tpId=82&tqId=35088&rp=1&ru=/exam/oj&qru=/exam/oj&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D240&difficulty=5&judgeStatus=undefined&tags=&title=)：牛客每个人最近的登录日期(五)

```sql
-- 我写的，复杂还错了
select t3.date, ifnull(round(t2.new_user_num/t1.next_new_user_num, 3), 0)
from
    (select l1.date, count(l1.user_id) as next_new_user_num  # 日期，新用户数量
    from login as l1
    where (l1.date, l1.user_id) in (
        select min(l2.date), l2.user_id
        from login as l2
        group by l2.user_id)
    group by l1.date) as t1 
inner join
    (select today.date, count(today.user_id) as new_user_num  # 该子查询，有问题没确保是昨天新用户，只是昨天登录过的用户
    from login as today
    inner join login as tomorrow
    on today.user_id = tomorrow.user_id and datediff(tomorrow.date, today.date) = 1
    group by today.date) as t2
on t1.date = t2.date
right join
    (select l3.date
    from login as l3
    group by l3.date) as t3
on t3.date = t1.date
```

```sql
select t0.date,
ifnull(round(count(distinct t2.user_id)/(count(t1.user_id)),3),0)
from
(
    select date
    from login
    group by date
) t0
left join
(
    select user_id,min(date) as date
    from login
    group by user_id
)t1
on t0.date=t1.date
left join login as t2
on t1.user_id=t2.user_id and datediff(t2.date,t1.date)=1
group by t0.date
```

体会：要联表查询不要子查询，需要哪些信息就多left join进去，最后group by统计



##### group by 和 having 和聚合函数

**group by**

每行为一个**分组**，配合聚合函数使用

* `sql_mode=only_full_group_by`要求 SELECT 列表中**检索的列必须为GROUP BY子句中的列或使用聚合函数进行计算**
  使用其他模式时会自动选一个值

* **聚合函数**对于group by子句定义的每个组各返回一个结果

**having**

对每个分组进行**聚合过滤**

* 通常与 group by 子句一起使用
* where**过滤行**，having**过滤组**
* 出现在同一SQL中的前后顺序：where > group by > having

**聚合函数**

`sql_mode=only_full_group_by` 模式

| 聚合函数                      |                            |      |
| ----------------------------- | -------------------------- | ---- |
| min()                         |                            |      |
| max()                         |                            |      |
| count(Sname) <br/>count(常量) | 计数非空行<br />计数每一行 |      |
| avg()                         |                            |      |



###### 笔试

题目1：找出平均分在90以上且没有一门课在80分以下的学生学号

```sql
select Sno 
from S, SC where S.Sno=SC.Sno
group by S.Sno 
having avg(grade) >= 90 and min(grade) >= 80;
```

题目2：查询没有学全所有课的同学的学号、姓名

```mysql
SELECT stu.student_id, stu.`name`, COUNT(course_id)
FROM student stu, score s WHERE stu.student_id = s.student_id
GROUP BY stu.student_id, stu.`name`
HAVING COUNT(course_id) < (
	SELECT COUNT(course_id) FROM course
)
```



##### 笔试

题目1：在学生表S表中删除既不是物理系、信息系也不是计算机系的学生

```sql
delete from S where Sdept not in ('物理系',''信息系','计算机系');
```

题目2：找出所有不姓张的学生的姓名、性别及系科

```sql
select Sname, Ssex, Sdept from S where Sname not like '张%';
```



##### 相关关键字

| 查询子句                      | 释义                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| distinct                      |                                                              |
| limit n,m                     | 表示跳过n条后的m条记录                                       |
| order by xx [asc(默认)\|desc] | 降序                                                         |
| like '%%'                     | %多个、_一个                                                 |
| in (value1,value2)            |                                                              |
| between value1 and value2     |                                                              |
| all()                         | 常与<,>,=,!=配合使用                                         |
| any()                         | 常与<,>,=,!=配合使用                                         |
|                               |                                                              |
| union                         | 并集，将两个查询结果合并成一个结果集，要求列数和数据类型兼容，且不会列出重复的行 |
| union all                     | 并集，会列出重复的行                                         |
| INTERSECT                     | 交集                                                         |
| EXCEPT                        | 差集                                                         |
|                               |                                                              |
| exists()                      | 检查子查询是否返回任何行                                     |

##### 窗口函数

| 窗口函数                               | 释义                                       |
| -------------------------------------- | ------------------------------------------ |
| row_number()                           |                                            |
| dense_rank() over(order by score desc) | 密集排序                                   |
| now()                                  | 返回datetime类型的 `'YYYY-MM-DD HH:MM:SS'` |
| DATEDIFF(d1,d2)                        |                                            |
| ifnull(xxx, 0)                         | 若xxx=null，则返回0                        |



#### INSERT

报错

```sql
insert into -- 如果表中已存在具有相同主键或唯一索引的行，则报错
```

修改

```sql
INSERT INTO employees (emp_no, first_name, last_name)
VALUES (1, 'John', 'Doe')
ON DUPLICATE KEY UPDATE first_name = 'John', last_name = 'Doe';  -- 如果表中已存在具有相同主键或唯一索引的行，则UPDATE
```

```sql
replace into  -- 如果表中已存在具有相同主键或唯一索引的行，则将旧行删除，并插入新行；不存在同INSERT INTO
```

忽略

```sql
insert ignore  -- 如果表中已存在具有相同主键或唯一索引的行，则忽略插入操作而不报错；不存在同INSERT INTO
```

##### 笔试

```java
-- 批量插入
insert into actor values(1,'PENELOPE','GUINESS','2006-02-15 12:34:33'),(2,'NICK','WAHLBERG','2006-02-15 12:34:33');
```

```sql
-- 了解
insert into table_name values(c1,c2),(c11,c22) on duplicate key update c1=123,c2='xx';
replace into
insert ignore 
```





#### UPDATE

```mysql
update tb_user set password = password('123456'),info = '123' where user = 'root';  --password()密码加密方法
```



#### DELETE和TRUNCATE

```mysql
DELETE FROM tb_name WHERE id = 1;
```

```sql
TRUNCATE TABLE table_name;  -- 快速清空表中的所有数据(比DELETE FROM高效)，还能重置自增计数器
```



##### 笔试

[题目](https://www.nowcoder.com/practice/3d92551a6f6d4f1ebde272d20872cf05?tpId=82&tqId=29810&rp=1&ru=/exam/oj&qru=/exam/oj&sourceUrl=%2Fexam%2Foj%3Fdifficulty%3D5%26page%3D1%26pageSize%3D50%26search%3D%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82&difficulty=undefined&judgeStatus=undefined&tags=&title=%E5%88%A0%E9%99%A4)：删除emp_no重复的记录，只保留最小的id对应的记录。

```sql
-- 错误写法：SQL_ERROR_INFO: "You can't specify target table 'titles_test' for update in FROM clause(子句)"
-- 连续两条from同张表
delete from titles_test 
where (id, emp_no) not in (    
        select min(id) as min_id, emp_no
        from titles_test
        group by emp_no
);
```

**mysql不允许在子查询的同时删除原表数据**

```sql
-- 正确写法
delete from titles_test 
where (id, emp_no) not in (    
    select min_id, emp_no from (
        select min(id) as min_id, emp_no
        from titles_test
        group by emp_no
    ) as t1
);
```



#### index

```mysql
CREATE INDEX index_name ON table_name (column1, column2, ...);
```

主键索引：id
唯一索引：身份证
普通索引：name
全文索引：大文本
组合索引



where ， join ， <，<=，=，>，>=，BETWEEN，IN，以及某些时候的 LIKE(% 和 _ ) 



建立一个联合索引 (col1, col2, col3) 等于实际建立了(col1)、(col1,col2)、(col,col2,col3)三个索引。

在检索数据时从联合索引的最左边开始匹配





#### 事务

```mysql
begin;  -- 显示地开启一个事务
start transaction;
commit;  -- 提交事务，并使已对数据库进行的所有修改变为永久性的
rollback;  -- 回滚事务，并撤销正在进行的所有未提交的修改
```



#### 锁

```sql
LOCK TABLES person_info_myisam READ;  -- 读锁，共享锁
LOCK TABLES person_info_myisam WRITE;  -- 写锁，排他锁
LOCK TABLES person_info_myisam READ|WRITE;
UNLOCK TABLES;  -- 释放锁

SELECT * FROM person_info_large WHERE id = 3;  -- 默认是非阻塞 SELECT
SELECT * FROM person_info_myisam WHERE id BETWEEN 1 AND 1000000 FOR UPDATE;  -- FOR UPDATE上行级排他锁
SELECT * FROM person_info_large WHERE id = 3 LOCK IN SHARE MODE;  -- LOCK IN SHARE MODE上行级共享读锁
```



### 4.2DDL

drop、create、alter

#### 建库建表

```mysql
create database db_name default charset uft8;

CREATE TABLE table_name (
    column1 datatype constraint [asc|desc],
    column2 datatype constraint [asc|desc],
    ...
    CONSTRAINT constraint_name
    constraint_type (column1, column2, ...)
);

-- 一般在所有的表创建完成后，再用alter table添加约束的方式创建外键，不用考虑创建表的sql顺序
alter table tb_name1
add constraint fk_name1_name2
foreign key(name1id) references tb_name2(name2id) on delete cascade on update cascade;
```



##### 数据类型

整数
tinyint(1)，只占1个字节，适合存储存储布尔值、状态标志、开关状态。务必设置长度为1(否则 tkMybatis生成Byte)

浮点数
decimal(p, s)：p表示总位数，s表示小数部分的位数。由于浮点数float/double有误差，所以对于需要精确计算的数据(金额)应使用该类型

日期/时间
date：'YYYY-MM-DD'，时间或日期需要用单引号或双引号进行包裹，作为字符串的特殊值解析处理
datetime：'YYYY-MM-DD HH:MM:SS'，时间或日期需要用单引号或双引号进行包裹，作为字符串的特殊值解析处理

字符串
char()：固定长度。会使用空格填充剩余的空间，取数据的时候需要用 trim() 去掉多余的空格。查询效率更高
varchar()：长度可变。动态分配存储空间，更节省空间
nchar()：在 char() 的基础上，支持 Unicode 字符集
nvarchar()：在 varchar() 的基础上，支持 Unicode 字符集

二进制



##### 约束类型

主键约束：`PRIMARY KEY(column1, column2, ...)`

外键约束：`FOREIGN KEY(column1, column2, ...) references tb_name(col_name) on delete cascade on update cascade;`

唯一约束：`UNIQUE`

非空约束：`NOT NULL`

默认值约束：`DEFAULT defaultvalue`

自动递增约束：`AUTO_INCREMENT`



##### 外键

并发应用不推荐使用外键：
	数据库服务器很容易成为性能瓶颈且不容易轻易地水平扩展，使应用受 IO 能力限制
	外键需要额外的维护，更消耗资源，还会因为需要请求其他表而对其他表加锁，容易出现死锁情况。
	可以不使用外键级联操作，而使用事务控制数据一致性，让应用服务器承担此部分的压力



外键的级联操作(不推荐使用)：
	如果父表中的记录被更新/删除，则子表中对应的记录自动被更新/删除

```mysql
alter table tb_staff add constraint fk_staff_dep foreign key(did) references department(id) 
on delete cascade on update cascade;
```

外键的级联更新和级联删除：

* on update 操作类型：是指在主键表中被参考字段的一条记录的值更新时，进行某种操作

* on delete 操作类型：是指在主键表中被参考字段的一条记录的值被删除时，进行某种操作

操作类型：

* no action 表示 不做任何操作，
* set null    表示在外键表中将相应字段设置为null
* set default 表示设置为默认值
* cascade 表示级联操作，就是说，如果主键表中被参考字段更新，外键表中也更新，主键表中的记录被删除，外键表中改行也相应删除



### 4.3其他SQL(DCL)

grant、revoke、commit、rollback

| 其他SQL                                                      | 释义                                      |
| ------------------------------------------------------------ | ----------------------------------------- |
| **切换数据库**                                               |                                           |
| show databases;                                              | 显示所有数据库的名称                      |
| use 数据库名;                                                | 使用数据库                                |
| select database();                                           | 显示当前数据库的名称                      |
| **权限**                                                     |                                           |
| GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "root"; | 为root添加远程连接的能力，链接密码为 root |
| **数据库/表结构**                                            |                                           |
| show create database db_name;                                | 显示指定数据库的名称和创建语句            |
| show tables;                                                 | 显示当前所在数据库中的所有表名            |
| desc tb_name; \| SHOW COLUMNS FROM db_name.tb_name;          | 显示指定表的设计视图                      |
| **用户**                                                     |                                           |
| select user();                                               | 显示当前用户名                            |
| **视图**                                                     |                                           |
| CREATE VIEW view_s AS SELECT * FROM STU WHERE 性别='男'      | 创建视图                                  |



## 5理论范式

范式理论：为了规范化关系数据库模式，减少数据冗余和依赖关系，提高数据的一致性和完整性。

1NF：关系表中的每个属性必须是原子的，**不可再分**。

2NF：在满足第一范式的基础上，**非主键属性**不能依赖于主键的部分属性，而只能**依赖于整个主键**。(要求消除部分函数依赖)

3NF：在满足第二范式的基础上，所有非主键属性都**不传递依赖**于候选键

BCNF(推荐)：在满足第二范式的基础上，所有非平凡函数依赖都是候选键的超键，即**非主键属性完全依赖于候选键**。(要求消除所有非平凡函数依赖)



候选键：指能够唯一标识关系表中每个元组的属性组合。一个关系表可以有多个候选键

主键：主键是在候选键中选择的一个，用于确保数据的唯一性和完整性。一个关系表只能有一个主键

函数依赖：描述了属性组合对令一属性组合的决定关系。
	A是平凡的函数依赖，A与B的值相同
	A是非平凡的函数依赖，B依赖于A，A决定了B的值



有时候我们会有反范式的设计，为了提高数据库查询的性能



# 我的

## 配置

[Reference#option-files](https://dev.mysql.com/doc/refman/8.0/en/option-files.html)

**Windous 下配置文件：**D:\Develop\mysql\mysql-8.0.32-winx64\my.ini

```ini
# MySQL Server Instance Configuration File  # 需要指出的是MySQL5中配置给Mysql8会报错
# 官方文档： https://dev.mysql.com/doc/refman/8.0/en/option-files.html
# ----------------------------------------------------------------------
#
# On Linux you can copy this file to /etc/my.cnf to set global options,
# mysql-data-dir/my.cnf to set server-specific options
# (@localstatedir@ for this installation) or to
# ~/.my.cnf to set user-specific options.
#
# On Windows you should keep this file in the installation directory 
# of your server (e.g. C:\Program Files\MySQL\MySQL Server X.Y). To
# make sure the server reads the config file use the startup option 
# "--defaults-file". 
#
# To run run the server from the command line, execute this in a 
# command line shell, e.g.
# mysqld --defaults-file="C:\Program Files\MySQL\MySQL Server X.Y\my.ini"
#
# To install the server as a Windows service manually, execute this in a 
# command line shell, e.g.
# mysqld --install MySQLXY --defaults-file="C:\Program Files\MySQL\MySQL Server X.Y\my.ini"
#
# And then execute this in a command line shell to start the server, e.g.
# net start MySQLXY
#
#
# Guildlines for editing this file
# ----------------------------------------------------------------------
#
# In this file, you can use all long options that the program supports.
# If you want to know the options a program supports, start the program
# with the "--help" option.
#
# More detailed information about the individual options can also be
# found in the manual.
#
#
# CLIENT SECTION  客户端部分
# ----------------------------------------------------------------------
#
# The following options will be read by MySQL client applications.
# Note that only client applications shipped by MySQL are guaranteed
# to read this section. If you want your own MySQL client program to
# honor these values, you need to specify it as an option during the
# MySQL client library initialization.
#
[client]
port=3306
default-character-set=utf8mb4

[mysql]
# 设置mysql客户端默认字符集。utf8不支持Emoji表情和极少数的汉字
default-character-set=utf8mb4  


# SERVER SECTION  服务端部分
# ----------------------------------------------------------------------
#
# The following options will be read by the MySQL Server. Make sure that
# you have installed the server correctly (see above) so it reads this 
# file.
#
[mysqld]
port=3306  # The TCP/IP Port the MySQL Server will listen on

# The maximum amount of concurrent sessions the MySQL server will
# allow. One of these connections will be reserved for a user with
# SUPER privileges to allow the administrator to login even if the
# connection limit has been reached.
max_connections=200

# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10

# The default character set that will be used when a new schema or table is
# created and no character set is defined
character-set-server=utf8mb4

# The default storage engine that will be used when create new tables when
default-storage-engine=INNODB

basedir="D:/Develop/mysql/mysql-8.0.32-winx64/"  #Path to installation directory. All paths are usually resolved relative to this.
datadir="D:/Develop/mysql/mysql-8.0.32-winx64/data/"  #Path to the database root  数据库存储路径

# 配置时区
default-time-zone='+08:00'

# Set the SQL mode to strict
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
#sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"  # 这个有问题


# 自己了解的配置
max_heap_table_size = 4000M  # 限制MEMORY引擎表的大小
log_timestamps = system  # 设置日志时区
```



## 安装

```cmd
>mysqld --verbose --help
>mysqld --initialize-insecure  # 创建默认数据库并退出。创建一个密码为空的root用户。在mysql根目录中生产data文件夹
>mysqld install MySQL8  # 安装服务
Install/Remove of the Service Denied!  # 说明权限不够，需以管理员身份运行
>mysql --help
>mysql -u root  # 登录
```

```mysql
mysql>show databases;
mysql>use mysql;
mysql>show tables;
mysql>select * from user;
mysql>alter user 'root'@'localhost' identified by 'MySQL1616';  // 设置root新密码为MySQL1616
mysql>quit
```



## 问题

问题1：远程无法通过root用户3306端口访问MySQL8
原因1：MySQL8默认root用户只能本地localhost访问
原因2：Win防火墙
解决1：修改mysql表中localhost为% where user = root
解决2：Win防火墙添加入站规则，设置3306端口允许连接



## CLI

| 命令行操作                                                   | 释义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **mysqld**                                                   |                                                              |
| mysqld --standalone                                          | 应用场景 生产环境                                            |
| mysqld --console                                             | 应用场景 开发环境                                            |
| mysqld --initialize -insecure --user=mysql --console         | 初始化MySQL，需要正确配置my.ini，会创建一个dada文件夹(存所有数据库数据) 在my.ini 中设置的位置，会在控制台打印临时密码<br />-insecure --user 用于指定文件夹的用户权限/拥有者 |
| mysqld install MySQL8 --defaults-file="D:/Develop/mysql/mysql-8.0.32-winx64/my.ini" | 安装MySQL服务<br />defaults file 要有，用于设置注册表的      |
| mysqld remove mysql8                                         | 卸载 MySQL 服务                                              |
| **mysqladmin**                                               |                                                              |
| mysqladmin -u用户名 -p旧密码 password 新密码                 | 修改密码（在MySQL的bin目录下），不建议采用这种方式(可能是错误的方式) |
| **mysql**                                                    |                                                              |
| mysql -h目标主机IP地址 -u用户名 -p [-P端口号]                | 连接到MySQL，代码通过命令行接口参数输入不安全，所以不在这里输入。第一次登录不需要密码 |
| mysql –uroot –p123 -Dtest<"C:\test.sql"或source C:\test.sql  | 执行sql脚本                                                  |
| exit或quit                                                   | 退出MySQL                                                    |
| **其他**                                                     |                                                              |
| net start mysql5                                             | Win管理员权限开启MySQL服务                                   |
| net stop mysql8                                              | Win管理员关闭MySQL服务                                       |
| sc delete MySQL5                                             | 删除服务列表里面的服务                                       |
|                                                              |                                                              |



## MySQL数据库驱动

### JDBC驱动程序

Java DataBase Connectivity standard Drive

Mysql5.0：com.mysql.jdbc.Driver
Mysql8.0：com.mysql.cj.jdbc.Driver，cj表示connector/J

```java
<!-- https://mvnrepository.com/artifact/com.mysql/mysql-connector-j -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.3.0</version>
</dependency>
```

```properties
# JDBC 连接字符串：
# driveType:databaseType://host:port/database?user=root&password=MySQL1616
jdbc:mysql://localhost:3306/db_name?useUnicode=true&amp;characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
# XML中需要使用&amp;代替&，因为&是特殊字符
```



## 数据库连接池

HikariCP

C3P0

Druid







## 数据库设计

### 表间关系一对一

把不经常访问和存在Null值的字段从原表分离，以获得更高的性能



### 表间关系多对多

通过一个中间表，关联两个一对多关系，就形成了多对多关系



# MariaDB11.3.2

2009Oracle收购MySQL，MariaDB11.3.2是MySQL的一个分支，CentOS7yum默认MariaDB
[MariaDB官方文档](https://mariadb.org/documentation/)	|	[mariadb-docker容器](https://hub.docker.com/_/mariadb)



# 未整理

# 业务设计（数据结构）

数据库设计

**电商数据库设计**

先写每张表的基本字段，再设计表间关系

不要从表、字段的角度设计，而从实体、模型的角度设计



## 分类表的常见结构设计

无限级分类：查询效率比较低

四级分类：中小型电商

两级：小项目

### 记住上级节点

```sql
-- ----------------------------
-- Table structure for category
-- 数据结构：分类表(category):用二维的关系表来表示的树状结构
-- ----------------------------
DROP TABLE IF EXISTS `category`;
CREATE TABLE `category` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `description` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
    -- 描述
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `delete_time` datetime(3) DEFAULT NULL,
  `is_root` tinyint(3) unsigned NOT NULL DEFAULT '0',
    -- 表示是否是根节点
  `parent_id` int(10) unsigned DEFAULT NULL,
    -- 父节点
  `img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `index` int(10) unsigned DEFAULT NULL,
    -- 尽量多添加一个索引字段，用于运营调整排序
  `online` int(10) unsigned DEFAULT '1',
    -- 表示上架还是下架，即使下架也不会删除而是打上删除表示
  `level` int(10) unsigned DEFAULT NULL,
    -- 表示分类的所属级别，可以简化查询请求
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=40 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

### 路径表示法

/node1_id/node2_id/node3_id

添加 path字段，用于保存完整路径，适用于五级分类以上

不太依赖其他记录，每一个节点都可以完整的表达当前的节点信息

缺点：Java代码中需要拼接路径。这是一个**反范式**的设计，有一定程度的冗余，会占用数据库的空间资源



/node1_id#name/node2_id/node3_id

进阶：采用丰富的路径信息 (如，记录name信息。通过自己的定义的方法解析路径)

## Theme主题设计

Theme：一组SPU集合

```sql
-- ----------------------------
-- Table structure for theme
-- ----------------------------
DROP TABLE IF EXISTS `theme`;
CREATE TABLE `theme` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `description` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `name` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `tpl_name` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `delete_time` datetime(3) DEFAULT NULL,
  `entrance_img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `extend` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `internal_top_img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `title_img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `online` tinyint(3) unsigned DEFAULT '1',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

title：可以显示到前端的标题

name：是一种文本标识

tpl_name: template_name专题的前端模板（页面模板）的名字

extend：备用的扩展字段

### 多对多关系表

```sql
-- ----------------------------
-- Table structure for theme_spu
-- ----------------------------
DROP TABLE IF EXISTS `theme_spu`;
CREATE TABLE `theme_spu` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `theme_id` int(10) unsigned NOT NULL,
  `spu_id` int(10) unsigned NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=101 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

### Model设计

```java
package com.zjbti.missyou.model;

import lombok.Getter;
import lombok.Setter;
import org.hibernate.annotations.Where;

import javax.persistence.*;
import java.util.List;

/**
 * @date 2021/5/7 @Created by cxc
 */
@Entity
@Getter
@Setter
@Where(clause = "delete_time is null")
public class Theme extends BaseEntity {
    @Id
    private Long id;
    private String title;
    private String description;
    private String name;
    private String tplName;
    private String entranceImg;
    private String extend;
    private String internalTopImg;
    private String titleImg;
    private Boolean online;

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(name = "theme_spu", joinColumns = @JoinColumn(name = "theme_id"),
            inverseJoinColumns = @JoinColumn(name = "spu_id"))
    private List<Spu> spuList;
}

```

## SPU设计

SPU：Standard Product Unit （标准化产品单元，是商品信息聚合的最小单位）。例如，iPhone X

```sql
-- ----------------------------
-- Table structure for spu
-- ----------------------------
-- spu: Standard Product Unit 标准化产品单元。 是商品信息聚合的最小单位
DROP TABLE IF EXISTS `spu`;
CREATE TABLE `spu` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `subtitle` varchar(800) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `category_id` int(10) unsigned NOT NULL,
--   root_category_id: 为了查询方便，冗余的字段
  `root_category_id` int(11) DEFAULT NULL,
  `online` tinyint(3) unsigned NOT NULL DEFAULT '1',
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `delete_time` datetime(3) DEFAULT NULL,
--   price: spu的价格只是用来展示的，文本类型
  `price` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '文本型价格，有时候SPU需要展示的是一个范围，或者自定义平均价格',

  `sketch_spec_id` int(10) unsigned DEFAULT NULL COMMENT '某种规格可以直接附加单品图片',
  `default_sku_id` int(11) DEFAULT NULL COMMENT '默认选中的sku',
  `img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `discount_price` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `description` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `tags` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `is_test` tinyint(3) unsigned DEFAULT '0',
  `spu_theme_img` json DEFAULT NULL,
  `for_theme_img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=29 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

category_id：表示二级category_id

root_category_id：表示根节点category_id，为了查询方便的冗余字段，不然要一级一级往上查

sketch_spec_id：sketch 略图

price：这价格只是用来展示的，采用文本类型

-- 需要计算的价格也不采用浮点类型(存在误差)，一般用decimal类型

```java
package com.zjbti.missyou.model;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.util.List;

/**
 * @date 2021/4/22 @Created by cxc
 */
@Entity
@Getter
@Setter
public class Spu extends BaseEntity {
    @Id
    private Long id;
    private String title;
    private String subtitle;
    private Long categoryId;
    private Long rootCategoryId;
    private Boolean online;
    private String price;
    private Long sketchSpecId;
    private Long defaultSkuId;
    private String img;
    private String discountPrice;
    private String description;
    private String tags;
    private Boolean isTest;
//    private Object spuThemeImg;
    private String forThemeImg;

    @OneToMany(fetch = FetchType.LAZY)
    @JoinColumn(name = "spuId")
    private List<Sku> skuList;

    @OneToMany(fetch = FetchType.LAZY)
    @JoinColumn(name = "spuId")
    private List<SpuImg> spuImgList;

    @OneToMany(fetch = FetchType.LAZY)
    @JoinColumn(name = "spuId")
    private List<SpuDetailImg> spuDetailImgList;
}

```

### spu_detail_img

```sql
-- ----------------------------
-- Table structure for spu_detail_img
-- ----------------------------
DROP TABLE IF EXISTS `spu_detail_img`;
CREATE TABLE `spu_detail_img` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `spu_id` int(10) unsigned DEFAULT NULL,
  `index` int(10) unsigned NOT NULL,
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `delete_time` datetime(3) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=28 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

```java
package com.zjbti.missyou.model;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.Entity;
import javax.persistence.Id;

/**
 * @date 2021/4/22 @Created by cxc
 */
@Entity
@Getter
@Setter
public class SpuDetailImg {
    @Id
    private Long id;
    private String img;
    private Long spuId;
    private Long index;
}

```



### spu_img

```sql
-- ----------------------------
-- Table structure for spu_img
-- ----------------------------
DROP TABLE IF EXISTS `spu_img`;
CREATE TABLE `spu_img` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `spu_id` int(10) unsigned DEFAULT NULL,
  `delete_time` datetime(3) DEFAULT NULL,
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=194 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

```java
package com.zjbti.missyou.model;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.Entity;
import javax.persistence.Id;

/**
 * @date 2021/4/22 @Created by cxc
 */
@Entity
@Getter
@Setter
public class SpuImg extends BaseEntity {

    @Id
    private Long id;
    private String img;
    private Long spuId;
}

```



## SKU设计

SKU：Stock Keeping Unit（库存量单位）。例如，iPhone X,64g,银色 是一个SKU

```sql
-- ----------------------------
-- Table structure for sku
-- ----------------------------
DROP TABLE IF EXISTS `sku`;
CREATE TABLE `sku` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `price` decimal(10,2) unsigned NOT NULL,
  `discount_price` decimal(10,2) unsigned DEFAULT NULL,
  `online` tinyint(3) unsigned NOT NULL DEFAULT '1',
  `img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `title` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `spu_id` int(10) unsigned NOT NULL,
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `delete_time` datetime(3) DEFAULT NULL,
  `specs` json DEFAULT NULL,
  `code` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `stock` int(10) unsigned NOT NULL DEFAULT '0',
  `category_id` int(10) unsigned DEFAULT NULL,
  `root_category_id` int(10) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=52 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

SPU里category_id、root_category_id是冗余的，因为可根据SKU找SPU再找Category，是因为不想做级联的查询

specs：JSON表示和规格有关的复杂数据结构，冗余字段，用于减少查询次数

这里的specs表示商品的规格，是可以商家自定义的，因此如果设计为一个表则没有能增加字段的方式，所以设计为一个Json字段表示一个对象，需要时再反序列化为对象使用

> 序列化
>
> 解决对象对象的传输（把对象序列化为通用的字符串写入到一个字段中）和存储（可以使用MongoDB，文档型数据库）的问题
> 前端是可以将字符串组合成JSON对象的
>
> 序列化有 Jackson和FastJson等库

code：如15$1-19#6-28，15是SPU的id，1-19表示颜色，6-28表示尺码，是自定义的字符串，表示每个SKU的唯一的标识，和id类似，但是code具有可解析性，便于前端处理

stock：表示库存量（如何业务需要不同单位，可以添加一个单位的表）

```java
package com.zjbti.missyou.model;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.Entity;
import javax.persistence.Id;
import java.math.BigDecimal;

/**
 * @date 2021/4/22 @Created by cxc
 */
@Entity
@Getter
@Setter
public class Sku extends BaseEntity {
    @Id
    private Long id;
    private BigDecimal price;
    private BigDecimal discountPrice;
    private Boolean online;
    private String img;
    private String title;
    private Long spuId;
    private Long categoryId;
    private Long rootCategoryId;

    private String specs;
    private String code;
    private Long stock;
}

```

### 规格Spec

```sql
-- ----------------------------
-- Table structure for sku_spec
-- ----------------------------
DROP TABLE IF EXISTS `sku_spec`;
CREATE TABLE `sku_spec` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `spu_id` int(10) unsigned NOT NULL,
  `sku_id` int(10) unsigned NOT NULL,
  `key_id` int(10) unsigned NOT NULL,
  `value_id` int(10) unsigned NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=90 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

规格Spec和库存量单位SKU的多对多关系，SKU_Spec表是多对多关系的第三张表

spu_id、key_id 是冗余字段

### SPU-SKU-Spec关系

Spu - Spec_Key 多对多

Sku - Spec_Value 多对多

Spu - Sku 一对多

### 规格名Spec_Kye

如颜色、尺码

```sql
-- ----------------------------
-- Table structure for spec_key
-- ----------------------------
DROP TABLE IF EXISTS `spec_key`;
CREATE TABLE `spec_key` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `unit` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `standard` tinyint(3) unsigned NOT NULL DEFAULT '0',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `delete_time` datetime DEFAULT NULL,
  `description` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```



### 规格值Spec_Value

如橙色、蓝色；XL、L、M、S

```sql
-- ----------------------------
-- Table structure for spec_value
-- ----------------------------
DROP TABLE IF EXISTS `spec_value`;
CREATE TABLE `spec_value` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `value` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `spec_id` int(10) unsigned NOT NULL,
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `delete_time` datetime(3) DEFAULT NULL,
  `extend` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=46 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

## Banner

```sql
-- ----------------------------
-- Table structure for banner
-- ----------------------------
DROP TABLE IF EXISTS `banner`;
CREATE TABLE `banner` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `description` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `delete_time` datetime(3) DEFAULT NULL,
  `title` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '部分banner可能有标题图片',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

```java
package com.zjbti.missyou.model;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.util.List;

/**
 * @date 2021/2/8 @Created by cxc
 */
@Entity
@Getter
@Setter
public class Banner extends BaseEntity {
    @Id
    private Long id;
    private String name;
    private String description;
    private String title;
    private String img;

    @OneToMany(fetch = FetchType.LAZY)
    @JoinColumn(name="bannerId")
    private List<BannerItem> items;
}

```



### Banner_Item

```sql
-- ----------------------------
-- Table structure for banner_item
-- ----------------------------
DROP TABLE IF EXISTS `banner_item`;
CREATE TABLE `banner_item` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `keyword` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `type` smallint(5) unsigned NOT NULL DEFAULT '0',
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `delete_time` datetime(3) DEFAULT NULL,
  `banner_id` int(10) unsigned NOT NULL,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=16 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

```java
package com.zjbti.missyou.model;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.Entity;
import javax.persistence.Id;

/**
 * @date 2021/2/8 @Created by cxc
 */
@Entity
@Getter
@Setter
public class BannerItem extends BaseEntity {
    @Id
    private Long id;
    private String img;
    private String keyword;
    private Short type;
    private Long bannerId;
    private String name;
}

```

## Category

```sql
-- ----------------------------
-- Table structure for category
-- ----------------------------
DROP TABLE IF EXISTS `category`;
CREATE TABLE `category` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `description` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `delete_time` datetime(3) DEFAULT NULL,
  `is_root` tinyint(3) unsigned NOT NULL DEFAULT '0',
  `parent_id` int(10) unsigned DEFAULT NULL,
  `img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `index` int(10) unsigned DEFAULT NULL,
  `online` int(10) unsigned DEFAULT '1',
  `level` int(10) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=40 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

grid_category：宫格分类，表示需要特别显示的分类

## Coupon 优惠卷

加入优惠卷后小麻烦，需要管理优惠卷还需要校验

```sql
-- ----------------------------
--  Table structure for `coupon`
-- ----------------------------
DROP TABLE IF EXISTS `coupon`;
CREATE TABLE `coupon` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `start_time` datetime DEFAULT NULL,
  `end_time` datetime DEFAULT NULL,
  `description` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `full_money` decimal(10,2) DEFAULT NULL,
  `minus` decimal(10,2) DEFAULT NULL,
  `rate` decimal(10,2) DEFAULT NULL ,
  `type` smallint(6) NOT NULL COMMENT '1. 满减券 2.折扣券 3.无门槛券 4.满金额折扣券',
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `delete_time` datetime(3) DEFAULT NULL,
  `activity_id` int(10) unsigned DEFAULT NULL,
  `remark` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `whole_store` tinyint(3) unsigned DEFAULT '0',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

## user_coupon 用户有的优惠卷

```sql
-- ----------------------------
-- Table structure for user_coupon
-- ----------------------------
DROP TABLE IF EXISTS `user_coupon`;
CREATE TABLE `user_coupon` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `user_id` int(10) unsigned NOT NULL,
  `coupon_id` int(10) unsigned NOT NULL,
  `status` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT '1:未使用，2：已使用， 3：已过期',
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `order_id` int(10) unsigned DEFAULT NULL,
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  PRIMARY KEY (`id`),
  UNIQUE KEY `uni_user_coupon` (`user_id`,`coupon_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=31 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

SET FOREIGN_KEY_CHECKS = 1;

```

status不能完全表示优惠卷的状态，因为要引入Redis和消息队列，那么是异步的，需要考虑线程安全。

优惠卷过期问题 status:未使用、已使用、已过期：触发机制
	1.主动触发：轮询机制（隔多久扫描一次数据库）
	2.被动触发：设置第三方在过期时间点发消息
		Redis / RocketMQ

在判断是否过期时，尽量不能使用 状态字段(status) ，而是使用 startTime、endTime



计算折扣时，使用银行家算法

### 优惠卷过期状态问题

通过复杂查询，间接查询优惠卷的状态

## Order 订单

```sql
-- ----------------------------
-- Table structure for order
-- ----------------------------
DROP TABLE IF EXISTS `order`;
CREATE TABLE `order` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
	-- 表间自增id可能不唯一
  `order_no` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
    -- 唯一订单编号，方便分布式的分库分表
  `user_id` int(10) unsigned DEFAULT NULL COMMENT 'user表外键',
  `total_price` decimal(10,2) DEFAULT '0.00',
    -- 原始价格
  `total_count` int(11) unsigned DEFAULT '0',
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
  `delete_time` datetime(3) DEFAULT NULL,
  `expired_time` datetime(3) DEFAULT NULL,
  `placed_time` datetime(3) DEFAULT NULL,
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
  `snap_img` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `snap_title` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `snap_items` json DEFAULT NULL,
  `snap_address` json DEFAULT NULL,
    -- 快照，不记录实时信息
  `prepay_id` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
    -- 用于支付的id
  `final_total_price` decimal(10,2) DEFAULT NULL,
  `status` tinyint(3) unsigned DEFAULT '1',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `uni_order_no` (`order_no`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=272 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

订单的校验参数：
1.商品是否有货
2.商品最大购买数量
3.SKU是否存在
4.totalPrice
5.finalTotalPrice
6.是否拥有优惠卷
7.优惠卷是否过期

### 前端传过来的订单数据

```java
package com.zjbti.missyou.dto;

import lombok.Getter;
import lombok.Setter;

import javax.validation.constraints.DecimalMax;
import javax.validation.constraints.DecimalMin;
import java.math.BigDecimal;
import java.util.List;

/**
 * 前端价格不可信
 * @date 2021/5/9 @Created by cxc
 */
@Getter
@Setter
public class OrderDTO {

    @DecimalMin(value = "0.00", message = "不在合法范围内")
    @DecimalMax(value = "99999999,99", message = "不在合法范围内")
    private BigDecimal totalPrice;
    private BigDecimal finalTotalPrice;
    private Long couponId;
    private List<SkuInfoDTO> skuInfoList;
    private OrderAddressDTO address;
}

```

### 在service中处理的订单业务对象BO

```java
package com.zjbti.missyou.bo;

import com.zjbti.missyou.dto.SkuInfoDTO;
import com.zjbti.missyou.model.Sku;
import lombok.Getter;
import lombok.Setter;

import java.math.BigDecimal;

/**
 * @date 2021/5/10 @Created by cxc
 */
@Getter
@Setter
public class SkuOrderBO {

    private BigDecimal actualPrice;  // 真实价格
    private Integer count;
    private Long categoryId;

    public SkuOrderBO(Sku sku, SkuInfoDTO skuInfoDTO) {
        this.actualPrice = sku.getActualPrice();
        this.count = skuInfoDTO.getCount();
        this.categoryId = sku.getCategoryId();
    }

    public BigDecimal getTotalPrice() {
        return this.actualPrice.multiply(new BigDecimal(this.count));
    }
}

```

### 订单自动取消机制

消息队列，Redis 实现

两种方式

不记录 expired_time，在更该支付延长时间会有问题



Order（根据库存，下订单）和Coupon（根据优惠卷分类查询）不一样，需要明确的状态

Order 还需要 （自动取消订单，归还库存）
	怎么归还库存、什么时候、谁来触发归还库存

怎么归还？

​	1主动轮询，定时器，Java Timer，Linux Corntab，定时任务框架 Quartz

​	2被动触发，用户触发，过期时间 类似用户/角色

​	3队列，先进先出，RocketMQ，RabbitMQ，Kafka  系统 取，队列主动返回

​	4延迟消息队列 Redis 键名通知、RocketMQ(比Redis可靠)

触发时机？

​	redis的键空间通知(在某个键控件发生了指定类型的操作比如expire过期操作，那么订阅了该键控件的客户端都会收到通知)

当然也可以使用RocketMQ 作为延迟消息队列，都是可以d

### 库存扣除机制

在支付时扣除库存，容易因并发，出现超卖的情况

多线程并发要有乱序思维

情况：
	1.正常
	2.负数
	3.报错（库存不足）
	4.加锁 排队
		数据库行锁 悲观锁  性能不高   （开启事务 select .. sku.tock for update）,不推荐，因为并发消耗大，如果索引不正确容易锁表
		数据库 锁 **乐观锁**乐观并发机制	没有实际锁  并发/竞态 思想
			`update sku set stock = newValue version = version + 1 where version = 查出来的version`用version保证不重复

​		Java加锁（分布式无效）  

错误思维：先查询再判断再执行业务逻辑，线程不安全，两条SQL不是一个原子的操作
	事务也不能解决，事务不等于锁

#### 预扣除库存机制



