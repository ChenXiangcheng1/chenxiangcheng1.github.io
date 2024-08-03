---
title: Lombok
date: 2023-08-08
update: 2023-08-08

tags:
categories:
  - Dev
  - Java Library
keywords: Lombok
description: 主要关于Lombok的使用
top_img:
cover:
---



# lombok

通过使用注解来简化Java类的开发，自动生成模板代码

## 下载

* 方式一：下载 jar 包：https://projectlombok.org/setup/javac

```
目录：
projectname
	|-lib
			|-lombok.jar  // 下载 jar 包
	|-src
			|-cn.cxc.projectname
			|-module-info.java  // 模块依赖
```

```java
// module-info.java
module javabasic {
    requires static lombok;
}
```



* 方式二：maven 导入：https://projectlombok.org/setup/maven



## 使用

https://projectlombok.org/features/

https://projectlombok.org/api/lombok/package-summary



| 注解                     | 元注解Target | 释义                                                         | 成员变量                                                     | 例子                                                         |
| ------------------------ | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| @EqualsAndHashCode       |              |                                                              |                                                              |                                                              |
| @Nonnull                 |              | 自动生成非空检查，并在参数为 null 时抛出NullPointerException<br />区别：<br />JSR的只是一个标记，不会进行校验<br />Lombok的是编译时校验<br />Spring的是运行时校验<br />[Spring MVC笔记#校验](../Spring/spring-core/核心/Web/spirng_mvc.md#校验) |                                                              |                                                              |
| @With                    |              | 自动生成浅拷贝                                               |                                                              |                                                              |
| @Log4j2                  |              | 自动生成基于Log4j2的日志记录器                               |                                                              |                                                              |
| @Jacksonized             |              | 自动生成Jackson所需方法                                      |                                                              |                                                              |
| @Value                   |              | 构建 final 不可变对象                                        |                                                              |                                                              |
| **常用**                 |              |                                                              |                                                              |                                                              |
| **@Data**                |              | 等价于 @Getter @Setter @RequiredArgsConstructor @EqualsAndHashCode @ToString |                                                              |                                                              |
| **@Builder**             |              | 默认使用无参构造                                             |                                                              | `Customer customer1 = Customer.builder().name("Tom").age(16).build();` |
| **@NoArgsConstructor**   |              | 生成无参构造函数                                             |                                                              |                                                              |
| **@AllArgsConstructor**  |              | 生成包含所有字段的构造方法                                   |                                                              |                                                              |
| @RequiredArgsConstructor | TYPE         | 生成具有必需参数final/@NonNull字段的构造函数                 | onConstructor=<br />@__({@AnnotationsHere})<br />将注释放到生成的构造函数上 | `@RequiredArgsConstructor(onConstructor = @__(@Autowired))`  |
| @slf4j                   |              | 自动生成基于SLF4J的日志记录器logger。                        |                                                              |                                                              |
| @xslf4j                  |              | 自动生成基于SLF4J的日志记录器logger。<br />x支持占位符，可能需要slf4j依赖，反正我没生成logger(不用也罢)<br />`log.info("Logging with @XSlf4j: {}", "message");` |                                                              |                                                              |
| **实验性注解**           |              |                                                              |                                                              |                                                              |
| @Accessors               |              | getter和setter链式调用                                       |                                                              |                                                              |
| @UtilityClass            |              | 私有构造函数和静态方法                                       |                                                              |                                                              |
| @SuperBuilder            |              | 可以读取父类的属性                                           |                                                              |                                                              |
