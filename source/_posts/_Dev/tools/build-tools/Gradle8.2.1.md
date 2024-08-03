---
title: Gradle
date: 2023-08-08
update: 2023-08-09

tags:
categories:
  - Dev
  - Build Tools
keywords: Build Tools,Gradle
description: 没什么内容
top_img:
cover:
---

# Gradle



## 文档

[Gradle User Manual](https://docs.gradle.org/8.2.1/userguide/userguide.html)

[TODO：B站教程](https://www.bilibili.com/video/BV1yT41137Y7/?p=10&spm_id_from=pageDriver&vd_source=93b89c6f2af23a0943ec1a2ee2419b49)	|	[Guides](https://gradle.org/guides/#getting-started)



## 项目目录结构

```bash
.
├─gradle  			// (Gradle Wraper才有) Wrapper包装器文件夹
│  └─wrapper
│		├─gradle-wrapper.jar
│		└─gradle-wrapper.properties
└─src
│   ├─main
│   │  ├─java    
│   │  └─resources
│   └─test
│       ├─java
│       └─resources  
└─build 			// 编译后的字节码文件、build后的jar/war包等信息
├─build.gradle  	// 构建脚本，类似于maven中的pom.xml
├─settings.gradle  	// 设置文件，定义项目及子项目名称信息，和项目是一对一关系
├─gradlew 			// (Gradle Wraper才有)
└─gradlew.bat 		// (Gradle Wraper才有) Wraper包装器启动脚本
```



## 配置

环境变量：`GRADLE_USER_HOME = E:\Applications\Scoop\apps\gradle-bin\current\.gradle` 指定 gradle 缓存依赖的位置



### .gradle

初始化脚本(.gradle 结尾的文件)的加载顺序: 命令行指定、`USER_HOME/.gradle/`、`USER_HOME/.gradle/init.d/`、`GRADLE_HOME/init.d/`

```init.gradle
// USER_HOME/.gradle/init.gradle
allprojects {
    repositories {
        maven { url 'https://maven.aliyun.com/repository/public/' }
        mavenLocal()
        mavenCentral()
    }
}
```



### 镜像

https://developer.aliyun.com/mirror/maven

[云效 Maven 仓库](https://developer.aliyun.com/mvn/guide)



### Groovy DSL

https://docs.gradle.org/8.2.1/dsl/



### Kotlin DSL

https://docs.gradle.org/8.2.1/kotlin-dsl/



## 命令

需要注意的是：gradle 的命令要在含有 build,gradle 的目录执行。

| 命令                 | 释义                                                         |
| -------------------- | ------------------------------------------------------------ |
| gradle clean         | 清空 build 目录                                              |
| gradle classes       | 编译业务代码和配置文件                                       |
| gradle test          | 编译测试代码，生成测试报告                                   |
| gradle build         | 构建项目<br />检测并下载依赖项到`GRADLE_USER_HOME/caches/modules-2/files-2.1`目录中 |
| gradle build -x test | 跳过测试构建项目                                             |
|                      |                                                              |



### Gradle Wrapper

| 命令                                                         | 释义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| gradle wrapper --gradle-version=x.x.x [--distribution-type all] | 修改gradle.properties中wrapper版本，待build时下载到`GRADLE_USER_HOME/wrapper/dists`目录中，同时缓存到`GRADLE_USER_HOME/caches`目录中<br />--distribution-type all 同时下载 Gradle Wrapper 源码和文档 |
| gradle wrapper --gradle-distribution-url=x.x.x               | 指定下载 Gradle 的地址distributionUrl                        |
|                                                              |                                                              |

```properties
# gradle-wrapper.properties
distributionBase=GRADLE_USER_HOME  													# 解压位置
distributionPath=wrapper/dists  													# 解压位置
distributionUrl=https\://services.gradle.org/distributions/gradle-8.2.1-bin.zip  	# 下载地址
networkTimeout=10000
validateDistributionUrl=true
zipStoreBase=GRADLE_USER_HOME														# 下载位置
zipStorePath=wrapper/dists															# 下载位置
```



## 插件

[插件搜索引擎](https://plugins.gradle.org/)

