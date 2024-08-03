



# Zookeeper

Zookeeper是一个`分布式服务框架`，是Apache Hadoop 的一个子项目，它主要是用来解决分布式应用中经常遇到的一些**元数据管理**问题，如：**统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理**等。

简单来说zookeeper=文件系统+监听通知机制。

**Zookeeper维护一个类似文件系统的数据结构**：

```txt
/
|-/NameService
|	  |-/Server1
|	  |_/Server2
|-/configuration
|-/GroupMembers
|	  |-/Member1
|	  |_/Member2
\_/Apps
	  |-/App1
	  |-/App2
	  |_/App3
		  |-/SubApp1
		  |_/SubApp2
```



znode(目录节点)：可以增删znode，znode还能能存储数据，znode类型：

- **PERSISTENT-持久化目录节点**

  客户端与zookeeper断开连接后，该节点依旧存在

- **PERSISTENT_SEQUENTIAL-持久化顺序编号目录节点**

  客户端与zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号

- **EPHEMERAL-临时目录节点**

  客户端与zookeeper断开连接后，该节点被删除

- **EPHEMERAL_SEQUENTIAL-临时顺序编号目录节点**

  客户端与zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号

**监听通知机制**

客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、被删除、子目录节点增加删除）时，zookeeper会通知客户端。

## zookeeper端口号说明：

　　2181：客户端连接zookeeper集群使用的监听端口号
　　3888：选择leader使用
　　2888：集群内机器通讯使用（leader和follower之间数据同步使用的端口号，leader监听此端口）

## 命令操作zookeeper

| 命令                 | 释义                                                  |
| -------------------- | ----------------------------------------------------- |
| ./zkServer.sh start  | 启动zookeeper                                         |
| ls /                 | 查看当前 ZooKeeper 的 / 目录中所包含的内容 znode      |
| create /zkPro myData | 创建一个新的 znode，存储数据 myData                   |
| get /zkPro myData    | get 命令来确认 znode 是否包含我们所创建的myData字符串 |
| set /zkPro myData123 | set 命令来对 zk 所关联的字符串进行设置                |
| delete /zkPro        | 将 znode 删除                                         |
|                      |                                                       |



## API操作zookeeper

## Curator

curator管理员，curator-recipes管理员手册

Apache Curator是一个比较`完善的ZooKeeper客户端框架`，通过封装的一套高级API 简化了ZooKeeper的操作，解决了很多Zookeeper客户端非常底层的细节开发工作

Curator主要解决了三类问题：

- 封装ZooKeeper client与ZooKeeper server之间的`连接处理`
- 提供了一套`Fluent风格的操作API`
- 提供ZooKeeper`各种应用场景(recipe， 比如：分布式锁服务、集群领导选举、共享计数器、缓存机制、分布式队列等)的抽象封装`

### Fluent流风格的操作API

```java
private void makeNormal(Customer customer) {
    Order o1 = new Order();
    customer.addOrder(o1);
    OrderLine line1 = new OrderLine(6, Product.find("TAL"));
    o1.addLine(line1);
    OrderLine line2 = new OrderLine(5, Product.find("HPK"));
    o1.addLine(line2);
    OrderLine line3 = new OrderLine(3, Product.find("LGV"));
    o1.addLine(line3);
    line2.setSkippable(true);
    o1.setRush(true);
}
```

```java
private void makeFluent(Customer customer) {
    customer.newOrder()
            .with(6, "TAL")
            .with(5, "HPK").skippable()
            .with(3, "LGV")
            .priorityRush();
}
```

### 使用 Curator

https://www.cnblogs.com/erbing/p/9799098.html

