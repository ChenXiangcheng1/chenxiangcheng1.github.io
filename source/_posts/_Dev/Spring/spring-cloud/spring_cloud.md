# Spring Cloud2023.0.0-M2

是快速构建分布式系统的工具集

[官网](https://spring.io/projects/spring-cloud)



## 微服务

[架构笔记#微服务](../../架构/架构.md#微服务)

![image-20230927142551934](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_27_14_25_52.png)

[阿里云微服务生态全景图](https://start.aliyun.com/ecosystem.html)



## 组件

<center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_08_27_11_08.png" alt="image-20230827110827235" style="zoom:55%;" /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_08_27_11_15.png" alt="image-20230827111545973" style="zoom:40%;" /></center>

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1009_181301.png" alt="image-20231009181301035" style="zoom:50%;" />



| 特征组件                                                     | SpringCloud                                      | SCN OSS      | SCA                                    | 其他                                  |
| ------------------------------------------------------------ | ------------------------------------------------ | ------------ | -------------------------------------- | ------------------------------------- |
| 配置中心 - 分布式/版本化的配置                               | Spring Cloud Config                              | ~~Archaius~~ | **Nacos Config**<br />ACM              | **Apollo**(携程)、Zookeeper、Consul、 |
| 服务注册发现 - 注册中心                                      | Spring Cloud Consul                              | ~~Eureka~~   | **Nacos Discovery**<br />ANS           | Zookeeper、K8S的etcd                  |
| 流量控制熔断降级 Circuit Breakers 断路器                     |                                                  | ~~Hystrix~~  | **Sentinel**                           | Resilience4J                          |
| 网关路由                                                     | **Spring Cloud Gateway**                         | ~~Zuul~~     | Dubbo                                  | Nginx+Lua                             |
| 服务调用                                                     | RestTemplate、WebClient、OpenFeign               | Feign        | Dubbo(是RPC框架还达不到服务网关的级别) |                                       |
| 负载均衡 Load balancing                                      | Spring Cloud Loadbalancer                        | ~~Ribbon~~   | Dubbo                                  |                                       |
| 链路跟踪                                                     |                                                  |              |                                        | Zipkin、**Skywalking**、Pinpoint      |
| 全局锁 Global locks                                          | Spring Cloud Cluster(已迁移到Spring Integration) |              |                                        |                                       |
| Leadership election and cluster state 领导选举与集群状态管理 | Spring Cloud Cluster(已迁移到Spring Integration) |              |                                        |                                       |
| 分布式消息                                                   | Spring Cloud Stream + Kafka/RabbitMQ             |              | **SCS + RocketMQ**                     |                                       |
| 分布式任务调度                                               |                                                  |              | SchedulerX                             |                                       |
| 分布式事务                                                   |                                                  |              | Seata                                  |                                       |

2018 年 Netflix 公司宣布 SCN 核心组件 Archaius、Eureka、Hystrix、Zuul、Ribbon 等进入维护状态

Spring Cloud Alibaba 已经成为事实标准



## 版本

[SpringBoot 支持时间表](https://spring.io/projects/spring-boot#support)
[Spring Cloud 支持策略和版本命名规则](https://github.com/spring-cloud/spring-cloud-release/wiki/Supported-Versions)

[Spring Cloud 支持的 Spring Boot 版本](https://start.spring.io/actuator/info)
[Spring Cloud Alibaba 支持的 Spring Cloud 版本](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)



[github Spring Cloud Relaese wiki - 新的值得关注的特性"new and noteworthy" features](https://github.com/spring-cloud/spring-cloud-release/wiki#release-notes)
[github Spring Cloud Relaese milestones - 未来计划](https://github.com/spring-cloud/spring-cloud-release/milestones)



## 注解

| 注解                     | 元注解Target             | 释义                                                         | 成员变量 |
| ------------------------ | ------------------------ | ------------------------------------------------------------ | -------- |
| `@RefreshScope`          | TYPE、METHOD             | 将 Bean 放置到 refresh 作用域，刷新后获取新实例<br />可以实现 Spring Cloud 配置自动更新<br />refresh 作用域允许 Bean 在运行时被动态刷新 |          |
| `@EnableDiscoveryClient` | TYPE                     | 启动 DiscoveryClient 实现，自动注册本地服务`@Import(EnableDiscoveryClientImportSelector.class)`<br />从 Spring Boot 2.0 版本开始，不再需要显式使用 `@EnableDiscoveryClient` 注解来启用服务发现功能，只需在 pom.xml 中添加相应的服务发现依赖即可。 |          |
| `@LoadBalanced`          | FIELD、PARAMETER、METHOD | 标记一个 RestTemplate 或 WebClient bean 使用 LoadBalancerClient<br />`@Qualifier` |          |
|                          |                          |                                                              |          |



## API

### DiscoveryClient

表示提供给发现服务 (Netflix Eureka或consult.io) 的读取操作

`getInstances(String):List<ServiceInstance>` 获取与 serviceId 相关的**服务实例**

`getService():List<String>` 返回所有已知的 serviceId



### ServiceInstance

表示发现系统中的一个服务实例

`getUri():URI` 返回服务 URI 地址



# Spring Cloud Alibaba2022.0.0.0

[TODO: 看官网(挺好的)](https://sca.aliyun.com/zh-cn/)	|	[github wiki(版本说明、Nacos Config、Nacos Discovery)](https://github.com/alibaba/spring-cloud-alibaba/wiki)	|	[Spring官网](https://spring.io/projects/spring-cloud-alibaba)

[阿里云云原生脚手架](https://start.aliyun.com/)



[2.2.9 参考文档](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/zh-cn/index.html)



## Nacos2.2.3

Nacos(Dynamic Naming and Configuration Service) 是一个服务发现组件，提供**发现、配置、管理服务**的功能。

[文档](https://nacos.io/zh-cn/docs/v2/ecology/use-nacos-with-spring-cloud.html)	|	[Nacos架构与原理](https://developer.aliyun.com/ebook/36?spm=a2c6h.20345107.ebook-index.18.152c2984fsi5ST)

路由策略(根据具体条件将请求导向适当的服务实例)：基于权重的轮询、随机选择、一致性哈希等
流量控制



**使用**

> * 单机启动服务器
>   Linux 			`sh startup.sh -m standalone`
>   Windows 		`startup.cmd -m standalone`
>   http://localhost:8848/nacos
>   
> * 关闭服务器
>   Linux 			`sh shutdown.sh`
>   Windows 		`shutdown.cmd`
>
> * 服务注册 `curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instanceserviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'`
>
> * 服务发现 `curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instance/list?serviceName=nacos.naming.serviceName'`
>
> * 发布配置 `curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configsdataId=nacos.cfg.dataId&group=test&content=HelloWorld"`
>
> * 获取配置 `curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"`



### 配置

```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        namespace: 450cb8f7-64d4-4be0-8c5d-49b7450c2d6c  # Nacos命名空间ID
        cluster-name: HZ  # Nacos集群名称
        metadata:
          instance: content-center-instanceName
          # 自己这个实例的版本
          version: v1
          # 允许调用的提供者版本
          target-version: v1
      config:
        server-addr: 127.0.0.1:8848
  config:
    import:
      - optional:nacos:dev.yml  # 监听nacos配置中心的DEFAULT_GROUP:dev.yml
      - optional:nacos:aaa.yml?group=group_01&refreshEnabled=false # 不开启动态刷新
    activate:
      on-profile: nacos
  application:
    name: user-center  # 服务名称尽量用-, 不要用_  # 作为Nacos服务名称
```



### 动态配置管理

**配置 ID** 

Nacos Spring Cloud中，dataId 的完整格式为 \${prefix}-\${spring.profiles.active}.${file-extension} 
${prefix} = spring.application.name
${spring.profiles.active} 
${file-exetensionfile-exetension} = spring.cloud.nacos.config.file-extension



### 动态服务发现

**Nacos 服务发现的领域模型**

范围：服务 - 集群 - 实例

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1014_150403.png" alt="image-20231014150403313" style="zoom: 60%;" />



**元数据**

作用：
提供描述信息
让微服务调用更加灵活(例如：微服务版本控制)

分为：服务级别、集群级别、实例级别



### 问题

本地 IP 发生变化后，发生

```bash
caused: errCode: 500, errMsg: do metadata operation failed ;caused: com.alibaba.nacos.consistency.exception.ConsistencyException: The Raft Group [naming_instance_metadata] did not find the Leader node;caused: The Raft Group [naming_instance_metadata] did not find the Leader node
```

```log
java.lang.IllegalStateException: Fail to get leader of group naming_service_metadata, Unknown leader
	at com.alipay.sofa.jraft.core.CliServiceImpl.getPeers(CliServiceImpl.java:605)
	at com.alipay.sofa.jraft.core.CliServiceImpl.getPeers(CliServiceImpl.java:498)
	at com.alibaba.nacos.core.distributed.raft.JRaftServer.registerSelfToCluster(JRaftServer.java:353)
	at com.alibaba.nacos.core.distributed.raft.JRaftServer.lambda$createMultiRaftGroup$0(JRaftServer.java:264)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:539)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
	at java.base/java.lang.Thread.run(Thread.java:833)
2023-10-16 00:00:00,552 ERROR Failed to join the cluster, retry...
```

解决：
检查 alipay-jraft.log 和 Naming-raft.log
删除目录 nacos2-service\current\data\protocol



## Ribbon

负载均衡库，用负载均衡算法计算出一个实例，交给 RestTemplate 去请求

```java
restTemplate.getForObject("http://service-provider/echo/" + str, String.class);
```

 [文档](https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-ribbon.html)



**负载均衡**

<center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1014_152759.png" alt="image-20231014152759451" style="zoom:50%;" /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1014_152808.png" alt="image-20231014152808791" style="zoom:50%;" /></center>



**架构变化**

<center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1014_215834.png" alt="image-20231014215828922" style="zoom:55%;" /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1014_215845.png" alt="image-20231014215845007" style="zoom:55%;" /></center>



### Ribbon 组成

(配置项)

| 接口                     | 作用                           | 默认值                                                       |
| ------------------------ | ------------------------------ | ------------------------------------------------------------ |
| IclientConfig            | 读取配置                       | DefaultClientConfigImpl                                      |
| IRule                    | 负载均衡规则，选择实例         | ZoneAvoidanceRule                                            |
| IPing                    | 筛选掉 ping 不通的实例         | DummyPing                                                    |
| ServerList<Server>       | 交给 Ribbon 的实例列表         | Ribbon: ConfigurationBasedServerList<br />Spring Cloud Alibaba: NacosServerList |
| ServerListFilter<Server> | 过滤掉不符合条件的实例         | ZonePreferenceServerListFilter                               |
| ILoadBalancer            | Ribbont 的入口                 | ZoneAwareLoadBalancer                                        |
| ServerListUpdater        | 更新交给 Ribbon 的 List 的策略 | PollingServerListUpdater                                     |



**Ribbon 内置的负载均衡规则**

| 规则名称                  | 特点                                                         |
| ------------------------- | ------------------------------------------------------------ |
| AvailabilityFilteringRule | 过滤掉一直连接失败的被标记为 circuit tripped 的后端 Server，并过滤掉那些高并发的后端 Server或者使用一个 AvailabilityPredicate 来包含过滤 server 的逻辑，其实就就是检查 status 里记录的各个Server 的运行状态 |
| BestAvailableRule         | 选择一个最小的并发请求的 Server，逐个考察 Server，如果 Servert 被 tripped 了，则跳过 |
| RandomRule                | 随机选择一个 Server                                          |
| ResponseTimeWeightedRule  | 已废弃，作用同 WeightedResponseTimeRule                      |
| RetryRule                 | 对选定的负载均衡策略机上重试机制，在一个配置时间段内当选择 Server 不成功，则一直尝试使用subRule 的方式选择一个可用的 server |
| RoundRobinRule            | 轮询选择，轮询 index，选择 index 对应位置的Server            |
| WeightedResponseTimeRule  | 根据响应时间加权，响应时间越长，权重越小，被选中的可能性越低 |
| ZoneAvoidanceRule(默认)   | 复合判断 Server 所 Zone 的性能和 Server 的可用性选择 Server,在没有 Zone 的环境下，类似于轮询(RoundRobinRule) |



### 注解

| 注解                          | 元注解Target | 释义                                                         | 成员变量                                                     |
| ----------------------------- | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@RibbonClient`               | TYPE         | 声明 ribbon client 的配置，<br />@Import(RibbonClientConfigurationRegistrar.class) | name = ribbon client 名称<br />configuration = RibbonConfiguration.class<br />RibbonConfiguration 可以包含ILoadBalancer、ServerListFilter、IRule实例<br />需要避免被application context扫描，否则该配置会被所有RibbonClients 应用，**需要避免上下文重复扫描问题** |
| `@RibbonClients`              |              | 在单个类上组合多个@RibbonClient注解，即声明 ribbon client 的**全局配置**<br />@Import(RibbonClientConfigurationRegistrar.class) | defaultConfiguration = RibbonConfiguration.class<br />RibbonConfiguration 可以包含ILoadBalancer、ServerListFilter、IRule实例<br />**需要避免上下文重复扫描问题** |
|                               |              |                                                              |                                                              |
| **Spring Cloud Loadbalancer** |              |                                                              |                                                              |
| @LoadBalancerClient           | TYPE         | 声明 load balancer client 的配置，                           | name = load balancer client 名称<br />configuration = XXXXConfiguration.class<br />**需要避免上下文重复扫描问题** |



### 配置

```yaml
<clientName>:  # 局部配置 - 当Ribbon调用xxx-center/service<clientName>时，使用RandomRule...配置的负载均衡器
  ribbon:
	NFLoadBalancerclassName: ILoadBalancer实现类全路径
	NFLoadBalancerRuleclassName: IRule实现类全路径
	NFLoadBalancerPingclassName: IPing实现类全路径
	NIWSServerListclassName: ServerList实现类全路径
	NIWSServerListFilterclassName: ServerListFilter实现类全路径
ribbon:
  eager-load:  # 创建和初始化负载均衡器LoadBalancer的时机，eager-load enabled饥饿加载、懒加载
    enabled: true
    clients: user-center,xxx-service
```



### 扩展Ribbon

实现：定义实现 AbstractLoadBalancerRule 抽象类的自定义Rule 

#### 支持 Nacos 权重

目标：使 Ribbon 能根据 Nacos 权重进行负载均衡
具体实现：调用 `Instance instance = namingService.selectOneHealthyInstance(name);`

```java
package cn.cxc.contentcenter.configuration;
// ...
@Slf4j public class NacosWeightedRule extends AbstractLoadBalancerRule {
//    @Autowired
//    private NacosDiscoveryProperties nacosDiscoveryProperties;
    @Autowired
    private NacosServiceManager nacosServiceManager;

    @Override
    public void initWithNiwsConfig(IClientConfig iClientConfig) {
        // 读取配置文件并初始化 NacosWeightedRule
    }

    @Override
    public Server choose(Object o) {
        NamingService namingService = null;
        try {
            BaseLoadBalancer loadBalancer = (BaseLoadBalancer) this.getLoadBalancer();
            log.info("NacosWeightedRule: lb = {}", loadBalancer);
            // 将要请求的微服务名称
            String name = loadBalancer.getName();
            // 实现负载均衡算法...
            // 拿到服务发现的相关API
            // namingService = nacosDiscoveryProperties.namingServiceInstance();
            namingService = nacosServiceManager.getNamingService();
            // nacos client 自动通过基于权重的负载均衡算法，给我们选择一个实例
            Instance instance = namingService.selectOneHealthyInstance(name);
            log.info("选择的实例是：port = {}, instance = {}", instance.getPort(), instance);
            return new Server(instance.getIp(), instance.getPort()){
                @Override
                public MetaInfo getMetaInfo() {
                    return new MetaInfo() {
                        @Override public String getAppName() {
                            return instance.getServiceName();
                        }
                        @Override public String getServerGroup() {
                            return "";
                        }
                        @Override public String getServiceIdForDiscovery() {
                            return null;
                        }
                        @Override public String getInstanceId() {
                            return instance.getInstanceId();
            }	};	}	};
        } catch (NacosException e) {
            throw new RuntimeException(e);
}	}	}
```



#### 支持同一 Nacos 集群优先

目标：使 Ribbon 负载均衡支持同一 Nacos 集群的微服务实例优先调用
具体实现：
获得同集群下的所有服务实例，再调用负载均衡算法
查看 `namingService.selectOneHealthyInstance(name);` 具体实现，发现 `protected Balance.getHostByRandomWeight(List<Instance>):Instance`。通过调用该方法使得能在自定义实例范围中，调用负载均衡算法

```java
package cn.cxc.contentcenter.configuration;
// ...
@Slf4j public class NacosSameClusterWeightedRule extends AbstractLoadBalancerRule {
    @Autowired private NacosServiceManager nacosServiceManager;
    @Autowired private NacosDiscoveryProperties nacosDiscoveryProperties;

    @Override
    public void initWithNiwsConfig(IClientConfig iClientConfig) {}

    @Override
    public Server choose(Object key) {
        try {
            // 获取配置文件中Nacos集群名称
            String clusterName = nacosDiscoveryProperties.getClusterName();
            // 获取将要请求的微服务名称
            BaseLoadBalancer loadBalancer = (BaseLoadBalancer) this.getLoadBalancer();
            log.info("NacosSameClusterWeightedRule: lb = {}", loadBalancer);
            String name = loadBalancer.getName();
            // 获取服务发现的相关API - NamingService
            NamingService namingService = nacosServiceManager.getNamingService();

            // 1. 找到指定服务的所有实例 A
            List<Instance> instances = namingService.selectInstances(name, true);
            // 2. 过滤出相同集群下的所有实例 B
            List<Instance> sameClusterInstances = instances.stream()
                    .filter(instance -> Objects.equals(instance.getClusterName(), clusterName))
                    .toList();
            // 3. 如果B是空，就用A
            List<Instance> instancesToBeChosen = new ArrayList<>();
            if (CollectionUtils.isEmpty(sameClusterInstances)) {
                instancesToBeChosen = instances;
                // 发生跨集群调用，需要打印警告日志
                log.warn("发生跨集群的调用，serverName = {}, clusterName = {}, instances = {}",
                        name, clusterName, instances);
            } else {
                instancesToBeChosen = sameClusterInstances;
            }
            // 4. 基于权重的负载均衡算法，返回一个实例
            Instance instance = ExtendBalancer.getHostByRandomWeight2(instancesToBeChosen);
            log.info("选择的实例是：port = {}, instance = {}", instance.getPort(), instance);
            return new Server(instance.getIp(), instance.getPort()) {
                @Override 
                public MetaInfo getMetaInfo() {
                    return new MetaInfo() {
                        @Override public String getAppName() { return instance.getServiceName(); }
                        @Override public String getServerGroup() { return instance.getClusterName(); }
                        @Override public String getServiceIdForDiscovery() { return null; }
                        @Override public String getInstanceId() { return instance.getInstanceId(); }
            };	}	};
        } catch (NacosException e) {
            log.error("NacosSameClusterWeightedRule: 发生异常，原因未知", e);
            return null;
//            throw new RuntimeException(e);
}	}	}
// 自定义类继承Balancer，才能调用protected方法
class ExtendBalancer extends Balancer {
    public static Instance getHostByRandomWeight2(List<Instance> hosts) {
        return getHostByRandomWeight(hosts);
}	}
```



#### 支持 Nacos 元数据的版本管理

目标：选择符合配置中metadata定义的版本要求的微服务实例

```java
package cn.cxc.contentcenter.configuration;
// ...
@Slf4j public class NacosFinalRule extends AbstractLoadBalancerRule {
    @Autowired private NacosDiscoveryProperties nacosDiscoveryProperties;
    @Autowired private NacosServiceManager nacosServiceManager;

    @Override public void initWithNiwsConfig(IClientConfig iClientConfig) {}

    @Override
    public Server choose(Object key) {
        // 负载均衡规则：优先选择同集群下，符合metadata的实例
        // 如果没有，就选择所有集群下，符合metadata的实例

        // 1. 查询所有实例 A
        // 2. 筛选元数据匹配的实例 B
        // 3. 筛选出同cluster下元数据匹配的实例 C
        // 4. 如果C为空，就用B
        // 5. 随机选择实例
        try {
            // 获取Nacos元数据(实例级别)
            String targetVersion = this.nacosDiscoveryProperties.getMetadata().get("target-version");
            // 获取Nacos集群名称
            String clusterName = this.nacosDiscoveryProperties.getClusterName();
            // 将要请求的微服务名称
            DynamicServerListLoadBalancer loadBalancer = (DynamicServerListLoadBalancer) getLoadBalancer();
            log.info("NacosFinalRule: lb = {}", loadBalancer);
            String name = loadBalancer.getName();
            // 获取服务发现的相关API - NamingService
            NamingService namingService = this.nacosServiceManager.getNamingService();
            // 获取指定服务的所有实例
            List<Instance> instances = namingService.selectInstances(name, true);
            // 如果配置了版本映射，那么只调用元数据匹配的实例
            List<Instance> metadataMatchInstances = instances;
            if (StringUtils.isNotBlank(targetVersion)) {
                metadataMatchInstances = instances.stream()
                        .filter(instance -> Objects.equals(targetVersion, instance.getMetadata().get("version")))
                        .collect(Collectors.toList());
                if (CollectionUtils.isEmpty(metadataMatchInstances)) {
                    log.warn("未找到元数据匹配的目标实例！请检查配置。targetVersion = {}, instance = {}", targetVersion, instances);
                    return null;
            }	}
            // 如果配置了集群名称，需筛选同集群下元数据匹配的实例
            List<Instance> clusterMetadataMatchInstances = metadataMatchInstances;
            if (StringUtils.isNotBlank(clusterName)) {
                clusterMetadataMatchInstances = metadataMatchInstances.stream()
                        .filter(instance -> Objects.equals(clusterName, instance.getClusterName()))
                        .collect(Collectors.toList());
                if (CollectionUtils.isEmpty(clusterMetadataMatchInstances)) {
                    clusterMetadataMatchInstances = metadataMatchInstances;
                    log.warn("发生跨集群调用。clusterName = {}, targetVersion = {}, clusterMetadataMatchInstances = {}", clusterName, targetVersion, clusterMetadataMatchInstances);
                }
            }
            Instance instance = ExtendBalancer.getHostByRandomWeight2(clusterMetadataMatchInstances);
            return new Server(instance.getIp(), instance.getPort()) {
                @Override
                public MetaInfo getMetaInfo() {
                    return new MetaInfo() {
                        @Override public String getAppName() { return instance.getServiceName(); }
                        @Override public String getServerGroup() { return instance.getClusterName(); }
                        @Override public String getServiceIdForDiscovery() { return null; }
                        @Override public String getInstanceId() { return instance.getInstanceId(); }
            };	}	};
        } catch (Exception e) {
            log.warn("NacosFinalRule: 发生异常，原因未知", e);
            return null;
}	}	}
```



## OpenFeign

[github](https://github.com/openfeign/feign)



### 组成

| 接口               | 作用                                   | 默认值                                                       |
| ------------------ | -------------------------------------- | ------------------------------------------------------------ |
| Feign.Builder      | Feign 的入口，构建 FeignClient         | Feign.Builder                                                |
| Client             | Feign 底层用什么去请求                 | 和Ribboni配合时：LoadBalancerFeignClient<br />不和Ribboni配合时：feign.Client.Default |
| Contract           | 契约，注解支持                         | SpringMvcContract                                            |
| Encoder            | 编码器，用于将对象转换成HTTP请求消息体 | SpringEncoder                                                |
| Decoder            | 解码器，将响应消息体转换成对象         | ResponseEntityDecoder                                        |
| Logger             | 日志管理器                             | SIf4jLogger                                                  |
| RequestInterceptor | 用于为每个请求添加通用逻辑             | 无                                                           |



### 注解

| 注解                  | 元注解Target | 释义                                                         | 成员变量                                                     |
| --------------------- | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@EnableFeignClients` | TYPE         | 扫描@FeignClient接口，配置组件扫描指令<br />@Import(FeignClientsRegistrar.class) | basePackageClasses<br />defaultConfiguration<br />clients 若不为空，则禁用类路径扫描，指定@FeignClient类路径 |
| `@FeignClient`        | TYPE         | 声明一个 REST client                                         | name 带有可选协议前缀的服务名称<br />configuration<br />path 所有方法级映射使用的路径前缀 |

 feign.codec.Decoder, feign.codec.Encoder, feign.Contract.




# 未整理	

# Spring Cloud



服务提供者向服务发现组件发送心跳



客户端A 请求服务B集群（B不仅多还会动态变化），所以需要服务注册中心

服务端发现：Nginx、zookeeper、Kubernetes

客户端发现（通过轮询等负载均衡算法）：Eureka（支持异构，不同语言的服务）



## Eureka Server

注册中心

![1610702691547](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_23_15_42.png)

### eureka配置

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8763/eureka/,http://localhost:8762/eureka/
    register-with-eureka: true
  server:
#   自我保护模式：关闭
    enable-self-preservation: false
spring:
  application:
    name: eureka
#server:
#  port: 8761
management:
  context-path: /
```



## Client

### 配置

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/,http://localhost:8763/eureka/
# 自定义链接 http://clientname:8080/info
#  instance:
#    hostname: clientName
spring:
  application:
    name: client
```

## 注解

| 注解                   | 释义                                                         |
| ---------------------- | ------------------------------------------------------------ |
| @EnableEurekaServer    | 提供服务注册的功能，拥有所有服务节点的信息<br />在监控界面可以看到心跳检测机制(如果没有心跳就会被剔除)、<br />健康检查、负载均衡 (可以增加相对应的服务实例) |
| @EnableDiscoveryClient | 使用其他注册中心时可以使用                                   |
| @EnableEurekaClient    | 只有在使用Eureka作为注册中心时才可以使用                     |
|                        |                                                              |
|                        |                                                              |
|                        |                                                              |
|                        |                                                              |
|                        |                                                              |
|                        |                                                              |

