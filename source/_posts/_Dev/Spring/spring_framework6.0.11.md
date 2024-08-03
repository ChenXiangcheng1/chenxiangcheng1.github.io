---
title: Spring
date: 2023-07-13
update: 2023-07-13

tags:
categories:
  - Spring
keywords: Spring
description: Spring家族的思维导图
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_13_13_54.png
---

# Spring

一款依赖注入框架

![2018120318301458 (1)](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_13_13_54.png)



## 文档

[Reference](https://docs.spring.io/spring-framework/reference/)	|	[Reference(中文)](https://springdoc.cn/docs/)	|	[API 文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/)	|	[Github](https://github.com/spring-projects/spring-framework)



## 指南

[Guides][https://spring.io/guides]

[Todo：做几个start多的 Guides Github][https://github.com/orgs/spring-guides/repositories?q=&type=all&language=&sort=stargazers]

[todo 1: 在 Spring Academy 上做 Building a RESTful Web Service][https://spring.io/guides/gs/rest-service/]

[todo 2][https://spring.io/guides/gs/serving-web-content/]

[todo 3][https://spring.io/guides/tutorials/rest/]

[Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)

[Building a RESTful Web Service with Spring Boot Actuator](https://spring.io/guides/gs/actuator-service/)

[Converting a Spring Boot JAR Application to a WAR](https://spring.io/guides/gs/convert-jar-to-war/)

[spring-cloud-loadbalancer](https://spring.io/guides/gs/spring-cloud-loadbalancer)

[ytb spring-cloud](https://www.youtube.com/watch?v=aO3W-lYnw-o)



### 已完成

gs-caching



## 库

| Spring Artifacts(构件) | 分类                                                        |
| ---------------------- | ----------------------------------------------------------- |
| spring-core            | sping 基础包                                                |
| spring-context         | sping 基础包，提供 IOC 支持                                 |
| spring-beans           | sping 基础包，提供 Bean 管理支持                            |
| spring-aop             | sping 基础包，整合 aopalliance 和 aspectjweaver，运行时织入 |
| spring-expression      | 提供 spring 表达式支持                                      |
| spring-tx              | 提供 spring 事务支持                                        |
| spring-mvc             | 提供 Spring Web(MVC模式) 支持                               |
| spring-webflux         | 提供 Spring Web(响应式) 支持                                |
| spring-test            | spring 整合 Junit                                           |
| spring-jdbc            | spring 整合 Jdbc                                            |
| spring-aspects         | spring 整合 AspectJ，编译时织入                             |
| **第三方**             |                                                             |
| mybatis-spring         |                                                             |
| **其他**               |                                                             |
| mysql-connector-java   |                                                             |



## API

### CollectionUtils

org.springframework.util.CollectionUtils 提供了各种各样的集合实用方法，主要是框架内部使用

`CollectionUtils.isEmpty(Collection<?>):boolean` 判断给定的集合是否为空



## Spring Initializr

是一个初始化Spring项目的脚手架工具，提供选择项目Java版本、构建工具、依赖项的UI界面。

[Using start.spring.io Github](https://github.com/spring-io/start.spring.io/blob/main/USING.adoc)
[阿里云原生应用脚手架(最佳)](https://start.aliyun.com/)
[节点在香港](https://start.springboot.io/)



```bash
# 项目目录结构
.gitignore
HELP.md  # 依赖项的Reference和Guide链接
mvnw
mvnw.cmd
pom.xml
src
├── main
│ ├── java
│ │ └── org
│ │     └── acme
│ │         └── DemoApplication.java
│ └── resources
│     ├── application.properties
│     ├── static
│     └── templates
└── test
    └── java
        └── org
            └── acme
                └── DemoApplicationTests.java
```



## Experimental

[github spring experimental](https://github.com/spring-projects-experimental)

## 版本

[Spring与JDK版本对应关系](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-Versions)



### Spring5.X 的新特性

* 引入 Spring WebFlux 非阻塞 Web 框架。
  支持 reactive 响应式编程(响应式流操作，将应用程序组织为响应事件的流式处理)。但是要想非阻塞需要在整个处理链中都使用反应式库，一直到持久层，要求较高。

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_08_07_18_15.png" alt="image-20230807181500928" style="zoom: 50%;" />

> [响应式系统](https://www.reactivemanifesto.org/zh-CN)
> 响应式编程：核心1数据流驱动(flatmap)，比起命令式编程更关注数据流的处理。核心2数据驱动
> 不利好：Brian Goetz: "I think Project Loom is going to kill Reactive Programming"，拭目以待
>
> 异步编程：基于异步IO(NIO AIO)，有Java NIO Golang Goroutine/Channel



### Spring6.X 的新特性

[Wiki](https://github.com/spring-projects/spring-framework/wiki/What's-New-in-Spring-Framework-6.x)

* 基于 JDK 17+
* J2EE 迁移到 Jakarta EE 9+
* AOT(Ahead-Of-Time) 提前编译，编译前优化
  JIT(Just-in-Time) 即时编译，运行时优化
* Spring Native，基于 GraalVM 将 Spring 应用程序编译成原生镜像(本地机械码)
* 虚拟线程

