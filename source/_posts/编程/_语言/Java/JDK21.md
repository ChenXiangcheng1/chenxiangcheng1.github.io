---
title: Java
date: 2023-07-13
update: 2023-09-09

tags:
categories:
  - Java

keywords: Java
description: 介绍Java各平台版本，关于Java的思维导图
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_29_22_32.png
---

# Java主要考点

[云原生时代，Java 的危与机 周志明](https://www.bilibili.com/video/BV1CY411N7o2)



对于一种语言关注：
Null Safety：运行时通过Optional检查



服务端方向云原生架构：应用以容器部署，快速启动
JVM优点：多线程锁优化(但异步编程不好)、GC强，Spring生态丰富、适合团队开发
JVM缺点(依赖运行时环境VM 属于重量级编程语言 解决方案: GraalVM虚拟机  quarkus框架)：占内存(运营成本 解决: Valhalla值类型无需解引用)、启动慢(解决方案: AOT 提前编译 但是大量反射还是慢)、包大、~~人守旧(新特性)~~

期待：GraalVM、Loom虚拟线程



其它：Spring WebFlux 不如 vert.x



![image-20230329223218824](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_29_22_32.png)



## 对 Java 的理解

1. **平台无关性**：Compile Once, Run Anywhere 一次编译到处运行
   Java源码首先被编译成二进制**字节码**，再**由不同平台的JVM进行解析成机器指令**。
   Java语言在不同的平台上运行时不需要进行重新编译，Java虚拟机在执行字节码的时候，把字节码转换成具体平台上的机器指令。
   字节码优点：
   准备工作方面：因为直接解析每次执行都需要各种检查效率低，而字节码进行多次执行的效率更高
   兼容性方面：也可以将别的语言scala、Kotlin解析成字节码，增加了兼容扩展能力
2. **JVM：**hotSpot
   JVM = ClassLoader+Runtime Data Area(PC+JVM栈+本地方法栈+GC堆+元空间)+JNI+执行引擎
   GC机制(不必手动释放堆内存)
   [JVM笔记](./JVM/JVM.md)
3. **语言特性**：泛型、(反射、注解)
   [Java泛型笔记](./Java语言特性/Java泛型.md)、[Java反射笔记](./Java语言特性/Java反射机制.md)、[Java注解笔记](./Java语言特性/Java注解.md)
4. **OOP**面向对象编程：封装继承多态
5. 常用类库：集合、J.U.C 并发库、网络库、IO
6. 异常处理



## Java平台版本

* Java SE：用于桌面或简单服务器应用的 Java 平台。
  Java SE 允许您在桌面和服务器上开发和部署 Java 应用程序。Java SE 和组件技术提供了当今应用程序所需的丰富的用户界面、性能、多功能性、可移植性和安全性。

* Jakarta EE：用于复杂应用服务器应用的 Java 平台。
  Java EE 提供了一个 API 和运行时环境，用于开发和运行大型、多层、可靠且安全的企业应用程序，这些应用程序可移植、可扩展，并且可以轻松地与遗留应用程序和数据集成。

* Java ME：用于手机和其他小型设备的 Java 平台



### Java SE20

[OpenJDK 官网](https://openjdk.org/projects/jdk/)	|	[OpenJDK Wiki](https://wiki.openjdk.org/)	|	[OpenJDK github](https://github.com/openjdk/jdk)	

[Oracle Java 入门教程](https://dev.java/learn/)



[Java工具笔记](./JVM/JVM.md#Java工具)

常用：[Oracle JDK 官网](https://docs.oracle.com/en/java/javase/index.html)	|	[JDK17 API 文档](https://devdocs.io/openjdk~17/)

[Java 语言规范		|	JVM 规范](https://docs.oracle.com/javase/specs/index.html)

>JDK20#Java 语言规范
>
>
>1. Introduction
>2. Grammars
>3. Lexical Structure
>4. Types, Values, and Variables
>5. Conversions and Contexts
>6. Names
>7. Packages and Modules
>8. Classes
>9. Interfaces
>10. Arrays
>11. Exceptions
>12. Execution *
>13. Binary Compatibility
>14. Blocks, Statements, and Patterns
>15. Expressions
>16. Definite Assignment
>17. Threads and Locks *



### Jakarta EE10

[Jakarta EE 官网](https://jakarta.ee/)

Jakarta EE 标准是由 Eclipse 基金会 Foundation 开源社区领导和管理的企业级 Java 平台标准。

2010 年 Oracle 收购 Sun，2017 年 Java EE 移交给 Eclipse  Foundation 开源社区，Java EE 改名为 Jakarta EE。
目前 Oracle 维护 Java SE、Eclipse Foundation 维护 Jakarta EE。



#### 组件

* 客户端组件：客户端应用程序、applet
* 服务器端的 web 组件：Servlet、JSP
  Servlet 负责接收和处理 HTTP 请求，执行业务逻辑，并生成响应
  JSP 允许在 HTML 中嵌入 Java 代码，用于动态生成内容
* 服务器端的业务组件：EJB



#### 体系结构

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_08_25_12_17.png" style="zoom: 50%;" />



#### 十三种规范

1. JDBC(Java DatabaseConnectivity)

>​		JDBC API为访问不同数据库提供了统一的路径,像ODBC一样,JDBC开发者屏蔽了一些细节问题,另外,JDBC对数据库的访问也具有平台无关性。JDBC 可做三件事：与数据库建立连接、发送 操作数据库的语句并处理结果。
>​		有了JDBC，向各种关系数据发送SQL语句就是一件很容易的事。换言之，有了JDBC API，就不必为访问Sybase数据库专门写一个程序，为访问Oracle数据库又专门写一个程序，或为访问Informix数据库又编写另一个程序等等，程序员只需用JDBC API写一个程序就够了，它可向相应数据库发送SQL调用。同时，将Java语言和JDBC结合起来使程序员不必为不同的平台编写不同的应用程序，只须写一遍程序就可以让它在任何平台上运行，这也是Java语言“编写一次，处处运行”的优势，其次它增进了访问数据的效率和快捷程度。



2. JNDI(Java Name and DirectoryInterface)

> ​		JNDI API 被用于执行名字和目录服务.它提供了一致的模型来存取和操作企业级的资源DNS和LDAP,本地文件系统,或应用服务器中的对象。一个应用程序设计的API，为开发人员提供了查找和访问各种命名和目录服务的通用、统一的接口，类似JDBC都是构建在抽象层上。



3. EJB

​		Enterprise JavaBean 是 Jakarta EE 服务器端组件模型，用于部署分布式应用程序。提供了网络服务支持和 SDK，会话Bean(Session Bean)，实体Bean(Entity Bean)和消息驱动Bean(MessageDriven Bean)，定义了一个基于组件开发企业级应用程序的标准。



4. RMI(Remote Method Invoke)

> ​     远程方法请求,RMI协议调用远程对象上的方法.它使用了序列化的方式在客户端和服务器之间传递数据.RMI是一种被EJB使用的更底层的协议。



5. Java IDL/CORBA(通用对象请求代理架构是软件构建的一个标准 )

> ​       在Java IDL的支持下,开发人员可以将Java和CORBA集成在一起.他们可以创建Java对象并使之可在CORBAORB中展开,或者他们还可以创建Java类并和其它ORB一起展开的CORBA对象客户.后一种方法提供了另外一种途径,通过它Java可以被用于将你的新的应用程序和旧的系统集合在一起。



6. JSP

> ​		 JSP(JavaServer Pages) 类似 ASP 是一种动态模板引擎，是 Spring MVC 默认使用 JSP 解析 View。通过在传统的网页 HTML 中插入 Java 程序段(Scriptlet)和 **JSP标签**(tag)，从而形成 JSP 文件(*.jsp)。
> ​		用 JSP 开发的 Web 应用是跨平台的，既能在 Linux 下运行，也能在其他操作系统上运行。
> ​		服务器在页面被客户端所请求以后对 JSP 页面的 Java 代码进行处理，然后将生成的 HTML 页面返回给客户端浏览器。



7. Java Servlet

> ​		Servlet 是一种小型的 Java 程序,它扩展了 web 服务器的功能。作为一种服务器的应用,当被请求时开始执行。
> ​		Servlet 容器负责调度和管理 Servlet 的生命周期，会根据 Web 应用程序的部署描述符(web.xml)配置信息，实例化 Servlet 对象。还负责接收 HTTP 请求并传递给相应的 Servlet 进行处理并相应。Servlet容器有 Tomcat、Jetty、Undertow等。
>
> **JavaBean、Servlet、JSP 分别对应 Model、Controller、View**



8. XML

> ​		XML（标准通用标记语言的子集）是一种可以用来定其它标记语言的语言.它被用来在不同的商务过程中共享数据.XML的发展和java是相互独立的,但是,它和java具有的相同目标是平台独立性。



9. JMS

> ​		JMS是用于和面向对象消息的中间件相互通信的应用程序接口.它既支持点对点的域,又支持发布/订阅类型的域,并且提供了下列类型的支持:消息传递,事务型消息的传递,一致性消息和具有持久性的订阅者支持.JMS还提供了另一种方式来对新系统和旧后台系统相互集成。



10. JTA

> ​		JTA定义了一种标准API,应用程序由此可以访问各种事务监控。



11. JTS

> ​		JTS是CORBA OTS事务监控的基本实现.JTS规定了事务管理的实现方法.该事务管理器是在高层支持java Transaction API规范,并且在较低层次实现OMGOTS specification 和Java印象.JTS事务管理器为应用程序服务器,资源管理器,独立的应用以及同学资源管理器提供了事务服务。



12. JavaMail

> ​		JavaMail是用于存取邮件服务器的API,它提供了一套邮件服务器的抽象类.不仅支持SMTP服务器,也支持IMAP服务器。



13. JAF(JavaBeans ActivationFramework)

> ​		JavaMail利用JAF来处理MIME编码的邮件附件.MIME的字节流可以被转换成java对象,大多数应用都可以不需要直接使用JAF。



### Java ME



## JDK

Java Development Kit(JDK) = JRE + Java Tools

Java Runtime Environment(JRE) = JVM + Java API，不是开发人员只需要安装 JRE 就可运行 Java 程序



|           | 许可证                                                       | 说明                                                         |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| OpenJDK   | GPL协议(GNU General Public License)：任何源码的衍生产品对外发布必须保持同样的许可，公开源码并且他人可以复制和分发 | 是 JavaSE 规范的开源实现，不包含部署的功能(Java Web Stack、Java 控制面板)、部分源码使用开源代码替换、只包含最精简的JDK、不能使用 $Java^{TM}$ 商标 |
| OracleJDK | JRL协议：只允许个人研究使用                                  | 是在 OpenJDK 的基础上发布的                                  |

| **OpenJDK分发** |                                                |
| --------------- | ---------------------------------------------- |
| Eclipse Temurin | 由 Adoptium OpenJDK 改名而来，提供 Docker 镜像 |

[OpenJDK 第三方发行版推荐](https://whichjdk.com/)



Java Enhancement Proposal(JEP)，Java增强提案

### JDK8的新特性

* Lambda 表达式和函数式编程
  使得事件处理和并行编程等方面更加便捷和灵活
  [Lambda和方法引用笔记](./类库APIs/Lambda.md)

* Stream API
  新的操作集合的方式：过滤、映射、排序、归约等。
  [Stream API.md笔记](./类库APIs/Stream_API.md)
* 并行流（Parallel Streams）
  并行流是 Stream API 的扩展，提供了一种简单的方式来并行处理集合数据，可以有效地利用多核处理器来加速集合操作

* 默认方法（Default Methods）
  接口中可以有默认的方法实现

* 方法引用（Method References）
  [Lambda和方法引用笔记](./类库APIs/Lambda.md)

* Optional 类
  Optional 类是一个容器对象，可以包含或不包含非空值。它提供了一种更安全和更规范的方式来处理可能为空的值，避免了空指针异常。
  [Optional笔记](./类库APIs/Optional.md)
* 新的日期/时间 API
  引入了新的日期/时间 API(java.time包)，支持更方便的日期和时间操作、格式化、解析和时区处理



### JDK10的新特性

* 局部变量类型推断
  `var i = 10;`，var 只能用于局部变量



### JDK11的新特性

* JEP181：基于嵌套的访问控制

摘要：在 private、public、protected 的基础上，JVM 又提供了一种新的访问控制机制：Nest。

目标：如果你在一个类中嵌套了多个子类，那么子类中可以访问彼此的私有成员



* JEP309：添加动态类文件常量

摘要：增加一个常量类型 CONSTANT_Dynamic [JVM规范#常量池](https://docs.oracle.com/javase/specs/jvms/se20/html/jvms-4.html#jvms-4.4)

目标：为了简化和改进在编译时动态生成类的过程。

相比反射类型安全会进行编译时类型检查，减少了运行时开销

```java
import java.Lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.invoke.MethodType;

/**
 * <h1>动态语言API测试<h1>
 */
public class DynamicTest {
    public static void main(String[] args) throws Throwable {
    	MethodHandles.Lookup lookup MethodHandles.lookup();
    	MethodHandle mh lookup.findStatic(DynamicTest.class, "test", MethodType.methodType(void.class));
   		mh.invokeExact();
    }
    private static void test() {
        System.out.println("hello");    
    }
}
```



* JEP321：标准 HTTP 客户端

目标：HTTPClient 取代 HttpURLConnection

HTTPClient 更强大且异步



* JEP323：Lambda参数的本地变量语法

摘要：允许 var 在声明隐式类型的 lambda 表达式的形式参数时使用 `(@NutNull var parm1)->{}`

目标：将隐式类型的 lambda 表达式中的形式参数声明的语法与局部变量声明的语法(JDK10特性)对齐



* JEP330：启动单文件源代码程序

1`javac` 编译类 2`java -jar` 模块程序 3 `java HelloWorld.java`



* JEP333：ZGC

摘要：Z 垃圾收集器，也称为 ZGC，是一个可伸缩的低延迟垃圾收集器。

目标：（最核心）无论开了多大的堆内存(128G、2T)，保证低于10ms的JVM停顿，远胜于前一代的 G1。



* Epsilon GC

用于测试



* 新增的常用 API

String 支持 Unicode10

`strip():String` 类似 `trim()` 删除字符串的头尾空白符，基于Unicode

`stripLeading():String`

`stripTrailing():String`

`isBlank():boolean`

`lines():Stream<String>` 例`"1\n2\n3".foreach(System.out::println);`

`repeat(int):String`

Files

`Files.readString(Path):String`

`Files.writeString(Path, CharSequence, OpenOption...): Path` 例`Files.writeString("/tmp/a.txt", "JDK11", StandardCharsets.UTF-8);`

List

`List.of(E):List<E>` 返回的是一个不可变集合对象

`toArray(T[]):T[]`



### JDK12的新特性

* JEP230：微基准测试

摘要：Microbenchmark 作为常规性能测试的一部分，在 JDK 源代码中添加一组基础的微基准测试

目标：可以基于 Java Microbenchmark Harness(JMH) 轻松测试 JDK 的性能。



Java Microbenchmark Harness (JMH)

[github](https://github.com/openjdk/jmh)	|	[Maven仓库](https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-generator-annprocess/1.37)	|	[IDEA插件](https://github.com/artyushov/idea-jmh-plugin)



* JEP325：增强的 switch 语句

```java
int result = switch (Calendar.MONDA) {
	case Calendar.MONDAY -> System.out.println(1);
	case Calendar.SATURDAY -> System.out.println(6);
	default -> System.out.println();
};
```



* JEP344：G1垃圾收集器的增强，可中止的 G1 垃圾收集器

344摘要：如果G1垃圾收集器有可能超过预期的暂停时间，则可以使用中止选项。

344目标：G1可以中止可选部分的回收以达到停顿时间目标。



* JEP346：G1垃圾收集器的增强，G1 归还未使用的内存

346摘要：如果应用程序活动非常低，G1 应该在合理的时间段内释放未使用的 Java 堆内存给操作系统。

346目标：可以在空闲时自动将 Java 堆内存返还给操作系统。



### JDK15的新特性

* JEP 374：弃用并禁用偏向锁



### JDK16的新特性

* JEP 394：instanceof 的模式匹配

```java
// 以前的方式
if (obj instanceof String) {  // 测试obj是不是String，obj转换为String，声明新的局部变量s
    String s = (String) obj;
}
```

```java
// 使用 JEP 394 重构后
if (obj instanceof String s) {  //将目标与类型模式匹配，如果obj是String的实例则将其强制转换为String并将值赋值给模式变量s
    // Let pattern matching do the work!
}
```



### JDK21的新特性

* JEP439：[Generational ZGC](https://openjdk.org/jeps/439) 分代 ZGC
  [JVM#分代ZGC](VM/JVM.md#分代ZGC)

* JEP444：虚拟线程
  `Executors.newVirtualThreadPerTaskExecutor()`

TODO：本月发布，有空去看

