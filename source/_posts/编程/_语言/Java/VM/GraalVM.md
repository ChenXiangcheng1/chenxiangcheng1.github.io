---
title: GraalVM
date: 2023-10-27
update: 2023-10-27

tags:
categories:
  - Java
  - VM
keywords: Java,GraalVM,AOT
description: 主要是GraalVM，Spring6.X支持GraalVM Native Image
top_img:
cover:
---

# GraalVM

[GraalVM 官方文档](https://www.graalvm.org/latest/docs/introduction/)	|	[GraalVM Guides](https://www.graalvm.org/latest/guides/)

编译成可执行文件(GraalVm Native Image)，无需依赖 VM，更适合 docker 容器



GraalVM 与 HotSpot 差异：
对应用程序的静态分析是在构建时从 main 入口点进行的 (HotSpot 在运行时进行分析)
在创建本地镜像时无法到达的代码将被删除，不会成为可执行文件的一部分
GraalVM不能直接知道你的代码中的动态元素，需要 hint 文件
应用程序的在构建时固定，不能改变
没有类的延迟加载，可执行文件中的所有内容都将在启动时加载到内存中
限制：Bean不能在运行时改变



AOT(ahead-of-time) 处理，将程序源代码提前编译成可执行文件，生成：

- Java 源代码
- 字节码(用于动态代理等)
- GraalVM的JSON hint文件(GraalVM不能直接知道你的代码中的动态元素，需要 hint 文件)
  - 资源 hint(`resource-config.json`)
  - 反射 hint(`reflect-config.json`)
  - 序列化 hint（`serialization-config.json`）。
  - Java代理 hint (`proxy-config.json`)
  - JNI hint (`jni-config.json`)



优点：内存占用更小、启动速度更快
缺点：反射(Hint文件)、动态代理(build时创建)需要特殊处理



## Spring GraalVM Native Image

在 AOT 处理阶段，应用程序被启动到可以使用 BeanDefinition 的程度，不会创建 Bean 实例

https://springdoc.cn/spring-boot/native-image.html#native-image.developing-your-first-application