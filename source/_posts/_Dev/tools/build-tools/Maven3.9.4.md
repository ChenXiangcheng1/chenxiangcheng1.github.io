---
title: Maven
date: 2023-08-04
update: 2023-08-04

tags:
categories:
  - Dev
  - Build Tools
keywords: Build Tools,Maven
description: 主要关于Maven的配置，POM，插件，IDEA集成Maven
top_img:
cover:
---



# Maven

* 对软件项目提供统一的构建**打包**方式，兼容不同的 IDE。
  JAR: 标准java项目
  WAR: web项目 (通过 maven-assembly-plugin 插件打包)

* **依赖管理**，将项目依赖的组件从远程仓库下载到本地仓库，并在工程中引用。



[入门指南 Maven Getting Started Guide 官网](https://maven.apache.org/guides/getting-started/index.html)

[Maven中央仓库搜索引擎 官方](https://search.maven.org/])

[Maven中央仓库搜索引擎 社区](https://mvnrepository.com/)

[IDEA 集成 Maven (重要)](https://www.jetbrains.com/help/idea/2023.2/maven-support.html)



本地 Maven(优先，因为更新方便)
Wrapper Maven 是项目自带的 [廖雪峰mvnw教程](https://www.liaoxuefeng.com/wiki/1252599548343744/1305148057976866)
Bundled Maven 是 IntelliJ 自带的



## 项目目录结构

```bash
${basedir}				// 项目根目录，basedir为项目名
|-- src					// 源码目录
|  	|-- main
|   |	|-- java		// Java源代码目录
|   |	`-- resources	// 资源文件目录，保存配置文件和静态图片
|   `-- test
|   	|-- java		// 测试类的源代码
|   	`-- resources	// 测试时使用的资源文件目录
|-- target				// 项目编译输出的目标目录，保存字节码文件和jar/war包
|   `- classes			// 字节码文件的编译输出目录
|-- pom.xml				// Project Object Model   
|-- mvnw				// (Maven Wraper才有)
`-- mvnw.cmd			// (Maven Wraper才有) Wraper包装器启动脚本
```



## 配置

pom(Project Object Model) 中配置(合并后优先级最高) 

用户配置(合并后优先级其次) `C:\Users\qq109\.m2\settings.xml`

全局配置(合并后优先级最低) `E:\Applications\Scoop\apps\maven\current\conf\settings.xml`



### 全局配置

[Settings Reference](https://maven.apache.org/settings.html)

[guide-configuring-maven](https://maven.apache.org/guides/mini/guide-configuring-maven.html)

```xml
<!-- 全局配置: E:\Applications\Scoop\apps\maven\current\conf\settings.xml -->
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!--
 | This is the configuration file for Maven. It can be specified at two levels:
 |
 |  1. User Level. This settings.xml file provides configuration for a single user,
 |                 and is normally provided in ${user.home}/.m2/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -s /path/to/user/settings.xml
 |
 |  2. Global Level. This settings.xml file provides configuration for all Maven
 |                 users on a machine (assuming they're all using the same Maven
 |                 installation). It's normally provided in
 |                 ${maven.conf}/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -gs /path/to/global/settings.xml
 |
 | The sections in this sample file are intended to give you a running start at
 | getting the most out of your Maven installation. Where appropriate, the default
 | values (values used when the setting is not specified) are provided.
 |
 |-->
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 https://maven.apache.org/xsd/settings-1.2.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->

  <!-- interactiveMode
   | This will determine whether maven prompts you when it needs input. If set to false,
   | maven will use a sensible default value, perhaps based on some other setting, for
   | the parameter in question.
   |
   | Default: true
  <interactiveMode>true</interactiveMode>
  -->

  <!-- offline
   | Determines whether maven should attempt to connect to the network when executing a build.
   | This will have an effect on artifact downloads, artifact deployment, and others.
   |
   | Default: false
  <offline>false</offline>
  -->

  <!-- pluginGroups
   | This is a list of additional group identifiers that will be searched when resolving plugins by their prefix, i.e.
   | when invoking a command line like "mvn prefix:goal". Maven will automatically add the group identifiers
   | "org.apache.maven.plugins" and "org.codehaus.mojo" if these are not already contained in the list.
   |-->
  <pluginGroups>
    <!-- pluginGroup
     | Specifies a further group identifier to use for plugin lookup.
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
  </pluginGroups>

  <!-- TODO Since when can proxies be selected as depicted? -->
  <!-- proxies
   | This is a list of proxies which can be used on this machine to connect to the network.
   | Unless otherwise specified (by system property or command-line switch), the first proxy
   | specification in this list marked as active will be used.
   |-->
  <proxies>
    <!-- proxy
     | Specification for one proxy, to be used in connecting to the network.
     |
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
    -->
  </proxies>

  <!-- servers
   | This is a list of authentication profiles, keyed by the server-id used within the system.
   | Authentication profiles can be used whenever maven must make a connection to a remote server.
   |-->
  <servers>
    <!-- server
     | Specifies the authentication information to use when connecting to a particular server, identified by
     | a unique name within the system (referred to by the 'id' attribute below).
     |
     | NOTE: You should either specify username/password OR privateKey/passphrase, since these pairings are
     |       used together.
     |
    <server>
      <id>deploymentRepo</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
    -->

    <!-- Another sample, using keys to authenticate.
    <server>
      <id>siteServer</id>
      <privateKey>/path/to/private/key</privateKey>
      <passphrase>optional; leave empty if not used.</passphrase>
    </server>
    -->
  </servers>

  <!-- mirrors
   | This is a list of mirrors to be used in downloading artifacts from remote repositories.
   |
   | It works like this: a POM may declare a repository to use in resolving certain artifacts.
   | However, this repository may have problems with heavy traffic at times, so people have mirrored
   | it to several places.
   |
   | That repository definition will have a unique id, so we can create a mirror reference for that
   | repository, to be used as an alternate download site. The mirror site will be the preferred
   | server for that repository.
   |-->
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
	<mirror>
      <id>aliyunmaven</id>
	  <mirrorOf>*</mirrorOf>
      <name>阿里云公共仓库</name>
      <url>https://maven.aliyun.com/repository/public</url>
	</mirror>									 
    <mirror>
      <id>maven-default-http-blocker</id>
      <mirrorOf>external:http:*</mirrorOf>
      <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
      <url>http://0.0.0.0/</url>
      <blocked>true</blocked>
    </mirror>
  </mirrors>
  
  <!-- profiles
   | This is a list of profiles which can be activated in a variety of ways, and which can modify
   | the build process. Profiles provided in the settings.xml are intended to provide local machine-
   | specific paths and repository locations which allow the build to work in the local environment.
   |
   | For example, if you have an integration testing plugin - like cactus - that needs to know where
   | your Tomcat instance is installed, you can provide a variable here such that the variable is
   | dereferenced during the build process to configure the cactus plugin.
   |
   | As noted above, profiles can be activated in a variety of ways. One way - the activeProfiles
   | section of this document (settings.xml) - will be discussed later. Another way essentially
   | relies on the detection of a property, either matching a particular value for the property,
   | or merely testing its existence. Profiles can also be activated by JDK version prefix, where a
   | value of '1.4' might activate a profile when the build is executed on a JDK version of '1.4.2_07'.
   | Finally, the list of active profiles can be specified directly from the command line.
   |
   | NOTE: For profiles defined in the settings.xml, you are restricted to specifying only artifact
   |       repositories, plugin repositories, and free-form properties to be used as configuration
   |       variables for plugins in the POM.
   |
   |-->
  <profiles>
    <!-- profile
     | Specifies a set of introductions to the build process, to be activated using one or more of the
     | mechanisms described above. For inheritance purposes, and to activate profiles via <activatedProfiles/>
     | or the command line, profiles have to have an ID that is unique.
     |
     | An encouraged best practice for profile identification is to use a consistent naming convention
     | for profiles, such as 'env-dev', 'env-test', 'env-production', 'user-jdcasey', 'user-brett', etc.
     | This will make it more intuitive to understand what the set of introduced profiles is attempting
     | to accomplish, particularly when you only have a list of profile id's for debug.
     |
     | This profile example uses the JDK version to trigger activation, and provides a JDK-specific repo.
    <profile>
      <id>jdk-1.4</id>

      <activation>
        <jdk>1.4</jdk>
      </activation>

      <repositories>
        <repository>
          <id>jdk14</id>
          <name>Repository for JDK 1.4 builds</name>
          <url>http://www.myhost.com/maven/jdk14</url>
          <layout>default</layout>
          <snapshotPolicy>always</snapshotPolicy>
        </repository>
      </repositories>
    </profile>
    -->

    <!--
     | Here is another profile, activated by the property 'target-env' with a value of 'dev', which
     | provides a specific path to the Tomcat instance. To use this, your plugin configuration might
     | hypothetically look like:
     |
     | ...
     | <plugin>
     |   <groupId>org.myco.myplugins</groupId>
     |   <artifactId>myplugin</artifactId>
     |
     |   <configuration>
     |     <tomcatLocation>${tomcatPath}</tomcatLocation>
     |   </configuration>
     | </plugin>
     | ...
     |
     | NOTE: If you just wanted to inject this configuration whenever someone set 'target-env' to
     |       anything, you could just leave off the <value/> inside the activation-property.
     |
    <profile>
      <id>env-dev</id>

      <activation>
        <property>
          <name>target-env</name>
          <value>dev</value>
        </property>
      </activation>

      <properties>
        <tomcatPath>/path/to/tomcat/instance</tomcatPath>
      </properties>
    </profile>
    -->
  </profiles>

  <!-- activeProfiles
   | List of profiles that are active for all builds.
   |
  <activeProfiles>
    <activeProfile>alwaysActiveProfile</activeProfile>
    <activeProfile>anotherAlwaysActiveProfile</activeProfile>
  </activeProfiles>
  -->
</settings>
```



#### 镜像

[阿里云 Maven 镜像](https://developer.aliyun.com/mirror/?serviceType=&tag=&keyword=maven)

[云效 Maven 仓库](https://developer.aliyun.com/mvn/guide)



### POM

Project Object Model

https://maven.apache.org/pom.html

#### Coordinates 坐标

| 坐标 cn.cxc:my-project:1.0 | 释义                                                     |
| -------------------------- | -------------------------------------------------------- |
| GroupId                    | 团队ID，一般采用逆向域名形式书写，com.zisu 公司.开发机构 |
| ArtifactIdId               | 构件ID (项目名称)                                        |
| Version                    | 版本号                                                   |



##### 版本

Alpha - 内测
Beta - 公测

M - 里程碑版本，试用
RC - Release Candidate，试用
RELEASE - 正式版本，建议等别人用
SR - Service Release，Bug 修复版本



#### Packaging 打包

```pom
<packaging>war</packaging>  <!-- 默认为jar -->
```

* jar
  用途：用于构建 Java 应用程序，如 Java 库、可执行程序等
* war
  用途：用于将整个 Web 应用程序打包成一个文件，然后部署到支持 Java Web 容器的服务器上





#### Properties 属性

指占位符

```pom
<properties>
    <spring.version>4.0.2.RELEASE</spring.version>
</properties>

<version>${spring.version}</version>
```



#### dependency

依赖关系

##### scope 作用域（重点）

类路径分为：编译类路径、测试类路径、运行类路径

| scope          | 释义                                                         |
| -------------- | ------------------------------------------------------------ |
| compile        | 这是默认范围。**依赖项在所有类路径上可用**，这些依赖项会传递到依赖项目。 |
| provided(重点) | 这与`compile`类似，但表示您希望 JDK 或容器在运行时提供依赖项。依赖项位于**编译**类路径和**测试**类路径中，并且不可传递。<br />例如，在构建 JavaEE 的 Web 应用程序时，您会将 Servlet API 和相关的 JavaEE API 的依赖关系范围设置为`provided`，因为Web容器会提供这些类。 |
| runtime        | 这个作用域表示编译时不需要该依赖项，但执行时需要。依赖项位于**运行**时类路径和**测试**类路径中，但不在编译类路径中。<br />例如，由于还在测试别的功能的阶段，还没有编写mysql，所以编译的时候不会用到这个包。但是生成环境Context上下文的时候会用到。也就是说这个包mysql包编译的时候是用不到的，但是在运行的时候会用到。所以如果我们在打包的时候没有将其打包到target中，运行时就会出错。所以要加上runtime。<br />虽然加了这个依赖，但是由于编译的时候没有用到，那么生成的target下的lib中是没有对应的jar包的。 |
| test           | 此作用域表明该依赖项不是应用程序正常使用时所必需的，仅用于**测试**阶段，不具有传递性。 |
| system         | 这与`provided`类似，但需要**显式**提供包含它的 **JAR 包**。该依赖项始终可用，不会在仓库中被查找。 |
| import         | 这个作用域仅适用于`<dependencyManagement>`部分中类型为`pom`的依赖项。它表示将POM中`<dependencyManagement>`部分中的依赖项替换为指定的依赖项。由于它们被替换，因此具有`import`作用域的依赖项实际上并不参与限制依赖项的传递性。 |
|                |                                                              |



##### optional

true，表示可选依赖，当其他项目依赖与本项目可选依赖项不会被直接引入，能够中断依赖传递

```xml
<optional>true</optional>
```



#### parent

 父子模块，用于继承父项目的配置，包括依赖项和其他构建配置。

```xml
<!-- 父模块配置 -->
<modules>
	<module>子模块artifactId</module>
	<module>another-project/pom-example.xml</module>
</modules>
```

```xml
<!-- 子模块配置 -->
<parent>
    <groupId>父模块groupId</groupId>
    <artifactId>父模块artifactId</artifactId>
    <version>父模块1.0-SNAPSHOT</version>
    <relativePath/> 
</parent>

<!-- 子模块间依赖 -->
<dependencies>
    <dependency>
        <groupId>com.imooc</groupId>
        <artifactId>sm_service</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```



##### relativePath

如果未指定，则在文件系统上 ../pom.xml Maven 查找父 pom，然后在本地仓库中查找，最后在远程仓库中查找
如果为空字符串，则从仓库解析



#### dependencyManagement

用于**集中**管理项目中所有模块的依赖项版本。

当该 POM 中声明了一个 groupId 和 artifactId 都匹配的依赖时，如果尚未指定版本和其他值，则 dependencyManagement 部分中的值将用于该依赖关系。

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2021.0.5.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```





## 命令

| 命令          | 释义                                                         | 参数                     |
| :------------ | :----------------------------------------------------------- | ------------------------ |
| mvn help      |                                                              |                          |
| mvn --version | 查看Maven版本和路径等信息                                    |                          |
| mvn clean     | 清空目标输出文件(不要打开target中的文件，不要有进程占用了)、删除临时文件、重置构建状态(下次完全重新编译测试) |                          |
| mvn install   | 编译项目、执行测试单元、打包项目、构建产物安装到本地仓库     | -DskipTests 跳过单元测试 |
| mvn compile   | 编译源代码                                                   |                          |
| mvn test      | 执行测试用例                                                 |                          |
| mvn package   | 项目打包到项目目录中                                         | -DskipTests 跳过单元测试 |
| ./mvnw        | 嵌入式 Maven                                                 |                          |



## 插件

[常用插件](https://juejin.cn/post/7142115195047903246#heading-11)	|	[文档](https://docs.spring.io/spring-boot/docs/3.2.0-SNAPSHOT/maven-plugin/reference/htmlsingle/)

基本上 IntelliJ IDEA 可以取代手动配置 Maven 插件，Maven 命令可以通过 GUI 查看并执行



### dependency

maven-dependency-plugin 是处理与依赖相关的插件。

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-dependency-plugin</artifactId>
      <version>2.8</version>
      <executions>
        <execution>
		  <phase>package</phase>
		  <goals>
		  	<goal>copy</goal>
		  </goals>
	    </execution>
      </executions>
      <configuration>
        <artifactItems>
         <artifactItem>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
          </artifactItem>
        </artifactItems>
        <!-- <outputDirectory>xxx</outputDirectory> -->
      </configuration>
    </plugin>
  </plugins>
</build>
```



| 在POM同级目录中，执行以下命令                                | 释义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| mvn dependency:resolve -Dclassifier=sources                  | 下载 POM 中所有组件的包的源码                                |
| mvn dependency:resolve -Dclassifier=javadoc                  | 下载 POM 中所有组件的包的 Javadocs                           |
| mvn dependency:sources -DdownloadSources=true -DdownloadJavadocs=true | 下载 POM 中所有组件的包的源码和 Javadocs                     |
| mvn dependency:get -Dartifact=[包groupId:artifactId:version]:jar:sources | 用插件下载单个包的源码<br />还可以在 **Project Structure 的 Libraries 中看对应包的源码sources** |
| mvn dependency:tree                                          | 显示项目的依赖树                                             |
|                                                              |                                                              |



### archetype 

| 命令                                                         | 释义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| mvn archetype:generate -DgroupId=com.zjbti  -DartifactId=webAppDemo -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false | 创建Maven的webapp项目工程结构<br />archetype: 原型，准确说就是一个项目模板<br />archetypeCatalog:internal，表示利用本地archetype-catalog.xml定义的archetype来创建项目。 |
|                                                              |                                                              |

### help 

| 命令                        | 释义                         |
| --------------------------- | ---------------------------- |
| mvn help:effective-settings | 输出合并后的配置 setting.xml |
| mvn help:effective-pom      | 输出合并后的 pom.xml         |
| help:active-profiles        | 输出 pom 中生效的 profile    |
|                             |                              |

### idea 

| 命令            | 释义             |
| --------------- | ---------------- |
| mvn idea:module | 自动生成.iml文件 |
|                 |                  |

### spring-boot-maven-plugin

| 命令                | 释义                  |
| ------------------- | --------------------- |
| mvn spring-boot:run | 启动 Speing Boot 项目 |
|                     |                       |

### maven-compiler-plugin

`mvn compile`会执行该插件，不用显式声明
除非需要在编绎阶段指定不同的JDK版本才需要显示声明，推荐用properties替代

```pom
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```



## 问题爬坑

### 打包报错

**错误描述：**

Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.22.2:test (default-test) on project missyou: There are test failures.

测试失败，可能是测试数据错误或者测试用例实现错误

**临时解决办法：**

![1612428669365](https://raw.githubusercontent.com/ChenXiangcheng1/image-hosting1/master/img/2023_03_12_20_56.png)

或者

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <!-- 忽略Test代码的测试失败 -->
        <testFailureIgnore>true</testFailureIgnore>
    </configuration>
</plugin>
```



